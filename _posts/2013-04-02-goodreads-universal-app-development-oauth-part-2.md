---
layout: post
title: GoodReads Universal App Development - OAuth [Part 2]
comments: true
tags: [API, C#, Development, GoodReads, OAuth, WinRT]
categories: [Development]
---
And I'm back. In the previous post I briefly touched on how the API calls for GoodReads looked like, now I'm going to mention authentication.

If you take a look at the GoodReads forums there are a lot of people with a few authentication troubles here and there, and solutions... not so much. The API is documented in ruby, and seen a couple of python implementations as well, but nothing on the C# side! So after much digging, banging my head, and cursing at my laptop I finally got it to work! ... I better write this down while it's still fresh. There is alot of information about OAuth at <a title="http://hueniverse.com" href="http://hueniverse.com">hueniverse.com</a>, which includes a great article on <a title="http://hueniverse.com/oauth/guide/authentication/" href="http://hueniverse.com/oauth/guide/authentication/">authentication</a>.
<!--more-->

<h1>Getting the OAuth Token</h1>
While there are a few API calls that do not require a user to be authenticated, the core of the social experience of GoodReads involves having a user logged in. WinRT gives us a nice class called WebAuthenticationBroker which brings up the familiar "Connect to a Service" popup, maintaining consistency across the OS, but GoodReads (and others) requires a bit more. It uses OAuth v1 so we need to deal with that first.

We need to get our API and Secret keys from <a title="http://www.goodreads.com/api/keys" href="http://www.goodreads.com/api/keys">here</a> and request an OAuth_token and OAuth_token_secret from <a href="http://www.goodreads.com/oauth/request_token">http://www.goodreads.com/oauth/request_token</a> . Once that's done we can call the WebAuthenticationBroker with <a href="http://www.goodreads.com/oauth/authorize">http://www.goodreads.com/oauth/authorize</a> and our tokens and, if the user authorizes our app, get an oauth_access_token in the process, which will allow us to make the other API calls. Phew, lots of work. I tried a couple of OAuth libraries for .Net/WinRT first but with no success and I didn't want to waste too much time so I decided to just do things manually.

Before we begin, a quick note: on your <a title="http://www.goodreads.com/api/keys" href="http://www.goodreads.com/api/keys">GoodReads API keys page</a>, there's an optional field you can use to write down your callback URL. Go ahead and do that now. You can get your app's callback url by calling WebAuthenticationBroker.GetCurrentApplicationCallbackUri().

For the request_token request, GoodReads is expecting something along the lines of:

```html
http://www.goodreads.com/oauth/request_token?oauth_nonce=95613465 &oauth_timestamp=1305586162 &oauth_consumer_key= &oauth_signature_method=HMAC-SHA1 &oauth_version=1.0 &oauth_signature=
```

In order to prepare our URL, we first need to get our parameters in order. These are the basic parameters used for oauth, nothing special. I stuck them in a SortedDictionary so I can later append them to the URL.

```csharp
public static SortedDictionary<string, string> GetOAuthParameters(string apikey, string secret)
{
    Random random = new Random();
    DateTime date = new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);
    TimeSpan timespan = DateTime.UtcNow - date;
    string oauthTimestamp = timespan.TotalSeconds.ToString(System.Globalization.NumberFormatInfo.InvariantInfo);
    string oauthNonce = random.Next(1000).ToString();

    var parameters = new SortedDictionary<string, string>();
    parameters.Add("oauth_nonce", oauthNonce);
    parameters.Add("oauth_timestamp", oauthTimestamp);
    parameters.Add("oauth_consumer_key", apikey);
    parameters.Add("oauth_signature_method", "HMAC-SHA1");
    parameters.Add("oauth_version", "1.0");

    return parameters;
}
```

With the parameters ready, we can now set up the call. We need to build a string consisting of the GoodReads request_token URL with the parameters added at the end. Note that the API secret must be signed with an HMAC-SHA1 hash.

```csharp
public static string CalculateOAuthSignedUrl(SortedDictionary<string, string> parameters, string url, string secret, bool toggle)
{
    StringBuilder baseString = new StringBuilder();
    string str;
    IBuffer keyMaterial;

    foreach (var param in parameters)
    {
        baseString.Append(param.Key);
        baseString.Append("=");
        baseString.Append(Uri.EscapeDataString(param.Value));
        baseString.Append("&");
    }

    //removing the extra ampersand 
    baseString.Remove(baseString.Length - 1, 1);
    str = "GET&" + Uri.EscapeDataString(url) + "&" + Uri.EscapeDataString(baseString.ToString());

    //calculating the signature 
    MacAlgorithmProvider HmacSha1Provider = MacAlgorithmProvider.OpenAlgorithm("HMAC_SHA1");

    if (toggle)
    {
        keyMaterial = CryptographicBuffer.ConvertStringToBinary(secret + "&" + OAuth_token_secret, BinaryStringEncoding.Utf8);
    }
    else
    {
        keyMaterial = CryptographicBuffer.ConvertStringToBinary(secret + "&", BinaryStringEncoding.Utf8);
    }

    CryptographicKey cryptoKey = HmacSha1Provider.CreateKey(keyMaterial);
    IBuffer dataString = CryptographicBuffer.ConvertStringToBinary(str, BinaryStringEncoding.Utf8);

    return url + "?" + baseString.ToString() + "&oauth_signature=" + Uri.EscapeDataString(CryptographicBuffer.EncodeToBase64String(CryptographicEngine.Sign(cryptoKey, dataString)));
}
```

With the URL ready and signed, all we need is to fire an HTTP GET Request and process the response. So far the order of things looks like this:

```csharp
public async static Task GetAuthTokens()
{
    string baseUrl = "http://www.goodreads.com/oauth/request_token";
    SortedDictionary<string, string> parameters = GetOAuthParameters(API_KEY, OAUTH_SECRET);

    string signedUrl = CalculateOAuthSignedUrl(parameters, baseUrl, OAUTH_SECRET, false);

    string response = await HttpGet(signedUrl);
    SetRequestToken(response);
}
```

After we do the GET request we must parse the response for our oauth_token and oauth_token_secret:

```csharp
private static void SetRequestToken(string response)
{
    string[] keyValPairs = response.Split('&');
    for (int i = 0; i < keyValPairs.Length; i++)
    {
        String[] split = keyValPairs[i].Split('=');
        switch (split[0])
        {
            case "oauth_token":
                {
                    OAuth_token = split[1];
                    break;
                }
            case "oauth_token_secret":
                {
                    OAuth_token_secret = split[1];
                    break;
                }
        }
    }
}
```

We can now store these safely and proceed to a very important moment...
<h1>Authentication</h1>
Almost there!

Now that we've got the tokens, we can use the WebAuthenticationResult to log a user in to GoodReads. This prompts the aforementioned 'Connect to a Service' dialog, which loads the GoodReads login page. The user logs in, authorizes our app, and we're thrown back into our code, to do the last bit of processing.

```csharp
public async void Authenticate()
{
    await GetAuthTokens();

    string goodreadsURL = "https://www.goodreads.com/oauth/authorize?oauth_token=" + OAuth_token;
    WebAuthenticationResult result = await WebAuthenticationBroker.AuthenticateAsync(WebAuthenticationOptions.None, new Uri(goodreadsURL), WebAuthenticationBroker.GetCurrentApplicationCallbackUri());

    if (result.ResponseStatus == WebAuthenticationStatus.Success)
    {
        await GetAccessToken(result.ResponseData);
    }
}
```

Once the login succeeds, we must parse the data in the ResponseData property to get our access_token from http://www.goodreads.com/oauth/access_token. So we very quickly to everything again: set up the parameters, don't forget our oauth_token, generate the URL, hash the secret key, fir the HTTP GET, and parse the response.

```csharp
public static async Task GetAccessToken(string responseData) 
{ 
    string oauth_token = null; 
    String[] keyValPairs = responseData.Split('&');
    string baseUrl = "http://www.goodreads.com/oauth/access_token"; 
    //parses the response string 
    for (int i = 0; i < keyValPairs.Length; i++) 
    { 
        String[] split = keyValPairs[i].Split('='); 
        if (split[0].Contains("oauth_token")) 
        { 
            oauth_token = split[1]; 
        } 
    }

    //Get basic parameters 
    SortedDictionary<string, string> parameters = GetOAuthParameters(API_KEY, OAUTH_SECRET);
    parameters.Add("oauth_token", oauth_token);
    string signedUrl = CalculateOAuthSignedUrl(parameters, baseUrl, OAUTH_SECRET, true);
    string response = await HttpGet(signedUrl);
    CalculateAccessToken(response);
}

public static void CalculateAccessToken(string responseData)
{
    string accessToken = null;
    string accessTokenSecret = null;
    string[] keyValPairs = responseData.Split('&');
    string username = string.Empty;
    for (int i = 0; i < keyValPairs.Length; i++)
    {
        String[] split = keyValPairs[i].Split('=');
        switch (split[0])
        {
            case "oauth_token":
                OAuthAccessToken = split[1];
                break;
            case "oauth_token_secret":
                OAuthAccessTokenSecret = split[1];
                break;
        }
    }
}
```

Phew! Done! We can now make calls to all the GoodReads API methods. Next step is to store the keys so they persist between uses, and maybe even roam between devices.

Well, that's it for now. I'll be back once I implement the API and am ready for the next step!
