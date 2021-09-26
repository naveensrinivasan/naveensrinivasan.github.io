---
title: 'Look ma I figured out why my ETW  EventSource isn’t tracing'
author: admin
type: post
date: 2015-05-05T13:47:39+00:00
url: /?p=1414
switch_like_status:
  - "1"
publicize_twitter_user:
  - snaveen
publicize_twitter_url:
  - http://t.co/kT0VJRVF3Q
categories:
  - .NET
  - ETW
tags:
  - .NET
  - ETW

---
The <a title="EventSource" href="https://msdn.microsoft.com/en-us/library/system.diagnostics.tracing.eventsource%28v=vs.110%29.aspx" target="_blank">EventSource </a>class in the framework 4.5 helps in writing custom ETW tracing.

When using EventSource class built within the framework, if the order of the methods don&#8217;t match ordinal number position in the class it would fail generating ETW traces. The EventSource has dependency on the order of the methods in the class.

This code would produce a valid ETW traces

[gist https://gist.github.com/naveensrinivasan/a1fcd0ec78d2473499d5]

This one would fail producing any ETW Traces.

[gist https://gist.github.com/naveensrinivasan/0247d4e644a3da2bff04]

The difference between them are the order of the methods. If you notice in the failing ETW tracing class the **FailedTraceEvent **is Second and the **FailedDetailedEvent **is first which is causing the trace not to be generated. The actual exception text would

> Event FailedDetailedEvent is givien event ID 1 but 2 was passed to WriteEvent.

It&#8217;s one of those quirks that I ran into when building ETW tracing.

**How to troubleshoot these kind of failures?**

By default these kind of failed ETW exceptions would not be raised to be handled by the client code. The reason being,in production if you enable ETW tracing and all of a sudden the application crashes would not be something that we would want.

To troubleshoot this, use ETW tracing for exceptions. Use ETW to trace custom ETW failures. How cool is this?  My choice of tool is <a title="Perfview" href="http://blogs.msdn.com/b/vancem/archive/tags/perfview/" target="_blank">Perfview.</a>

[<img class="alignleft wp-image-1424 size-large" src="https://naveensrinivasan.files.wordpress.com/2015/05/may-3-2015-etw-exception.jpg?w=660" alt="may-3-2015-etw-exception" width="660" height="107" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-etw-exception.jpg 1348w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-etw-exception-300x49.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-etw-exception-768x124.jpg 768w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-etw-exception-1024x166.jpg 1024w" sizes="(max-width: 660px) 100vw, 660px" />][1]

Within Perfview in the Events Window I enter **Test|Ex** in the filter text box which is shown in the above picture. FYI filter text box supports Regular expression. So by entering Test|Ex, I am filtering events with Test or Ex which for exceptions. With that information I could filter all the TestEvent&#8217;s and any exceptions that have been raised which shows ArgumentException.

The call-stack of the ArgumentException shows on the static contructor **.cctor** of  FailedEvent.

[<img class="alignleft wp-image-1434 size-large" src="https://naveensrinivasan.files.wordpress.com/2015/05/may-3-2015-exception-callstack.jpg?w=660" alt="may-3-2015-exception-callstack" width="660" height="366" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-exception-callstack.jpg 681w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/may-3-2015-exception-callstack-300x167.jpg 300w" sizes="(max-width: 660px) 100vw, 660px" />][2]

 [1]: https://naveensrinivasan.files.wordpress.com/2015/05/may-3-2015-etw-exception.jpg
 [2]: https://naveensrinivasan.files.wordpress.com/2015/05/may-3-2015-exception-callstack.jpg