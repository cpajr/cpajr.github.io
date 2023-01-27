---		
title: 'Microsoft Defender for Cloud'
classes: wide
---
This past week has been one of those annoying and challenging weeks.  I help support three servers which our used for our Integrated Platform as a Service (IPaas) within Azure.  I was approached by the application owner stating that a process on the server was causing high CPU.  Specifically, it was the process called *Windows Defender Advanced Threat Protection service*.[^1]  
<p align="center">
  <img src="https://cpajr.com/assets/images/screenshot_1.png">
</p>
My peer had been trying multiple approaches to remove or disable the service -- or at least limit its impact.  As you can imagine, the service was well protected from any tampering or forced removal.  That is when he pulled me in to put another pair of eyes on it.  

As any resourceful engineer might do, I turned to Google to find some answers.  My initial searches included the terms *uninstall defender server 2019*.  In all cases, the searches return articles on how to remove Defender.  We tried commands like `Uninstall-WindowsFeature Windows-Defender`[^2] or setting a registry key called `ForceDefenderPassiveMode` (see the following [article](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-antivirus-compatibility?view=o365-worldwide)).  None of our attempts were successful.  

During all of our searching I found that a feature within Azure called Defender for Cloud had been enabled for our servers.  
<p align="center">
  <img src="https://cpajr.com/assets/images/screenshot_2.png">
</p>
I had my suspicions that this feature -- which I did not enable personally -- was the source of our problems.  I personally was not familiar with this product, nor anyone on my team; however, we decided to disable the feature.  I let this change soak overnight.  
<p align="center">
  <img src="https://cpajr.com/assets/images/screenshot_5.png">
</p>
The next morning, I finally caught on to my search results being shown in Google: everything was talking about Defender Anti-Virus!  I needed to refocus my efforts on Defender Advanced Threat Protection (ATP).  I found an [article](https://vnextiq.com/windows-defender-advanced-threat-protection-iv/) which references Microsoft's Security Center.  This Microsoft product is now called *Microsoft 365 Defender*.  Our company does this use this product and was able to get access after bugging my security guy.  First, I found under the device inventory that these particular servers were showing as being *onboarded*. 
<p align="center">
  <img src="https://cpajr.com/assets/images/screenshot_3.png">
</p>
Continuing to follow the above article, I navigated to *Settings -> Endpoints -> Device Management -> Offboarding*.  I was able to download a script that would enable the offboarding of the servers.  
<p align="center">
  <img src="https://cpajr.com/assets/images/screenshot_4.png">
</p>
Having tried so many other things prior to this, I was hopeful.  

I went to the server and executed the provided script:
```
C:\Windows\system32>WindowsDefenderATPOffboardingScript_valid_until_2023-02-24.cmd

Starting Microsoft Defender for Endpoint offboarding process...

Testing administrator privileges
Script is running with sufficient privileges

Performing offboarding operations

Successfully offboarded machine from Microsoft Defender for Endpoint

Press any key to continue . . .
```

Success?  The service had been stopped and CPU usage was down:

```
C:\Windows\system32>sc query sense

SERVICE_NAME: sense
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

I tried a reboot to ensure that the settings stuck.  No luck!  With the reboot Defender ATP returned.  At this point, it was time to pull in some help from Microsoft.  So, I opened a support ticket.  Thankfully, the calling engineer had some experience with this.  Under each Virtual Machine, there is a section called *Extensions + applications*.[^3]  Under this section, we found an extension called *MDE.Windows* with a type called *Microsoft.Azure.AzureDefender...*.  
<p align="center">
  <img src="https://cpajr.com/assets/images/extension.png">
</p>
It was this extension that was causing Defender ATP to be reinstantiated upon a reboot.  We removed the extension, applied the offboarding script again, and Defender ATP remained disabled after a reboot.  

Finally, success! 

[^1]: This service was also redundant as another AV/ATP software was also installed.  Defender needed to be removed.   
[^2]: It was later discovered on other servers that Defender AV had been installed as well (my co-worker had already removed it from the trouble server by the time I arrived).  Using the command `Uninstall-WindowsFeature Windows-Defender` provided the needed results for this.
[^3]: I still need to do some additional research on this as I'm not entirely familiar with it. 