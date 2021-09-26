---
title: Combining Stack Overflow RSS, OData and API to query
author: admin
type: post
date: 2010-07-14T03:18:05+00:00
url: /?p=978
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1406555122;}'
twitter_cards_summary_img_size:
  - 'a:7:{i:0;i:1440;i:1;i:860;i:2;i:2;i:3;s:25:"width="1440" height="860"";s:4:"bits";i:8;s:8:"channels";i:3;s:4:"mime";s:10:"image/jpeg";}'
categories:
  - linqpad
  - odata

---
In my opinion [Stack Overflow][1] has a ton of knowledge to learn new tricks. And there are some really smart people in the SO community. I try and learn new things when I find time.

I subscribe to RSS feeds for new questions on a particular topic. Example, here is one for F# from Stack Overflow <http://stackoverflow.com/feeds/tag/f%23>. The advantage of the RSS feed is I get to see new questions, but the drawback is I would have to navigate to the site to look for answers. AFAIK the [stacky][2] (stack overflow API) does not provide a mechanism for querying new questions based on a tag.

It was easy for me to combine both of them to solve my problem. With RSS feed I could discover new questions and with the stacky I could get answers . And I use Linqpad as a scratchpad so it was easy to write-up something quick.

[sourcecode language=&#8221;csharp&#8221;]

void Main()
  
{
   
var reader = XmlReader.Create("http://stackoverflow.com/feeds/tag/f%23");
   
var feed = SyndicationFeed.Load<SyndicationFeed>(reader);
   
var length = "http://stackoverflow.com/questions/".Length;

var client = new StackyClient("1.0", File.ReadAllText(@"c:tempso.txt"),HostSite.StackOverflow,new UrlClient(), new JsonProtocol());

var feedItems = from item in feed.Items
                   
let nextOccurence = item.Id.ToString().IndexOf("/",length)
                   
let getId = new Func<int>(() => Convert.ToInt32( item.Id.Substring(length,nextOccurence &#8211; length)))
                   
select new {Id = getId(), Title = item.Title.Text, Body = item.Summary.Text.StripHTML()};

var answers = client.GetQuestionAnswers(feedItems.Select (y => y.Id),new AnswerOptions() { IncludeBody = true});

// The latest F# feed questions and answers
   
var qa = from question in feedItems
            
join answer in answers on question.Id equals answer.QuestionId
            
where answer.Accepted == true
            
select new { Title = question.Title, Question = question.Body.StripHTML(), Answer = answer.Body.StripHTML()};
   
qa.Dump();
  
}
  
public static class Extensions
  
{
        
public static string StripHTML(this string s)
        
{
           
return Regex.Replace(s, @"<(.|n)*?>", string.Empty);
        
}
  
}

[/sourcecode]

[<img class="alignnone size-full wp-image-981" title="SO" src="http://104.197.135.42/wp-content/uploads/2010/07/so1.jpg" alt="" width="700" height="418" />][3]

And if you have been following F# and functional programming then you would probably know [Tomas][4]. I would also like to read what he has been answering. Again stacky does not provide an API to query user by name. This is where the SO [OData][5] comes in handy and LinqPad handles OData very well. Here is the code to get Tomas user id via OData and query for questions and answers which he has answered using stacky .

[sourcecode language=&#8221;csharp&#8221;]
  
var tomas = Users.Where(u => u.DisplayName.StartsWith("Tomas Pet")).First().Id;
  
var tomasQA = from ans in  client.GetUsersAnswers(tomas,new AnswerOptions() { IncludeBody = true })
                
select new { Title = ans.Title, Question = client.GetQuestion(ans.QuestionId,true,false).Body.StripHTML(),
                
Answer = ans.Body.StripHTML()};
  
tomasQA.Dump();
  
[/sourcecode]

 [1]: http://stackoverflow.com/
 [2]: http://stacky.codeplex.com/
 [3]: http://104.197.135.42/wp-content/uploads/2010/07/so1.jpg
 [4]: http://tomasp.net/
 [5]: https://odata.sqlazurelabs.com/OData.svc/v0.1/rp1uiewita/StackOverflow