---
title: 'Configuring Solarized color scheme on Windows 10 Ubuntu bash'
categories:
- Blog
---
I've become a fan of the Ubuntu bash on Windows 10; however, like most other people, I am not a fan of the default color schemes.  Now comes the effort to configure a different color scheme.  
<!--more-->
### Solarized
I have also become a fan of the [Solarized](http://ethanschoonover.com/solarized) and wanted to have it on my Ubuntu bash.  To help with this, I found several blogs describing how to do this, but the best was from [Noel Bundick](https://www.noelbundick.com/posts/getting-started-bash-on-windows/).  He lays out specifically how to get it working for the Ubuntu bash.  I had to change solarized-dark.reg file a little:

```
[HKEY_CURRENT_USER\Console\C:_Program Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_1604.2017.922.0_x64__79rhkp1fndgsc_ubuntu.exe]
"ColorTable00"=dword:00362b00
"ColorTable01"=dword:00969483
...
```

I also implemented Noel's color changes for the directories. 
