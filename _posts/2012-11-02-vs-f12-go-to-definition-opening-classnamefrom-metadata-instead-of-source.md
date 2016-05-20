---
layout: post
title: VS - F12 (Go To Definition) opening ClassName[from metadata] instead of Source
comments: true
categories: [Error, Visual Studio]
---
Well now that's two bugs in a row. This week while trying to debug an older project I hit F12 to go the definition of a certain method and instead I'm shown the metadata for that class (and obviously if I hit F11 to step into the method I get nothing).. and it's exactly the function that needs fixing! Usually such a thing happens when a project refence is out of date... and that's something that shouldn't really happen, since when a new version of whatever is being referenced is built, the path remains the same, so your project should already be pointing to the newer version. And that's wasn't happening.

Now, firstly, there are a few ways you can add a reference to your project: you can add a .NET assembly, a COM component, a project reference, or manually select a DLL. If you're referencing another project in the same solution, then you'll want to pick "Project" (linking to the DLL will also work, but you'll loose some debug capabilities, such as my F12 issue). So here I am trying to figure out what's happening and I find out I'm just pointing to the DLL! OK, that's an easy fix, I though, but for some reason the project I needed to reference didn't appear in the projects list. Odd. Well, long story short, you open the .csproj file and check what's wrong with the project reference, or in my case, manually add the project reference.

The .csproj file should include an entry like this. There might be issues with the Path, or Project GUID. To get the project GUID, just check the .csproj file of the Project whose reference you want to add.
