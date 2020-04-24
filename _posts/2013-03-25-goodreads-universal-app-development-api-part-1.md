---
layout: post
title: GoodReads Universal App Development - API [Part 1]
comments: true
tags: [API, C#, Development, GoodReads, WinRT, UWP]
categories: [Development]
---
<em>UPDATE: I actually started this app before universal apps were announced and was using PCLs for the shared bits, but I have since updated that project to a Universal App.</em>

Well, I have a backlog of apps I want to build, and I've decided to start with an app that uses the Goodreads API.<!--more--> I've been wanting to do this for quite some time as I'm a bit of a bookworm, and there are a few features that come in very handy with a tablet or phone, and so far there's no Windows 8 app that fulfils my needs, so I thought I'd get started on something for both platforms.

My plan is to do this in 2-3 steps: first I want to get my API calls doing what I need, then I'll build the actual Windows 8 app with the data from the API calls, and later, since most of the work will already be done, I'll extend it to Windows Phone.

Simple! So let's get cracking!
<h1>Setting up a basic API call</h1>
So, the first thing on the agenda is to get familiar with the <a title="http://www.goodreads.com/api" href="http://www.goodreads.com/api" target="_blank">API that GoodReads kindly supplies </a>and get a library making the calls I need, and processing the data. Some API calls require a user to be authenticated, while others are free to use at any time. Well deal with authentication later, right now I just want to get the basic calls working, get the response and see the data I am given to work with. The search function is called like this:

```html
http://www.goodreads.com/search.xml?key=x1o7WGS4UEMFOGRtxIPoJA&q=Ender%27s+Game
```

First, we need to hit this with an with an Http Get Request:

```csharp
private static async Task HttpGet(string url)
{
    HttpWebRequest Request = (HttpWebRequest)WebRequest.Create(url);
    Request.Method = "GET";
    
    string httpResponse = null;
    HttpWebResponse response = (HttpWebResponse)await Request.GetResponseAsync();
    
    if (response != null)
    {
        StreamReader data = new StreamReader(response.GetResponseStream());
        httpResponse = await data.ReadToEndAsync();
    }
    return httpResponse;
}
```

We set whatever we want to the query parameter, make the call, and get a neatly organized XML encoded result. Looking at the returned XML, and making some other calls for other functions we can easily construct a few data structures to store our data, so we can go ahead a process the response:

```csharp
string results = await HttpGet(BASEURL + SEARCH + "?q=" + query + KEY);
var document = XDocument.Parse(results);
var items = document.Descendants("work");
var data = new List();
foreach (var item in items)
{
    var xmlbestbook = item.Descendants("best_book").ToList()[0];
    var xmlauthor = xmlbestbook.Descendants("author").ToList()[0];

    var author = new Author()
    {
    ID = Int32.Parse(xmlauthor.Element("id").Value),
    Name = xmlauthor.Element("name").Value,
    };

    var bestbook = new BestBook()
    {
        ID = Int32.Parse(xmlbestbook.Element("id").Value),
        ImageURL = xmlbestbook.Element("image_url").Value,
        SmallImageURL = xmlbestbook.Element("small_image_url").Value,
        Title = xmlbestbook.Element("title").Value,
        Authors = new List() { author }
    };

    data.Add(
        new Work()
        {
            BooksCount = Int32.Parse(item.Element("books_count").Value),
            ID = Int32.Parse(item.Element("id").Value),
            OriginalPublicationYear = !String.IsNullOrEmpty(item.Element("original_publication_year").Value) ? Int32.Parse(item.Element("original_publication_year").Value) : -1,
            OriginalPublicationMonth = !String.IsNullOrEmpty(item.Element("original_publication_month").Value) ? Int32.Parse(item.Element("original_publication_month").Value) : -1,
            OriginalPublicationDay = !String.IsNullOrEmpty(item.Element("original_publication_day").Value) ? Int32.Parse(item.Element("original_publication_day").Value) : -1,
            RatingsCount = Int32.Parse(item.Element("ratings_count").Value),
            ReviewsCount = Int32.Parse(item.Element("text_reviews_count").Value),
            Averagerating = Single.Parse(item.Element("average_rating").Value),
            BestBook = bestbook
        }
    );
}
```

Done! Looks good so far. I will need to add a few changes to this call since it only returns the first 20 results by default, there is a 'page' parameter that can be sent with the GET request, but I'll work that in later.

Next up, user authentication.
