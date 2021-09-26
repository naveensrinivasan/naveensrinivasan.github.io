---
title: Script to load sos within Windbg based on .NET Framework version
author: admin
type: post
date: 2010-07-26T04:33:21+00:00
url: /?p=1032
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168819;}'
categories:
  - SOS
  - Windbg

---
I often debug  .NET Framework v 2.0 / v 4.0 code within windbg. In v 2.0 the main clr dll was called &#8220;mscorwks.dll&#8221; and in v 4.0 it is called &#8220;clr.dll&#8221;.  As many of you are aware , to load sos in v 2.0 we would have to enter &#8220;.loadby sos mscorwks&#8221; and in v 4.0 it would be &#8220;.loadby sos clr&#8221; . This was a pain for me. Came up with a script to automate loading sos based on clr version

[sourcecode]
  
!for\_each\_module .if(($sicmp( "@#ModuleName" , "mscorwks") = 0) ) {.loadby sos mscorwks} .elsif ($sicmp( "@#ModuleName" , "clr") = 0) {.loadby sos clr}
  
[/sourcecode]

You can take it up a notch by setting a break-point within clr based on the .NET Framework version

[sourcecode]
  
!for\_each\_module .if(($sicmp( "@#ModuleName" , "mscorwks") = 0) ) {bp mscorwks!WKS::GCHeap::SuspendEE ".if (dwo(mscorwks!WKS::GCHeap::GcCondemnedGeneration)==2) {.echo start of gen 2}"} .elsif ($sicmp( "@#ModuleName" , "clr") = 0) {bp clr!WKS::GCHeap::SuspendEE ".if (dwo(clr!WKS::GCHeap::GcCondemnedGeneration)==2) {.echo start of gen 2}"}
  
[/sourcecode]