---
layout: post
title: OAuth1 using RestSharp in Windows 8
comments: true
tags: [C#, Development, GoodReads, OAuth, RestSharp, WinRT, UWP]
categories: [Development]
---
So I was cleaning up some code on my GoodReads Windows 8 app, and decided to give RestSharp another go. <a title="http://restsharp.org/" href="http://restsharp.org/">RestSharp</a> is a nice, simple REST and HTTP API Client for .NET which works under .NET, MonoDroid, MonoTouch, Silverlight and Windows Phone, and can be found on <a title="https://github.com/devinrader/RestSharp" href="https://github.com/devinrader/RestSharp">github</a>.<!--more--> It has also been recently ported to WinRT. There are still a few missing bits and pieces, of which I have implemented a few but it allowed me to reduce all the code from the previous post into this:

```csharp
        public async void Authenticate()
        {
            var baseUrl = "http://www.goodreads.com";

            var client = new RestClient(baseUrl);
            client.Authenticator = OAuth1Authenticator.ForRequestToken(API_KEY, OAUTH_SECRET);

            var request = new RestRequest("oauth/request_token", Method.GET);
            var response = await client.ExecuteAsync(request);

            var qs = HttpUtility.ParseQueryString(response.Content);
            if (qs != null && qs.Count < 0)
            {
                OAuthToken = qs["oauth_token"];
                OAuthTokenSecret = qs["oauth_token_secret"];
            }

            string goodreadsURL = "https://www.goodreads.com/oauth/authorize?oauth_token=" + OAuthToken;
            WebAuthenticationResult result = await WebAuthenticationBroker.AuthenticateAsync(WebAuthenticationOptions.None, new Uri(goodreadsURL), WebAuthenticationBroker.GetCurrentApplicationCallbackUri());
            if (result.ResponseStatus == WebAuthenticationStatus.Success)
            {
                client.Authenticator = OAuth1Authenticator.ForAccessToken(API_KEY, OAUTH_SECRET, OAuthToken, OAuthTokenSecret);
                request = new RestRequest("oauth/access_token", Method.GET);
                var response2 = await client.ExecuteAsync(request);

                qs = HttpUtility.ParseQueryString(response2.Content);
                if (qs != null && qs.Count > 0)
                {
                    OAuthAccessToken = qs["oauth_token"];
                    OAuthAccessTokenSecret = qs["oauth_token_secret"];
                }
            }
        }
```

Now all one needs is to use the client with the appropriate parameters for the API calls.
