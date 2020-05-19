---
layout: post
title: Build 2020 Highlights - Day 1
comments: true
tags: [Article, Year in Review]
categories: [Article]
image:
    feature: build.png
---

Build has started today, and like most events during this time it has gone virtual. I've been following it online and just wanted to mention a handful of things from the opening keynotes that made an impression.
<!--more-->

# winget

We've had [chocolatey](https://chocolatey.org/) for a while, we've also had [scoop](https://scoop.sh/) and [appget](https://appget.net/) as alternatives. Microsoft then added `one-get` which also let you add providers such as chocolatey. But it looks like we now finally have a great package manager to use in Windows: [winget](https://github.com/microsoft/winget-cli). It seems it'll make things much simpler for us. It seems to use a [curated list of apps](https://github.com/microsoft/winget-pkgs/tree/master/manifests) and you can contribute to the list of apps in their repo [here](https://github.com/microsoft/winget-pkgs). I've been working on a script to set up my machine so it seems like I need to refactor them now 😅

# WSL2

WSL2 is out of preview as of the last version of windows, but we'll be getting more cool stuff soon. I was quite surprised to see a linux node in windows explorer, providing a quick link to the linux filesystem. And GUI apps running in windows, even if it's just in preview at the moment, is awesome. More details [here](https://devblogs.microsoft.com/commandline/the-windows-subsystem-for-linux-build-2020-summary).

# GitHub

I don't think any of this was "new new", but:
- Codespaces are great. And paired with having a dev environment working on the cloud is super useful.
- I hadn't heard of GitHub sponsors before. It's a great tool for all developers out there working on opens source stuff.

# Azure Static Web Apps

This was really cool to watch. It lets you deploy full stack web apps from a GitHub repo to Azure. These can be build by frameworks like Angular, React or View, or site generators like Gatsby. You can also optionally have a backend powered by Azure Functions. You set it up pointing to your repo and it automatically configures a GitHub Actions workflow. [Read more.](https://docs.microsoft.com/en-us/azure/static-web-apps/overview)

# WinUI 3

We had more details on the future of [WinUI](https://docs.microsoft.com/en-us/windows/apps/winui/winui3/)

# MAUI

OK, so they took Xamarin.Forms and turned it up to eleven. The [.NET Multi-platform App UI](https://devblogs.microsoft.com/dotnet/introducing-net-multi-platform-app-ui/) (MAUI) lets you target Windows, Android, iOS and macOS with a single project. (I need to look into this a bit more to see how it compares with [Uno Platform](https://platform.uno/))

# Project Reunion

So Microsoft is attempting to solve the UWP vs Win32 rift by decoupling APIs from these frameworks. This has been done for WinUI 3, Webview2, and MSIX. [Here](https://blogs.windows.com/windowsdeveloper/2020/05/19/developing-for-all-1-billion-windows-10-devices-and-beyond/)'s the Windows blog on this.

There's going to a lot more to unpack today and in the coming days, and there's a whole lot of content summarized in Microsoft's [Book of News](https://news.microsoft.com/build-2020-book-of-news) so I'm going to leave it here for now.
