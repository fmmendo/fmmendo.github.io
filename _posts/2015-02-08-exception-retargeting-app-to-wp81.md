---
layout: post
title: Exception retargeting app to WP8.1
comments: true
tags: [Background Task, C#, Error, Windows Phone]
categories: [Development]
---
Very quick post here.

So I recently retargeted a WP8 app to WP8.1 (Silverlight) and tried adding a background task<!--more-->, but kept getting an `UnauthorizedAccessException` when attempting to register the task.

As it turns out it seems retargeting an app might be slightly buggy here and there, and I just couldn't get it to work. On the other hand a new 8.1 project with all the code/content of the old project dragged into it worked just fine.

(Well, that was half a day wasted.)
