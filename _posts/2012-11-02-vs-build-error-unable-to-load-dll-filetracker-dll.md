---
layout: post
title: VS - Build error - Unable to load DLL "FileTracker.dll"
comments: true
categories: [Error, Error, Visual Studio]
---
Well, last week I was facing a bit of a Visual Studio bug which was proving really hard to fix. Out of the blue some(!) projects just started throwing out this build error for no apparent reason:
```
Error 1 The "GenerateResource" task failed unexpectedly.
System.DllNotFoundException: Unable to load DLL 'FileTracker.dll': The specified module could not be found. 
(Exception from HRESULT: 0x8007007E)
```
The fixes and workarounds that I managed to dig up were no good. Manually running CSC.exe seemed to compile, but then I had no way to generate the setup files. Modifying the "Microsoft.Common.targets" file at the .Net Framework installation folder also failed to correct the issue. Finally I managed to find a working fix. All that has to be done is to open the *.csproj of the project that is failing to build and add this little snippet before the &lt;/Project&gt; closing tag (or if it's already there but set to 'true'... set it to 'false').
<pre class="brush: html;">    false

</pre>
With this I was finally able to get some work done.
