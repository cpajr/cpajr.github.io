---		
title: 'Microsoft Defender for Cloud'
classes: wide
---
This past week has been one of those annoying and challenging weeks.  I help support three servers which our used for our Integrated Platform as a Service (IPaas) within Azure.  I was approached by the application owner stating that a process on the server was causing high CPU.  Specifically, it was the process called *Windows Defender Advanced Threat Protection service*.  

My peer had been trying multiple approaches to remove or disable the service -- or at least limit its impact.  As you can imagine, the service was well protected from any tampering or forced removal.  That is when he pulled me in to put another pair of eyes on it.  

As any resourceful engineer might do, I turned to Google to find some answers.  My initial searches included the terms *uninstall defender server 2019*.  In all cases, the searches return articles on how to remove Defender.  We tried commands like `Uninstall-WindowsFeature Windows-Defender` or setting a registry key called `ForceDefenderPassiveMode` (see the following [article](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/microsoft-defender-antivirus-compatibility?view=o365-worldwide)).  


