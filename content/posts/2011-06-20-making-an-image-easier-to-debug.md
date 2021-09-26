---
title: Making an Image Easier to Debug
author: admin
type: post
date: 2011-06-20T17:45:24+00:00
url: /?p=1368
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";s:1:"0";s:6:"author";s:8:"11476521";s:7:"blog_id";s:8:"11168708";s:9:"mod_stamp";s:19:"2011-06-20 17:45:57";}'
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168816;}'
categories:
  - .NET
  - Security
  - Windbg

---
I am doing security review for a managed application which is obfuscated. So I am doing a lot of   disassembling code at runtime using Windbg. One of the issues is that code gets JIT optimized because of the retail build. This makes it harder for me debug when mapping it back. Realized  that I could turnoff  JIT Optimization&#8217;s using the ini file.

[sourcecode]
  
[.NET Framework Debugging Control]
  
GenerateTrackingInfo=1
  
AllowOptimize=0
  
[/sourcecode]

Another use of <a href="http://msdn.microsoft.com/en-us/library/9dd8z24x.aspx" target="_blank">feature </a>which I guess wasn&#8217;t really intended for.

<pre></pre>