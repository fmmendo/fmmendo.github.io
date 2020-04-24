---
layout: post
title: WinRT Roaming Settings
comments: true
tags: [C#, Development, Settings, Storage, WinRT]
categories: [Development]
---
One thing consider when developing apps is that we might need to have data persist between uses or even between devices. With Windows 8 we are given 3 places where we can store data for each user profile (apart from the system libraries)<!--more-->:

<strong>Local:</strong> The local folder is just a basic, local folder for you to dump data to. Nothing special here. Good to store some settings for you app, for example.

<strong>Temp:</strong> The Temp folder is for temporary data... obviously. You can store data here but at your own risk since it might get deleted.

<strong>Roaming:</strong> This is a very cool little folder. Whatever you store in the Roaming folder, like the name suggests, automatically roams across all devices. So if a user has your app in all his devices you can have his personal settings synced in all instances of your app. The one thing you must pay attention to is that this folder has a <strong>size limit of 100KB</strong>. And if you by any chance go over that limit, <strong>nothing</strong>(!) will roam.
<h1>Storing and Retrieving Data</h1>
Storing data is quite easy. Everything you need is in the Windows.Storage.ApplicationData.Current class. You have acces to LocalFolder, LocalSettings, RoamingFolder, RoamingSettings, RoamingSettingsQuota (remember, 100KB limit) and TemporaryFolder. in order to actually write something, say in the RoamingSettings, you can the following:

```csharp
var roamingSettings = Windows.Storage.ApplicationData.Current.RoamingSettings;
var composite = new Windows.Storage.ApplicationDataCompositeValue();

composite["setting1"] = ...;
composite["setting2"] = ...;

roamingSettings.Values["mySettings"] = composite;
```

We get a reference to the RoamingSettings folder, stick our data into a container, and add the container to the settings. Getting our data back is just as easy:

```csharp
var roamingSettings = Windows.Storage.ApplicationData.Current.RoamingSettings;
var composite = (Windows.Storage.ApplicationDataCompositeValue)roamingSettings.Values["mySettings"];

//if null, then there's nothing in the Roaming folder
if (composite == null)
{
    ...
}
else
{
    MySetting1 = composite["setting1"].ToString();
    MySetting2 = composite["setting2"].ToString();
}
```

So, there. Simple and easy!
