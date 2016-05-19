---
layout: post
title: Google Reader, WinRT and C#
date: 2012-10-06 19:28
author: fmendo
comments: true
categories: [C#, Development, WinRT]
---
Lately I've been dealing with Google Reader while trying to accommodate it into a Windows 8 Store App I'm building. Google offers two flavours of it's <a title="feed API" href="https://developers.google.com/feed/v1/" target="_blank">feed API</a>: like most Google APIs there's a <a title="Javascript" href="https://developers.google.com/feed/v1/devguide" target="_blank">Javascript version</a>, and a <a title="Rest Version" href="https://developers.google.com/feed/v1/jsondevguide" target="_blank">REST version</a>, with a JSON interface. Now, both these APIs are nicely documented so I won't be wasting your time or mine on that. What I will do is touch on a couple subjects that weren't really all that well documented: user authentication, which oddly is not mentioned on Google's documentation, and having the whole thing work with WinRT, since there are a few differences when compared with .NET, Silverlight, and Windows Phone 7.
<h2>Authentication</h2>
OK, this actually isn't that much of a big deal, the reason I'm mentioning it is mostly because it's not specified in the documentation, and because the example code also ties in nicely with the rest of the API, so it's a good starting point as any other. It also applies to other situations/APIs, not just Google Reader, that authenticate users the same way.

If you look at the <a title="feed API" href="https://developers.google.com/feed/v1/" target="_blank">APIs documentation</a> there are a lot of things you can do without being authenticated, useful for many scenarios -- such as getting more items for an RSS feed rather than whatever default number it returns -- but not if you want to do something more interesting, like getting a user's subscriptions, unread items, and so on. For this we need a Session ID (SID) from Google, and we can get that by sending an Http GET Request to "https://www.google.com/accounts/ClientLogin?service=reader&amp;Email=email&amp;Passwd=password" and parsing the result, which contains an Auth key, a SID key and a LSID key. Something like this:
<pre class="brush: csharp;">var requestUrl = string.Format("https://www.google.com/accounts/ClientLogin?service=reader&amp;Email={0}&amp;Passwd={1}", 
                               _username, _password);
var req = (HttpWebRequest)WebRequest.Create(requestUrl);
req.Method = "GET";

var response = (HttpWebResponse)req.GetResponse();
using (var stream = response.GetResponseStream())
{
    using (var sr = new StreamReader(stream))
    {
        var resp = sr.ReadToEnd();

        var split = resp.Split(new string[] { "\n" }, StringSplitOptions.RemoveEmptyEntries);
        SID = split[0].Substring(split[0].IndexOf("=") + 1);
        LSID = split[1].Substring(split[1].IndexOf("=") + 1);
        AUTH = split[2].Substring(split[2].IndexOf("=") + 1);
    }
}
</pre>
With these in hand we can access a user's data on his Google reader account. Now, in the event that we need to write to the users' account, say, to mark something as read, add a star, or add a subscription, we will need to get an additional Token. The process is similar the previous request, but this time we need the SID we just stored, like so:
<pre class="brush: csharp;">_cookie = new Cookie("SID", _sid, "/", ".google.com");

var url = "http://www.google.com/reader/api/0/token";

var req = (HttpWebRequest)WebRequest.Create(url);
req.Method = "GET";
req.CookieContainer = new CookieContainer();
req.CookieContainer.Add(_cookie);

var response = (HttpWebResponse)req.GetResponse();
using (var stream = response.GetResponseStream())
{
    using(var sr = new StreamReader(stream))
    {
        _token = sr.ReadToEnd();
    }
}
</pre>
Now, if you just copy and paste those line into your nice Windows 8 App, what you get is a lot of ugly little squiggly red lines. Which brings us to my second point.
<h2>WinRT changes</h2>
The jump from .NET to WinRT caused a few changes on the APIs we used to know. While most of the code is almost identical, some classes and APIs might've been streamlined, replaced, or maybe (partly) deprecated, which is the case of HttpWebRequest and HttpWebResponse classes. Also, if your used to the WebClient class, it is also gone, replaced with the HttpClient In addition, you might have noticed that there are two very important keywords in WinRT development now: async and await.

As I said earlier, I'm not going to delve too deeply into the API itself, since it's pretty much Http Requests with different URLs and parameters (for said URLs and parameters, <a title="this" href="http://code.google.com/p/pyrfeed/wiki/GoogleReaderAPI" target="_blank">this</a> should provide all the information you need). What I will do is give two more examples here.

Take the Token request above, it can be rewritten as follows:
<pre class="brush: csharp;">private async void GetToken()
{
    var requestUrl = "http://www.google.com/reader/api/0/token";

    var request = (HttpWebRequest)WebRequest.Create(requestUrl);
    request.Method = "GET";
    request.Headers["Authorization"] = String.Format("GoogleLogin auth={0}", AUTH);
    request.Headers["Cookie"] = String.Format("SID={0}", SID);

    var response = await request.GetResponseAsync();
    using (var stream = response.GetResponseStream())
    {
        using (var sr = new System.IO.StreamReader(stream))
        {
            token = sr.ReadToEnd();
        }
    }
}
</pre>
This example, is not that much different. I used the HttpWebRequest and Response classes just to show a few of the changes there. The first thing you'll notice are a few asyncs and awaits here and there. The rest remains pretty much the same with two exceptions. Both HttpWebRequest.Headers and HttpWebRequest.CookieContainer have been slightly modified.  The Headers collection now does not support the Add() operation, nothing to keel over for. The CookieContainer retains much of its functionalities, but the Add() operation now requires additional parameters, so I find that sending the cookie as in the Headers is just cleaner.

The HttpClient offers some nice features, so to offer an example of its use lets kick things up a notch and make an API call.
<pre class="brush: csharp;">public async void GetUnreadCount()
{
    string requestUrl = "https://www.google.com/reader/api/0/unread-count?allcomments=true&amp;output=json&amp;ck=" + DateTime.Now.Ticks.ToString() + "&amp;client=scroll";

    var client = new HttpClient();
    client.DefaultRequestHeaders.Add("Authorization", String.Format("GoogleLogin auth={0}", AUTH));
    client.DefaultRequestHeaders.Add("Cookie", String.Format("SID={0}", SID));

    var response = await client.GetAsync(requestUrl);
    var content = await response.Content.ReadAsStringAsync();
    using (var stream = content.GetResponseStream())
    {
        using (var sr = new System.IO.StreamReader(stream))
        {
            JsonObject json = JsonObject.Parse(sr.ReadToEnd());

            //...
        }
    }
}
</pre>
Ok, so here we're getting the count of unread feeds for the logged in user. First thing to notice, you don't have to specify if the request is GET, POST, etc., as before, with the HttpClient class you just need to set up the attributes properly and call the GetAsync(), PostAsync() methods. Here we just need to set up the appropriate headers. For other API calls, just check what the appropriate URL and parameters are <a title="this" href="http://code.google.com/p/pyrfeed/wiki/GoogleReaderAPI" target="_blank">here</a> or <a title="here" href="http://blog.martindoms.com/2009/08/15/using-the-google-reader-api-part-1/" target="_blank">here</a>.
<h2>In conclusion</h2>
When moving from .NET to WinRT, a lot remains the same. There are some minor changes here and there, but if you're used to working XAML (whether you've come from WPF, Silverlight or WP7), Microsoft has managed to keep the same programming model we have been using for the past years, so transition should be painless.

I hope this small example might prove useful for someone out there. It is not a complex case, but it encompasses some of the changes I've noticed while moving to Windows 8, so I thought I'd just go ahead and share it.
