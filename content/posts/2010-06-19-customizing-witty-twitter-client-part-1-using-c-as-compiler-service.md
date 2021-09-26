---
title: 'Customizing Witty Twitter Client Part 1- using C# as Compiler Service'
author: admin
type: post
date: 2010-06-20T03:28:46+00:00
url: /?p=840
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659084;}'
categories:
  - .NET
  - 'C#'
  - Mono

---
This is going to be a multipart blog post where I am going to be demonstrating how I have customized [This is going to be a multipart blog post where I am going to be demonstrating how I have customized][1] twitter client for my need. I chose Witty because it is the only OSS .NET twitter client I know of.

One of the reasons for customizing is primarily using C# as compiler service to extend it for my needs dynamically. For example I follow this twitter list <http://twitter.com/shanselman/programmers> , it’s a cool list that Scott maintains, thanks to him. But there is one person in this list who keeps tweeting about weight loss, which I am least interested and I didn’t have control over it but to ignore, until now. And it is always fun to write  software for your daily needs, which for me saves time.  Thanks to Mono for C# as compiler service. FYI I have shown a simple usage of Mono Csharp compiler service in this [post][2].

Here are things that I could get done by just spending few hours of my weekend time

Here is my the default witty

[<img class="alignnone size-full wp-image-841" title="InitialWitty" src="http://104.197.135.42/wp-content/uploads/2010/06/initialwitty2.jpg" alt="" width="450" height="484" />][3]

Filter the list by user name  which I am not interested in

[sourcecode language=&#8221;csharp&#8221;]

new Func<Tweet, bool>(t => !t.User.Name.Contains("CNN"));

[/sourcecode]

Here is the same without CNN

[<img class="alignnone size-full wp-image-842" title="Witty-FilteredByName" src="http://104.197.135.42/wp-content/uploads/2010/06/witty-filteredbyname2.jpg" alt="" width="450" height="436" />][4]

FYI the filter C# code is in the Filter Text box

The next filter criteria is to look for tweets that have hashtag as fsharp and also at least have one link

[sourcecode language=&#8221;csharp&#8221;]

new Func<Tweet, bool>(t => t.HashTags.DefaultIfEmpty().Contains("#fsharp") && t.Urls.DefaultIfEmpty().Count() > 0);

[/sourcecode]

And here is the filtered list

[<img class="alignnone size-full wp-image-843" title="Witty-FilteredByFSharp" src="http://104.197.135.42/wp-content/uploads/2010/06/witty-filteredbyfsharp2.png" alt="" width="450" height="436" />][5]

Be even crazier look for tweets that have hashtag as fsharp and at least one link and then open the link automatically

[sourcecode language=&#8221;csharp&#8221;]
  
new Func<Tweet, bool>(t => {
   
if (t.HashTags.DefaultIfEmpty().Contains("#fsharp") && t.Urls.DefaultIfEmpty().Count() > 0)
   
System.Diagnostics.Process.Start(t.Urls.First().ToString());
   
return t.HashTags.DefaultIfEmpty().Contains("#fsharp") && t.Urls.DefaultIfEmpty().Count() > 0; });
  
[/sourcecode]

[<img class="alignnone size-full wp-image-844" title="Witty-BrowserOpenLink" src="http://104.197.135.42/wp-content/uploads/2010/06/witty-browseropenlink2.png" alt="" width="450" height="268" />][6]

The browser with the link opened automatically.

It&#8217;s a hack and I shouldn&#8217;t be doing the above, but it does the job.

There is so much more possibilities. I could easily provide a save feature for these search scripts and reuse the same.  At the end of the series I will post the entire code in GitHub. But if you want to try it before that here are the simple changes I started doing to the code.

Extension Method for Compilation

[sourcecode language=&#8221;csharp&#8221;]

static class Extensions
   
{
   
public static object Compile(this string code)
   
{
   
return Mono.CSharp.Evaluator.Evaluate(code);
   
}
   
public static void Run(this string code)
   
{
   
Mono.CSharp.Evaluator.Run(code);
   
}
   
}
  
[/sourcecode]

The filter code

[sourcecode language=&#8221;csharp&#8221;]

public bool TweetFilter(object item)
   
{
   
Tweet tweet = item as Tweet;

// this will prevent the fade animation from starting when the tweet is filtered
   
tweet.IsNew = false;
   
try
   
{
   
Func<Tweet, bool> compare = (Func<Tweet, bool>)FilterTextBox.Text.Compile();
   
return compare.Invoke(tweet);
   
}
   
catch(Exception ex)
   
{
   
Console.WriteLine(ex);

}
   
return true;
   
}
  
[/sourcecode]

The hashtags and Links weren’t part of the tweet class , but they were part of the UI class. So I had to bring them to the tweet class and populate them.

[sourcecode language=&#8221;csharp&#8221;]
  
public IEnumerable<Uri> Urls
   
{
   
get
   
{
   
return links;
   
}
   
set
   
{
   
links.Clear();
   
links.AddRange(value);
   
}
   
}
   
public IEnumerable<string> HashTags
   
{
   
get
   
{
   
return hashTags;
   
}
   
set
   
{
   
hashTags.Clear();
   
hashTags.AddRange(value);
   
}
   
}

[/sourcecode]

And here is the code to populate the above properties

[sourcecode language=&#8221;csharp&#8221;]
  
private IEnumerable<Uri> GetLinks(string text)
   
{
   
string[] words = Regex.Split(text, @"([ (){}[]])");
   
return (from word in words
   
let isUrl = new Func<string, bool>(s => UrlShorteningService.IsUrl(s))
   
where isUrl(word)
   
select new Uri(word) ).ToList();
   
}
   
private IEnumerable<string> GetHashTags(string text)
   
{
   
string[] words = Regex.Split(text, @"([ (){}[]])");
   
return (from word in words
   
let ishash = new Func<string, bool>(s => s.StartsWith("#"))
   
where ishash(word)
   
select word).ToList();
   
}
  
[/sourcecode]

I know for the above 2 functions I could have written a High-Order Function

 [1]: http://code.google.com/p/wittytwitter/
 [2]: http://naveensrinivasan.com/2010/05/11/using-c-compiler-as-a-service-in-f-poshconsole-powershell/
 [3]: http://104.197.135.42/wp-content/uploads/2010/06/initialwitty2.jpg
 [4]: http://104.197.135.42/wp-content/uploads/2010/06/witty-filteredbyname2.jpg
 [5]: http://104.197.135.42/wp-content/uploads/2010/06/witty-filteredbyfsharp2.png
 [6]: http://104.197.135.42/wp-content/uploads/2010/06/witty-browseropenlink2.png