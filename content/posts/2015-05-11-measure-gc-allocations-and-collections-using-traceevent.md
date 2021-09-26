---
title: Measure GC Allocations and Collections using TraceEvent
author: admin
type: post
date: 2015-05-12T01:14:34+00:00
url: /?p=1437
publicize_twitter_user:
  - snaveen
publicize_twitter_url:
  - http://t.co/YLFuW30x66
categories:
  - .NET
  - ETW
tags:
  - ETW
  - TraceEvent

---
In this post I will explore  how we could use <a href="https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent/" target="_blank">TraceEvent</a> to measure our code (even at function level) for GC Allocations and Collections.

<div id='gallery-1' class='gallery galleryid-1437 gallery-columns-1 gallery-size-full'>
  <figure class='gallery-item'> 
  
  <div class='gallery-icon landscape'>
    <img width="400" height="400" src="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/showmethecode.jpg" class="attachment-full size-full" alt="" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/showmethecode.jpg 400w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/showmethecode-150x150.jpg 150w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/showmethecode-300x300.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/showmethecode-100x100.jpg 100w" sizes="(max-width: 400px) 100vw, 400px" />
  </div></figure>
</div>

Save this with &#8220;.linq&#8221; extension and then open in  linqpad.

[gist https://gist.github.com/naveensrinivasan/b72fd80876eb67557ae8]

Here is the TL;DR

Why would I want to know GC events on a function level? Doesn&#8217;t the PerfMon counter  provide that information on an application level? Isn&#8217;t Premature optimization root of all evil?

Yes, for most of the part Premature optimization is not necessary. And PerfMon GC counter&#8217;s would give answers for the whole application. But it is usually after we build the application and when we start running into performance issues we start looking at them.

The motivation behind this are two things <a href="https://msdn.microsoft.com/en-us/magazine/cc500596.aspx" target="_blank">Measure Early and Often for Performance</a> and <a href="http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/DEV-B333" target="_blank">Essential Truths Everyone Should Know about Performance in a Large Managed Codebase</a>

If you haven&#8217;t read or watched the above video please do it. It let&#8217;s us know why and how to do it.

In the above video Dustin talks about Roslyn code base and how they used <a href="http://blogs.msdn.com/b/vancem/archive/tags/perfview/" target="_blank">Perfview </a>to measure Roslyn code base and identify potential bottlenecks early to avoid Perf issues.

One of the key differences between managed code and native code with respect to performance is GC. If GC is working hard then your application might not be able to get the performance that you are expecting. GC is good but if we don&#8217;t know which calls allocate what amount of data then it is an issue. Especially if you have a section code that is hit very often and which requires a lot of Perf, it is good know where the allocations are coming from. It is not explicit always.

In the above video Dustin shows few examples of Roslyn code where they were able to identify subtle issues that could allocate a lot when you are trying to get the most out of the code.There is also <a href="https://github.com/mjsabby/RoslynClrHeapAllocationAnalyzer" target="_blank">Roslyn Heap Allocation Analyzer</a> which looks at the code help us identify allocations which isn&#8217;t necessary. It is a cool project.

I took one of the examples from the video as a motivation to check if  I could measure and make it a utility in my toolbox to help me when I need one.

[gist https://gist.github.com/naveensrinivasan/9153a580b7f4bda55a38]

In the above example I am trying look for a word &#8220;pede&#8221; in the lorem ipsum text. The code could get it using &#8220;foreach&#8221; or using the &#8220;Any&#8221; operator. I would like to run this few times to check what are the allocations and how long does it take. I used LINQPad as a scratch pad.

Here is the result of GC Allocations of using &#8220;Any&#8221; for 500 iterations and NOT the foreach

[<img class="alignleft wp-image-1440 size-large" src="https://naveensrinivasan.files.wordpress.com/2015/05/gcallocations.jpg?w=660" alt="GCAllocations" width="660" height="272" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gcallocations.jpg 675w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gcallocations-300x124.jpg 300w" sizes="(max-width: 660px) 100vw, 660px" />][1]

The were 118 allocations of Enumerator and 146 allocations Func. GC usually allocates 100K each time it allocates that&#8217;s what is shown in the allocation amount column.

And here is GC Allocations when using &#8220;foreach&#8221;

[<img class="alignleft size-full wp-image-1445" src="https://naveensrinivasan.files.wordpress.com/2015/05/gcallocationwithforeach.jpg" alt="GCAllocationWithForEach" width="472" height="237" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gcallocationwithforeach.jpg 472w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gcallocationwithforeach-300x151.jpg 300w" sizes="(max-width: 472px) 100vw, 472px" />][2]

There are hardly any new allocations compared to the previous one.

Here is the GC Collections when using  &#8220;Any&#8221;

[<img class="alignleft size-full wp-image-1447" src="https://naveensrinivasan.files.wordpress.com/2015/05/gccollection.jpg" alt="GCCollection" width="660" height="395" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gccollection.jpg 704w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/gccollection-300x179.jpg 300w" sizes="(max-width: 660px) 100vw, 660px" />][3]

There were 18 GC Collections using Any.

Here it is using foreach  and there were 0 collections.

[<img class="alignleft size-full wp-image-1448" src="https://naveensrinivasan.files.wordpress.com/2015/05/zeorcollections.jpg" alt="ZeorCollections" width="229" height="65" />][4]

Here is measure it time duration results using Any

[<img class="alignleft size-full wp-image-1449" src="https://naveensrinivasan.files.wordpress.com/2015/05/measurewithany.jpg" alt="MeasureWithAny" width="660" height="223" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/measurewithany.jpg 759w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/measurewithany-300x102.jpg 300w" sizes="(max-width: 660px) 100vw, 660px" />][5]

And here it is using Foreach

[<img class="alignleft size-full wp-image-1450" src="https://naveensrinivasan.files.wordpress.com/2015/05/measurewithforeach.jpg" alt="MeasureWithForEach" width="660" height="253" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/measurewithforeach.jpg 744w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/measurewithforeach-300x115.jpg 300w" sizes="(max-width: 660px) 100vw, 660px" />][6]

 [1]: https://naveensrinivasan.files.wordpress.com/2015/05/gcallocations.jpg
 [2]: https://naveensrinivasan.files.wordpress.com/2015/05/gcallocationwithforeach.jpg
 [3]: https://naveensrinivasan.files.wordpress.com/2015/05/gccollection.jpg
 [4]: https://naveensrinivasan.files.wordpress.com/2015/05/zeorcollections.jpg
 [5]: https://naveensrinivasan.files.wordpress.com/2015/05/measurewithany.jpg
 [6]: https://naveensrinivasan.files.wordpress.com/2015/05/measurewithforeach.jpg