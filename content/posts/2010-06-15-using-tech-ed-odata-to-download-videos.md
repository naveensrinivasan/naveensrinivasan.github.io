---
title: Using Tech-Ed OData to download videos
author: admin
type: post
date: 2010-06-15T12:20:32+00:00
url: /?p=2071
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659095;}'
categories:
  - linqpad
  - odata

---
I wanted to watch the Teched 2010 videos, but the problem I had was going to the site manually to download files for offline viewing.  And I was also interested only in Dev sessions which were level 300 / 400. Thanks to OData for teched <http://odata.msteched.com/sessions.svc/> ,I  could write 3 statements in linqpad and had them all downloaded using [wget][1]

[sourcecode language=&#8221;csharp&#8221;]
  
File.Delete(@"C:tempdownload.txt");

Sessions
  
.Where (s => (s.Level.StartsWith("400") ||  s.Level.StartsWith("300") ) && s.Code.StartsWith("DEV"))
  
.Take(10)
  
.ToList()
  
.Select (s => @"http://ecn.channel9.msdn.com/o9/te/NorthAmerica/2010/mp4/" + s.Code + ".mp4" )
  
.Run(s => File.AppendAllText(@"C:tempdownload.txt",s + Environment.NewLine));

Util.Cmd(@"wget.exe -b -i c:Tempdownload.txt",true);
  
[/sourcecode]</pre> 

Forgot to mention for the Run extension method is from Reactive Extensions

 [1]: http://gnuwin32.sourceforge.net/packages/wget.htm