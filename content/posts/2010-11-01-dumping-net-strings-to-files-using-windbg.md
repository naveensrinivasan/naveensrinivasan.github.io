---
title: Dumping .NET strings to files using Windbg
author: admin
type: post
date: 2010-11-01T21:37:16+00:00
url: /?p=1128
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1361921316;}'
categories:
  - .NET
  - Windbg

---
In this post I would demonstrate how to dump strings from a memory dump /live process to a file. Recently I had to debug a process which had few big strings where I had to analyze its contents. The !dumpobj from sos would only dump partial strings.  I had to dump few hundred XML strings that I had to analyze using some automation. And hence comes the script.

[sourcecode]
  
$$ Dumps the managed strings to a file
  
$$ Platform x86
  
$$ Naveen Srinivasan http://naveensrinivasan.com
  
$$ Usage $$>a<"c:tempdumpstringtofolder.txt" 6544f9ac 5000 c:tempstringtest
  
$$ First argument is the string method table pointer
  
$$ Second argument is the Min size of the string that needs to be used filter
  
$$ the strings
  
$$ Third is the path of the file
  
.foreach ($string {!dumpheap -short -mt ${$arg1} -min ${$arg2}})
  
{ 

$$ MT Field Offset Type VT Attr Value Name
    
$$ 65452978 40000ed 4 System.Int32 1 instance 71117 m_stringLength
    
$$ 65451dc8 40000ee 8 System.Char 1 instance 3c m_firstChar
    
$$ 6544f9ac 40000ef 8 System.String 0 shared static Empty

$$ start of string is stored in the 8th offset, which can be inferred from above
    
$$ Size of the string which is stored in the 4th offset
    
r@$t0= poi(${$string}+4)*2
    
.writemem ${$arg3}${$string}.txt ${$string}+8 ${$string}+8+@$t0
  
}
  
[/sourcecode]

And to use the above script ,copy it to a file and invoke it within Windbg/cdb

> $$>a<&#8220;c:tempdumpstringtofolder.txt&#8221; 6544f9ac 5000 c:tempstringtest

Parameters to the script

  1. 6544f9ac :- Is the MT to string.
  2. 5000 :- Is the min size of the string that I want to dump
  3. c:tempstringtest :- Is the path along with partial filename for each string item

The dumped contents would be in Unicode format and to view its contents use something like this

[sourcecode language=&#8221;csharp&#8221;]

Console.WriteLine(ASCIIEncoding.Unicode.GetString(File.ReadAllBytes(@"c:tempstringtest03575270.txt")));

[/sourcecode]

And here is a sample code that downloads big xml strings ,that can be used by the above script to dump its contents to a folder

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
using System.Net;
  
namespace Test
  
{
      
class Program
      
{
          
static void Main(string[] args)
          
{
              
var speakers = new WebClient().DownloadString("http://www.codemash.org/rest/speakers");
              
var sessions = new WebClient().DownloadString("http://www.codemash.org/rest/sessions");
              
Console.Read();
          
}
      
}
  
}
  
[/sourcecode]

[twitter-follow screen\_name=&#8217;snaveen&#8217; show\_count=&#8217;yes&#8217; text_color=&#8217;00ccff&#8217;]</pre>