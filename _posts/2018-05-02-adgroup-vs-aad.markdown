---
layout: post
title:  "Get AD Group Members´ AAD devices using PowerShell"
date:   2018-05-02 09:30:00 +0200
categories: [PowerShell, Active Directory, Azure Active Directory]
---
Have you ever had someone asking you for a list of Azure Active Directory (AAD) devices registered for members in an Active Directory (AD) group? Me neither, but it might come in handy.

In field IT there are many occasions when you need to more about the users and environment you support (but you are not managing yourself), e.g. what is the potential impact of a policy change in AAD to a certain team?

PowerShell offers you the opportunity to query members of an AD group, and then pull AAD information based on this. This is an example where I get group members, retrieve their AAD devices and add it to a .csv. Especially for larger groups this is convinient for consuming in Excel or Power BI.

Find the folder for this exercise here: [Azure_70-533/Manage-Azure-Identities/AD-Group_vs_AD-Device/DGroupMember-vs-ADDDevice.ps1][ADGroup-vs-AADDevice-script] [(link to folder)][ADGroup-vs-AADDevice-folder]

Tweaking what you pull from AD is quite easy. You might want to query an office or country, for example.

This is post is part of my Azure learning path. One of the objectives is Manage Azure Identities and in the course AD is brought up in addition to AAD. This exercise provides a bit of insight into both and PowerShell (which is a key tool for both).

You can read more about the purpose of Azure learning path in the [readme.md][Azure_70-533-readme] in my [Azure_70-533][Azure_70-533] repository.

[ADGroup-vs-AADDevice-script]: https://github.com/therockvalley/Azure_70-533/blob/master/Manage-Azure-Identities/AD-Group_vs_AD-Device/ADGroupMember-vs-ADDDevice.ps1
[ADGroup-vs-AADDevice-folder]: https://github.com/therockvalley/Azure_70-533/tree/master/Manage-Azure-Identities/AD-Group_vs_AD-Device
[Azure_70-533-readme]: https://github.com/therockvalley/Azure_70-533/blob/master/readme.md
[Azure_70-533]: https://github.com/therockvalley/Azure_70-533/