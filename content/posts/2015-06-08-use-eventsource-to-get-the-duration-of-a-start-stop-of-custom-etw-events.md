---
title: 'Use Eventsource  to get the duration of a Start Stop of Custom ETW events'
author: admin
type: post
date: 2015-06-08T19:53:29+00:00
url: /?p=1493
publicize_twitter_user:
  - snaveen
categories:
  - ETW
tags:
  - ETW

---
The <a href="https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.EventSource/" target="_blank">EventSource</a> library provides an option to get duration of Custom ETW start and stop events and when used with Perfview we could leverage this to stop tracing when the duration is more than what we expect.

What it is for example ,there could an external API call the application makes that has to be traced with the start and when it finishes then the stop of the event is called. Ideally we would have a ability to view the duration of these events similar to ASP.NET calls.  The EventSource Library along with Perfview provides this ability to view the duration between the start and stop events.

Here is a code sample with CustomEvent

[gist https://gist.github.com/naveensrinivasan/7e54c72dc628ae7da69e]

And here is the output from Perfview with the duration.

[<img class="alignleft wp-image-1495 size-full" src="https://naveensrinivasan.files.wordpress.com/2015/06/startstopetw.jpg" alt="StartStopETW" width="494" height="204" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/06/startstopetw.jpg 494w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/startstopetw-300x124.jpg 300w" sizes="(max-width: 494px) 100vw, 494px" />][1]

How often we want to capture trace when the performance of our custom event goes down to figure out what went wrong. This is very much possible with this.

Here is the command

PerfView /StopOnEtwEvent:*CustomEvent//Start;TriggerMSec=2000 collect

This would record the ETW events on a flight recorder mode and would stop when the CustomEvent took more than 2 seconds. This is one of the features I really like because it is a great asset to DevOps to see when the issue arises.

Here is an example of Perfview Stop reason that shows why perfview stopped which clearly  indicates when the duration of event took more than 2000 milliseconds.

[<img class="alignleft size-full wp-image-1496" src="https://naveensrinivasan.files.wordpress.com/2015/06/perfviewstopreason.jpg" alt="PerfviewStopReason" width="660" height="44" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/06/perfviewstopreason.jpg 897w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/perfviewstopreason-300x20.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/perfviewstopreason-768x51.jpg 768w" sizes="(max-width: 660px) 100vw, 660px" />][2]

There is a bug in perfview that would not record Stop triggered events. I have reported this and I hope this would be fixed in the next public release.

The source code for these samples are here

<a href="https://github.com/naveensrinivasan/ETWSamples" target="_blank">https://github.com/naveensrinivasan/ETWSamples</a>

 [1]: https://naveensrinivasan.files.wordpress.com/2015/06/startstopetw.jpg
 [2]: https://naveensrinivasan.files.wordpress.com/2015/06/perfviewstopreason.jpg