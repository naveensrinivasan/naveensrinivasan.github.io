---
title: GC Start and Stop events in .NET using Windbg
author: admin
type: post
date: 2010-09-08T01:55:27+00:00
url: /?p=1064
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168819;}'
categories:
  - .NET
  - Windbg

---
I was recently showing someone the new ETW features in .NET especially the [GC Event notification][1] and I was asked if we can get this using Windbg.

So here is the sample code for the GC Collection

[sourcecode language=&#8221;csharp&#8221;]

namespace GCStartStop
  
{
  
public partial class Form1 : Form
  
{
  
public Form1()
  
{
  
InitializeComponent();
  
button1.Click += (s, b) => GC.Collect(2);
  
button1.Click += (s, b) => GC.Collect(1);
  
}
  
}
  
}
  
[/sourcecode]

The goal is set to a break-point only when the collection count is 2. Here is a bp script for doing this.

[sourcecode]

bp clr!WKS::GCHeap::SuspendEE ".if (dwo(clr!WKS::GCHeap::GcCondemnedGeneration)==2) {.echo start of gen 2;g} .else {gc}"

[/sourcecode]

The same thing can be done for clr!WKS::GCHeap::RestartEE.

When showing this to someone I was asked what does &#8220;EE&#8221; acronym in &#8220;SuspendedEE&#8221; ?  &#8220;EE&#8221; is  Execution Engine.

 [1]: http://msdn.microsoft.com/en-us/library/ff356162.aspx