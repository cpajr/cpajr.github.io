---		
title: 'Azure Function and Communication Services'
classes: wide
---
I was recently tasked with exploring a method to interact with Azure Communications Services (ACS) for our Call Center.  In short, when prompted by a caller, a text message will be sent to them with pertinent self-help options.  To ease the burden on configuring our call center system, I decided to build a simple Azure Function that would receive a web POST, which includes a phone number, and an SMS message via ACS.  

It just happened that I recently had built a webhook receiver for our Netbox instance and had already determined a way to do most of this.  I had also built some familiarity with [Azure Functions Core Tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Cwindows%2Ccsharp%2Cportal%2Cbash) which allows for local development before deploying into Azure.  Personally, leveraging the Functions Core Tools has made developing an Azure Function so much easier.  

# Environment Variables
The new thing I learned is how to leverage declared environment variables.  Specifically, I learned that from within the Functions configuration you should declare your *important* pieces of information (i.e. Connection strings, passwords, etc), as opposed to placing them directly in the code.  With these items as environment variables, they can then be accessed by the code during runtime.[^1]

<p align="center">
  <img src="https://cpajr.com/assets/images/azure_func_env_var.png">
</p>
<p align="center">
  <img src="https://cpajr.com/assets/images/azure_func_env_var_1.png">
</p>

# Python Packages
While developing locally for the Function, I also had to leverage the *requirements.txt* file for the first time.  I had done this previously for other Python developments, but never for an Azure Function.  That said, once the package is declared within *requirements.txt*, it will be included when the Function is actually pushed into Azure.  In this case, I had to leverage the *azure-communication-sms* Python package.  

# Code
I have placed my code on [Github](https://github.com/cpajr/azure-sms-func) for review.


[^1]: I know, this is probably common sense.  Yet, it is was a discovery for me and felt it worthwhile to document/mention.  