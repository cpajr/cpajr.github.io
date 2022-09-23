---		
title: 'Azure NSG and Tor-Exit Nodes'
classes: wide
---
# Introduction
Every so often, I would get a poke from my cybersecurity guy, mentioning that our publicly-exposed Azure servers were being probed by IP addresses related to Tor-Exit nodes.  I would usually ask if the server was being compromised; his typical return was, "no."  However, it was a frequent enough of events that the alerting was beginning to be bothersome.  Further, this gained importance following [Google's successfully blocking a 46 million rps DDoS.](https://cloud.google.com/blog/products/identity-security/how-google-cloud-blocked-largest-layer-7-ddos-attack-at-46-million-rps) Google found that the majority of the attack came from Tor-Exit nodes.  So, I finally decided I would try to remedy the situation.  The answer: apply a Network Security Group that would block traffic from IP addresses associated with Tor-Exit nodes.  

# Script Development
In our Palo Alto firewalls, I found that Palo Alto automatically creates a list with Tor-Exit node IP addresses.[^1]  Certainly, some kind person on the internet would generate and make a available a list of all Tor-Exit nodes.  To our rescue is a group called SecOps, an Institute of Information Security.  Every hour, an [updated list of Tor Exit nodes](https://github.com/SecOps-Institute/Tor-IP-Addresses) are generated.  Using this list as my source of information, I decided to build an Azure Function that would pull the list and then automatically apply it to an NSG.  

## Pull List
Wanting a small challenge, I decided to develop this script in Powershell instead of my typical Python.  In this script, it simply pulls down the list, parses it -- removing the unneeded IPv6 addresses, and store the list in a Blob container.  This function would only run on a weekly basis.  
```
# # Input bindings are passed in via param block.
param($Timer)

#################################
#       Global Variables        #
#################################

$url = "https://raw.githubusercontent.com/SecOps-Institute/Tor-IP-Addresses/master/tor-exit-nodes.lst"

#################################
#       Functions               #
#################################
function CheckIpAddr($ip_addr){
    <#
    This function will check whether the passed IP address is a 
    valied IPv4 address
    #>
    if ($ip_addr -match "^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$"){
        return $true
    }
    else{
        return $false
    }
}
#################################
#       Main                    #
#################################

$request = Invoke-WebRequest $url

if ($request.StatusCode -ne 200){
    Write-Host "ERROR: Invoke-WebRequest failed: $url"
    $host.Exit()
}

$output = ($request.content).Split("`n")

#Variable to follow position in list
$output_length = $output.Length
$counter = 1

$return_list = ""
foreach ($line in $output){
    $counter += 1
    if (CheckIpAddr($line)){
        if($counter -eq $output_length){ 
            $return_list = $return_list + $line
        }
        else{ 
            $return_list = $return_list + $line +"`n" 
        }
    }
}

Push-OutputBinding  -Name outputBlob -Value $return_list
```
The biggest challenge in this was putting the list in a format that I could easily then process.  I toiled with the idea of a CSV, but ultimately landed on a simple list.[^2]
## NSG Configuration
Once I have the list, a separate function, being triggered off a Container update, will then take the list and properly updated the desired NSG.  Let me share a few nuances I found in developing this particular function.
### Azure Powershell Authentication
Since I want this function to be executed autonomously, I needed a way to provide secure credentials.  I decided to use a [Service Principal Account](https://learn.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal) that only had access to the specific NSGs.[^3]  
### Set-AzNetworkSecurityGroup Commands
Microsoft is diligent in providing ways to manipulate Azure, either through API, Powershell, and Bash.  With using Powershell, I am leverage the `Get-AzNetworkSecurityGroup`, `Set-AzNetworkSecurityRuleConfig`, and `Set-AzNetworkSecurityGroup` commands.  Even though the commands are available, how to use them isn't always clear.  Specifically, the sequence needed to update a specific rule within an NSG.  When updating a specific rule, this is the following sequence of commands:
1. Using the `Get-AzNetworkSecurityGroup` command, you need the NSG object, not just the name.  This object contains all the specific information of the NSG, including the associated Resource Group, Location, ID, and all of the Security Rules.  
2. Now, using the `Set-AzNetworkSecurityRuleConfig` command, you need to generate the specifics of the update rule.  As noted in the [command reference](https://learn.microsoft.com/en-us/powershell/module/az.network/set-aznetworksecurityruleconfig?view=azps-8.3.0), `NOTE: There is no way to change a single attribute`.  It is possible to provide a list of IP addresses for, in the context of our application, the *Destination Address Prefix*.  However, this list must be an array of strings.
3. Once the Network Security rule has been set, you then need to use the `Set-AzNetworkSecurityGroup` command.  

Without this sequence of commands, the operation will not work.  
### Function Code
Putting this all together, the following is an example of my code:
```
# Input bindings are passed in via param block.
param([byte[]] $InputBlob, $TriggerMetadata)

#Global Parameters
#Update the following list with the list of NSG to be updated
$nsg_list = ""
#In my use case, I used the same name for the rule on all NSGs
$rule_name = ""

function ChangeRule($addr_list){
    #Needed variable to connect to Azure via Service Principal Acct
    $ApplicationId = $env:Application_ID
    $SecuredPassword = ConvertTo-SecureString $env:Application_Secret -AsPlainText -Force
    $TenantId = $env:Tenant_ID
    $Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $ApplicationId, $SecuredPassword
    Disable-AzContextAutosave | Out-Null
    Connect-AzAccount -ServicePrincipal -TenantId $TenantId -Credential $Credential | Out-Null
    
    foreach($the_nsg in $nsg_list){
        $nsg = Get-AzNetworkSecurityGroup -Name $the_nsg
        Set-AzNetworkSecurityRuleConfig `
            -Name $rule_name `
            -Access Deny `
            -Protocol * `
            -Priority 100 `
            -SourceAddressPrefix (@($addr_list)) `
            -SourcePortRange * `
            -DestinationAddressPrefix * `
            -DestinationPortRange * `
            -Direction Inbound `
            -NetworkSecurityGroup $nsg `
            | Out-Null
        $nsg | Set-AzNetworkSecurityGroup | Out-Null
        Write-Host "Successful process for $the_nsg"  
    }
}

# Write out the blob name and size to the information log.
Write-Host "PowerShell Blob trigger function Name: $($TriggerMetadata.Name) Size: $($InputBlob.Length) bytes"

$output = [System.Text.Encoding]::UTF8.GetString($InputBlob)
$list_addr = ($output -split '\r?\n').Trim()

ChangeRule($list_addr)
```

[^1]: For those wondering, you can find it under *Objects -> External Dynamic Lists -> Palo Alto Networks - Tor exit IP addresses*.  
[^2]: The last bit of code in the pull function was needed to avoid putting a *newline* in the list.  Unfortunately, I couldn't find a good way, in Powershell, to remove the last entry in an array.  
[^3]: Like from my previous post, I am leveraging the Function *Application Settings* and having my sensitive secrets exposed as environment variables within the function.  Thus, avoiding the need to hardcode the secrets into the code.