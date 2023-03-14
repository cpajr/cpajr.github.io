---		
title: 'Slow shell with Pyenv'
classes: wide
---
I leverage [Pyenv](https://github.com/pyenv/pyenv) to manage the different versions of Python I might use within my system.  Recently, I decided to install Pyenv to my WSL Ubuntu 20.04 instance on Windows 11.  Once I had done this, the prompt became slow as I would change directories or list the contents of a directory.  Initially, I was not able to find anything to suggest what was causing the problem.  It wasn't until I looked closely at what was added to .bashrc and .profile did I notice a discrepancy to the install instructions.  

Specifically, I believe it was the the *eval "$(pyenv virtualenv-init -)*.  Now, the following is what I have within my BASH related files:

```
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Problem solved with this change.  