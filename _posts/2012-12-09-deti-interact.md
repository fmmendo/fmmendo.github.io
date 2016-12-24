---
layout: post
title: DETI Interact
comments: true
tags: [.NET, Android, C#, Project, WPF, Xaml, XNA, App]
categories: [App]
---
I would like to kick off my Apps 'section' by briefly detailing a bit of the work I did during my Master’s Thesis.<!--more-->

The idea behind this project was to replace the information displayed in a few public displays at the lobby of my department. These were running a static webpage with news and lecturer information. The developed software would replace these webpages with dynamic content, and allow users to connect via mobile devices to interact with the system. I was free to use whatever technologies I wished, as long as I targeted Android for my mobile app, and used a few existing web services.

<img src="/assets/deti1.png" alt="deti1" width="474" height="266" />
<img src="/assets/deti2.png" alt="deti2" width="474" height="266" />

I decided to use this chance to learn WPF and dig deeper in to the .NET framework. The end result is a WPF application, which gets data from a server and presents it to passers-by, and an Android app that allows users to connect to the system and interact with the content. As an experiment I also added to the WPF application the Google Earth plugin, and a small XNA application, in order take advantage of some sensors available on current smartphones.

The source code for the WPF application can be found in <a href="http://detiinteract.codeplex.com">codeplex</a>.
