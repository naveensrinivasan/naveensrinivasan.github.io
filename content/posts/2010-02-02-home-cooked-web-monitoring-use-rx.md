---
title: Home cooked web monitoring use Rx
author: admin
type: post
date: 2010-02-03T03:07:22+00:00
url: /?p=14
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422207289;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422207289;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422207289;}'
categories:
  - Reactive Extensions
tags:
  - Rx

---
One of the key things in software is to write succinct , declarative and asynchronous code. The answer to that is <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx" target="_blank">Reactive Extensions for .NET</a>&#160;

I have been digging into Rx for sometime now. Though there isn’t much of documentation I have been kind of successful in getting certain things done with it. 

One of requirement that came up from our Operations was to monitor some websites and notify someone if the site was not accessible. But the added constraints to this were check for the site status only at a certain interval and report failure only if it was more than certain percentage within a certain duration.

So it was like check the site status every 5 seconds , buffer the results for a minute and in the buffered response if it had more 3&#160; failures then tweet someone about it.

And here is the code to solve the problem

<pre class="code">(<span style="color:blue;">from </span>time <span style="color:blue;">in </span><span style="color:#2b91af;">Observable</span>.Interval(<span style="color:#2b91af;">TimeSpan</span>.FromSeconds(<span style="color:brown;">5</span>))
 <span style="color:blue;">let </span>req = <span style="color:#2b91af;">WebRequest</span>.Create(<span style="color:#a31515;">"http://www.nonexisting.com"</span>)
 <span style="color:blue;">from </span>res <span style="color:blue;">in </span><span style="color:#2b91af;">Observable</span>.
    FromAsyncPattern&lt;<span style="color:#2b91af;">WebResponse</span>&gt;(
        req.BeginGetResponse, req.EndGetResponse)()
 .Materialize()<span style="color:blue;">select </span>res).Buffer(<span style="color:blue;">new </span><span style="color:#2b91af;">TimeSpan</span>(<span style="color:brown;"></span>, <span style="color:brown;"></span>, <span style="color:brown;">1</span>, <span style="color:brown;"></span>)).
    Select(failed =&gt; 
        failed.Where(
        n =&gt; n.Kind == <span style="color:#2b91af;">NotificationKind</span>.OnError)).
    Where(failed =&gt; failed.Count() &gt; <span style="color:brown;">3</span>).
    Subscribe(x =&gt; Tweet(<span style="color:#a31515;">"Nonexisting.com Failed thrice"</span>));
<span style="color:#2b91af;">Console</span>.Read();</pre></p> </p> 

[][1][][1][][1][][1]

The “from time in Observable.Interval(TimeSpan.FromSeconds(5)” is for ensuring that an Observable is generated every 5 seconds to check the status of the website. In the next line the code creates a web request.

Using the FromAsyncPattern I was able to reduce all the plumbing code to handle async i/o calls for the web request.&#160; The materialize is for the sequence to continue even if there is an exception. Here is an good write up on <a href="http://bartdesmet.net/blogs/bart/archive/2009/12/29/more-linq-with-system-interactive-exploiting-the-code-data-relationship.aspx" target="_blank">Materialize</a>. And the rest is just the usual Linq where the code filters Notification type of error. 

This was a fun exercise. I will continue to explore Rx and blog about it.

 [1]: http://11011.net/software/vspaste