---
layout: post
title: Jekyll on Windows
comments: true
tags: [Jekyll, Windows, Windows 10, Bash, Ubuntu]
categories: [Development]
---

With a new year starting I tasked myself to get a bit more active with my current and upcoming projects, as well as this website, so I've been setting up my dev environment from scratch to start the year with a clean slate.

A while back, I decided to move this site from a relatively cumbersome Wordpress to a rather nimble Jekyll instance.<!--more--> At the time I followed the instructions on the [Jekyll Website](https://jekyllrb.com/docs/windows/). While this worked at the time (with a few minor issues), I decided I now wanted to try my luck with the [Windows Subsystem for Linux](https://blogs.msdn.microsoft.com/wsl/). Let's get started!

### Install Bash on Ubuntu on Windows

The first thing we need to do is get Linux to run in Windows, this can be done by installing the Windows Subsystem for Linux, which is available as of the Anniversary Update (also known as version 1607, or build 14393 if my memory serves me right). The installation guide can be found [here](https://msdn.microsoft.com/en-gb/commandline/wsl/install_guide) but the gist of it is:


1. Turn-on Developer Mode (in Setings -> Update and Security -> For Developers)
2. "Turn Windows features on or off" and select **Windows Subsystem for Linux (beta)**
3. Open a command prompt, run `bash` and follow instructions


Once that's done a shortcut will be placed in the start menu (or you can run `bash` at a command prompt) and we can now get started.

### Installing Jekyll

Now, at the moment Ubuntu on Windows doesn't have much, so we need to start by installing `make` and `gcc`.


```
$ sudo -s 
$ apt update 
$ apt install make gcc
```

With that done, we can move on to installing Ruby. Jekyll needs at least Ruby 2.0, and the latest stable release is 2.3.0, so we're getting that.

```
$ apt-add-repository ppa:brightbox/ruby-ng
$ apt update
$ apt install ruby2.3 ruby2.3-dev
```

Finally we can install Jekyll and serve some websites!

```
$ gem install jekyll
$ cd /mnt/c/<path/to/website>
$ jekyll serve
```
I also had to install `jekyll-paginate` and `jekyll-gist` to get my site to work, no problem there.

One issue you may come across is that the current release doesn't support filesystem watchers (and Ruby/Jekyll require `libinotify`). ~~This has been addressed as of build #14942, so you'll have to join the Windows Insider Preview, or wait for to be pushed out which will most likely happen with the Creators Update.~~ This is fixed with the Creators Update.

That's all from me!
