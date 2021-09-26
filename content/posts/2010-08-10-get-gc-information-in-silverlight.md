---
title: Get GC Information in Silverlight
author: admin
type: post
date: 2010-08-11T02:32:52+00:00
url: /?p=1050
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168819;}'
categories:
  - ETW
  - Silverlight

---
I had earlier written a [post][1] on getting GC information on Silverlight using ETW. With that we would have to write code to parse the ETW csv file.  In this post I am going to be using [Perfmonitor][2] to do this. This tools uses the same ETW under covers, but it does all the plumbing and gives a nice report , which is much easier to read.  Here are the reports

[<img class="alignnone size-full wp-image-1051" title="GC" src="http://104.197.135.42/wp-content/uploads/2010/08/gc1.jpg" alt="" width="700" height="222" />][3]

[<img class="alignnone size-full wp-image-1052" title="GC-1" src="http://104.197.135.42/wp-content/uploads/2010/08/gc-12.jpg" alt="" width="700" height="308" />][4]

To demonstrate this I used the bing’s world leader search page and here is the url

<http://www.bing.com/visualsearch?q=World+leaders&g=world_leaders&FORM=Z9GE74#>

Steps to get the GC information are

  1. Start a cmd or powershell  as admin , this required to collect ETW tracing
  2. Browse the above mentioned webpage using IE
  3. Issue the command “PerfMonitor.exe /process:4180 start” where 4180 is the internet explorer’s process id
  4. Do the necessary actions
  5. Then issue “PerfMonitor.exe stop”
  6. The command to get the report “PerfMonitor.exe GCTime”. This will generate a report and open it in the browser

Perfmonitor is like xperf for managed code. This is non-intrusive and can collect some valuable information in production. This is an xcopy tool and does not need an install.

 [1]: http://naveensrinivasan.com/2010/03/21/get-gc-information-in-silverlight-using-etw/
 [2]: http://bcl.codeplex.com/wikipage?title=PerfMonitor&referringTitle=Home
 [3]: http://104.197.135.42/wp-content/uploads/2010/08/gc1.jpg
 [4]: http://104.197.135.42/wp-content/uploads/2010/08/gc-12.jpg