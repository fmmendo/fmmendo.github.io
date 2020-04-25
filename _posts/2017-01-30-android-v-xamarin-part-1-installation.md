---
layout: post
title: Android v Xamarin [Part 1] - Installation
comments: true
tags: [Android, Xamarin, Java, C#]
categories: [Development]
---

Two things that have been on my to do list for a while were learning a bit more Android and Xamarin, so I decided to do both at the same time. The plan was refresh my knowledge of Android and build _something_ with it, and then reproduce it in Xamarin, and use it as an exercise to compare them.<!--more-->

I had planned on starting with a very simple app. Not quite a hello world, just something with some input and output. This was also not a bad idea as just getting Xamarin to install and stop complaining was quite the endeavour. Due it taking way more time than I expected, I decided to vent a bit on this post and do the app comparison on a next post.


### Installing Android Studio and SDK

Ok, I didn't really need to add this section, but, whatever. Getting ready to work on Android is simple. You go to [Android's developer page](https://developer.android.com/studio/index.html), and download Android studio. Once it's installed you're ready to go. The only thing I don't quite like is that the SDK can't be installed on a path with spaces in it, _ugh_, so I can't put it where it belongs.

### Installing Xamarin

Oh boy.

OK, to be fair installing Xamarin can be a breeze. You can get the installer from their [website](https://www.xamarin.com/download) or just add it as a feature to your visual studio installation. I had trouble with it because I'm _very_ picky with what happens on my computer, and Xamarin doesn't play nice with it.

So, for a bit of context, once Xamarin is installed you can go into your options and change your JDK, Android SDK and Android NDK locations. But if you tell the installer to no bother downloading and installing all that stuff because you already have it on your machine, it just won't install. If you attempt to uncheck any of the items in the 20+GB of stuff it wants to install, it just unchecks everything.

<img class="aligncenter  wp-image-106" src="/assets/vsinstaller.png" alt="VS Installer" width="480" height="851" />

After searching for solutions to this the [only workaround](http://stackoverflow.com/questions/31792693/how-to-make-visual-studio-2015-installer-know-that-i-already-have-android-sdk) I found was to add a registry key that told the installer where the Android SDK was. This is done by going to `HKEY_LOCAL_MACHINE\SOFTWARE` adding an `Android SDK Tools` key, and in it a new String Value `Path` which would point to the SDK location.

With this the installer was happy with the Android SDK but I was still missing the JDK and the NDK. Now, while the same thing [can be done for the JDK](http://stackoverflow.com/a/13086434) I had another problem. Android studio installs the Android SDK but apparently has it's own built-in version of the JDK, so I had nothing to add. Reluctantly, I went and installed the most recent JDK and then Xamarin installed as expected.

(As a sidenote I'll just add that I _hate_ having Java installed and I _never_ do it on my machines as it's a security nightmare. It's also the reason I have touched Android or Xamarin in years... but I did want to get these comparisons done. I'll format my machine when I'm done)

### Can I do some work now?

No. That would be nice, but no. So I open Visual Studio, create a new Android project, add some stuff, and does it build? No. First, intellisense doesnt work and has no knowledge of any Android classes, and, well, the Output says it needs Java 6 or 7. _What?!_ So I uninstall, JDK 8,  fire up the installer again, check the JDK box, and it installs. Back to the project, does it build? No! Why? Well, internet says I need JDK 8. Oh ffs! I install 8 again, change the path in Xamarin's options and it finally builds and deploys.

Hurray! 
