---
title: Downloading PDC10 videos using the new async feature
author: admin
type: post
date: 2010-11-03T01:56:34+00:00
url: /?p=1156
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422208566;}'
categories:
  - .NET 4.0
  - odata

---
I knew PDC10 has an OData endpoint which is <http://odata.microsoftpdc.com/ODataSchedule.svc/> . The best part about  OData is querying for specific data that we are looking for. And here is my OData url for filtering twitter hashtag #languages

[sourcecode]

http://odata.microsoftpdc.com/ODataSchedule.svc/Sessions()?$filter=startswith(TwitterHashtag,&#8217;%23languages&#8217;)&$expand=DownloadableContent&$select=DownloadableContent

[/sourcecode]

With the above OData feed I could get urls for low bandwidth mp4&#8217;s that I can download. And here is the sample code for filtering

[sourcecode language=&#8221;csharp&#8221;]

var x =XDocument.Load(@"c:tempsession.xml").Descendants().AsParallel().Where(xd => xd.Name.LocalName=="Url"
  
&& xd.Value.Contains("_Low.mp4")).Select (xd => xd.Value);

[/sourcecode]

Now that I have the url&#8217;s ,here is the code to download the videos using the new async feature

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
using System.IO;
  
using System.Linq;
  
using System.Net;
  
using System.Threading.Tasks;
  
using System.Xml.Linq;

namespace Test
  
{
   
class Foo
   
{
   
static void Main(string[] args)
   
{
   
DownloadAsync();
   
Console.Read();
   
}
   
static async void DownloadAsync()
   
{
   
var result = new WebClient().DownloadStringTaskAsync("http://odata.microsoftpdc.com/ODataSchedule.svc/Sessions()?$filter=startswith(TwitterHashtag,&#8217;%23languages&#8217;)&$expand=DownloadableContent&$select=DownloadableContent");
   
var downloads = XDocument.Parse(await result).Descendants().AsParallel().
   
Where(xd => xd.Name.LocalName == "Url" && xd.Value.Contains("_Low.mp4")).
   
Select(xd => new WebClient().DownloadFileTaskAsync(xd.Value, Path.GetFileName(xd.Value)));
   
await TaskEx.WhenAll(downloads).ContinueWith(_ => Console.WriteLine("Downloading Complete"));
   
}
   
}
  
}
  
[/sourcecode]