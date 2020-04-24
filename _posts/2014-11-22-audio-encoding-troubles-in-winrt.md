---
layout: post
title: Audio Encoding troubles in WinRT
comments: true
tags: [Audio, Encoding, Error, WinRT]
categories: [Development]
---
Recently I've had to do some audio recording within a WinRT app, which is usually a fairly simple task as is demonstrated <a title="sample" href="https://code.msdn.microsoft.com/windowsapps/Media-Capture-Sample-adf87622">right here</a>.


My problem with it was with encoding. When you set off to record something you can set up your own MediaEncodingProfile, or use one of the presets by calling, for example, `MediaEncodingProfile.CreateM4a(...)`<!--more-->. That's all very nice. Unless you want to encode in MP3, which is supported on Windows 8 but not in Windows Phone 8, because the MP3 encoder is <a title="mp3" href="http://msdn.microsoft.com/en-US/library/windows/apps/windows.media.mediaproperties.mediaencodingprofile.createmp3">not shipped with WP8</a>. But that's not the reason I writing this post, oh no, I have a bigger issue here.

I'm going to cover Audio encoding specifically as it is the issue I was faced with, but I imagine the same thing will happen on the video encoding side. The main problem is the that the available MSDN docs seem to say that we can use the codecs from <a title="codecs" href="http://msdn.microsoft.com/en-us/library/windows/apps/hh986969.aspx">this list</a>. Awesome. Look at ALL those codecs. However, when you actually try to encode something things might not go exactly according to plan.

You see, I had to encode audio with the AMR-NB codec. It's not on the list, I know, but it does show up in the Audio Suptypes enumeration so I thought I'd give it a go (...it used to work on WP7). Well, that ended up not being possible because no codec was available. Ok... So I went for AD PCM instead, which <em>is</em> on the list. But, "computer says no". Well... GSM610 is also on the list. Nope. 

There I was trying to set the MediaEncodingProfile and it just kept throwing exceptions left and right. I even tried the <a href="http://msdn.microsoft.com/en-US/library/windows/apps/windows.media.mediaproperties.mediaencodingprofile.createfromfileasync">`CreateFromFileAsync()`</a> call, which "creates an encoding profile from an existing media file" to which I tried providing a file with the encoding I required. And I was still getting nowhere.

Well, as it turns out there's <a title="subtypes" href="http://msdn.microsoft.com/en-us/library/windows.media.mediaproperties.audioencodingproperties.subtype.aspx">another list</a>. And if you have a look at it, it's not quite as extensive as the first list. Turns out if I try to set up the encoding subtype to something that's not on that list, WinRT just throws an exception.

So unless I'm missing something it seems MSDN is providing some conflicting information in this case, which is quite sad. And ends up wasting our time.
