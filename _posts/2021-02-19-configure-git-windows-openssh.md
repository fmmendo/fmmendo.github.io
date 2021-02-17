---
layout: post
title: Configuring git to use Windows's OpenSSH implementation
comments: true
tags: []
categories: []
---

Short one today. I had one of those moments where I was trying to fix something that wasn't actually broken, so I'm going to write it down before I make the same mistake again 😅.

With Windows 10 version 1803 (the April 2018 update), Microsoft added a direct implementation of OpenSSH.<!--more--> But if like me (who tends to just use https most of the time) and you've just blindly set up `git` to connect to a repo using ssh it's most likely using the built-in ssh client.

Let's skip all the `ssh-keygen` stuff, as that's not really relevant at this point. You've got your key, the repo has the public key, everything seems to work and yet every time you `pull` or `push` you get asked for your password. Frustrating.

First, ensure the `ssh-agent` is running. You can do it via the Services UI (`Win`+`R`: `services.msc`), or since your most likely already in a command line:

```bash
Set-Service ssh-agent -StartupType Automatic
```

Now add the key to the agent:

```bash
ssh-add ~/.ssh/id_rsa
```

And here's the bit that I was missing and was driving me nuts: actually configure git to use the ssh agent you added the key to:

```bash
git config --global core.sshComand C:/Windows/System32/OpenSSH/ssh.exe
```

It should now no longer keep pestering you for your password.
