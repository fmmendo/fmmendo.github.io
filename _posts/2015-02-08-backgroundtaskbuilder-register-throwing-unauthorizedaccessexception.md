---
layout: post
title: BackgroundTaskBuilder.Register throwing UnauthorizedAccessException
date: 2015-02-08 21:52
author: fmendo
comments: true
categories: [Background Task, C#, Error, Error, Windows Phone]
---
Very quick post here.

So I recently retargeted a WP8 app to WP8.1 (Silverlight) and tried adding a background task, but kept getting an UnauthorizedAccessException when attempting to register the task.
As it turns out it seems retargeting an app might be slightly buggy here and there, and I just couldn't get it to work. On the other hand a new 8.1 project with all the code/content of the old project dragged into it worked just fine.

(Well, that was half a day wasted.)
