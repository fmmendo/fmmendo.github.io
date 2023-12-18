---
layout: post
title: Android v Xamarin [Part 2] - First Impressions
comments: true
tags: [Android, Xamarin, Java, C#]
categories: [Development]
---

As part of an ongoing project of mine, I build a very simple app to compare development on Android vs Xamarin.

This was intended to be a very simple thing, just to point out the first things one would notice between the two environments. This probably didn't even warrant a post, but there was one thing that did stand out for me.<!--more--> 

### XML vs AXML and Java vs C&#35;

I built an easy sample I found on the web, for a simple currency converter with an image, an input field, a button, some code behind and a toast. Nothing fancy. I just wanted to see what that would look like in C#. (I'll still be looking at AXML for now, I'll pick up Xamarin.Forms at some point).

First off, I was happy that I could literally copy and paste Android XML straight into Xamarin's AXML file, with one minor change, the namespace.

From this:
```xml
<RelativeLayout 
    tools:context="com.example.fmmendo.currencyconverter.MainActivity">
```

To this:
```xml
<RelativeLayout 
    tools:context="CurrencyConverter.MainActivity">
```

In the codebehind we find a few slight changes, but nothing crazy, mostly differences between Java and C#. Personally I find the `usings` nicer than the `imports`, but Xamarin does rely on class and method decorators, which looks less nice.

Here's regular Android:
```java
package com.example.fmmendo.currencyconverter;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        EditText dollarField = (EditText) findViewById(R.id.dollarField);
    }
}
```

And now Xamarin:
```cs
using Android.App;
using Android.Widget;
using Android.OS;
using Android.Views;

namespace CurrencyConverter
{
    [Activity(Label = "CurrencyConverter", MainLauncher = true, Icon = "@drawable/icon")]
    public class MainActivity : Activity
    {
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
            SetContentView(Resource.Layout.Main);

            EditText dollarFIeld = FindViewById<EditText>(Resource.Id.dollarField);
        }
    }
}
```

So far so good, just one more thing. UI event handling was were I had to pause and google how to make it work.

I added an `onClick` event to a button, which worked as expected on Android, but Xamarin couldn't find the method I had given. I could just us `FindViewById` in the codebehing to find the button and attach an event handler, but I didn't have to do that in Java, so I didn't want to do that in C# either.

Turns out it is possible by adding a reference to `Mono.Android.Export.dll`, and adding a method decorator:

```cs
    [Java.Interop.Export(nameof(convert))]
    public void convert(View v)
    {
        EditText dollarFIeld = FindViewById<EditText>(Resource.Id.dollarField);
        double dollarAmount = double.Parse(dollarFIeld.Text);
        double poundAmount = dollarAmount * 0.65;

        Toast.MakeText(ApplicationContext, "{poundAmount}", ToastLength.Long).Show();
    }
```

On the plus side though, at least it's possible to use some nice C# features like `nameof()` so I wont mess it up with typos in strings :)

That's all for this post.

I don't expect to post about Xamarin too soon, as I need to get to work on suff, but once I get closer to the end I'll have more to say. In the meantime I'm sure there will be some non-Xamarin content to talk about.
