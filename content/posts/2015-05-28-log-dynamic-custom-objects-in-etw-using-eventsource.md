---
title: Log dynamic Custom objects in ETW using EventSource
author: admin
type: post
date: 2015-05-29T03:35:50+00:00
url: /?p=1489
publicize_twitter_user:
  - snaveen
categories:
  - ETW
  - perfview
tags:
  - .NET
  - ETW

---
With the latest release of <a href="http://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.EventSource" target="_blank">EventSource</a> we could create dynamic events without having to create class that inherits from EventSource. This is will be not be good for Performance.

Using these methods we could either log Anonymous objects or Classes that have the EventData Attribute applied to it. The caveat is that these objects public properties alone will be serialized. These properties have to be of native types like string,int,datetime, guid , IEnumerable. If you don&#8217;t want a property to be serialized you could apply the attribute EventIgnore.

The source code for this repository is in <a href="https://github.com/naveensrinivasan/ETWSamples" target="_blank">https://github.com/naveensrinivasan/ETWSamples</a>

Here is the sample code of using  Dynamic eventsource to generate ETW traces

[gist https://gist.github.com/naveensrinivasan/83ded09f7d754ad0b3a8]

Here is the trace from Perfview generated using

[gist https://gist.github.com/naveensrinivasan/11a793b35a18fc9546dd]

[<img class=" size-full wp-image-1490 aligncenter" src="https://naveensrinivasan.files.wordpress.com/2015/05/dynamicetw.jpg" alt="DynamicETW" width="660" height="171" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/dynamicetw.jpg 929w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/dynamicetw-300x78.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/dynamicetw-768x199.jpg 768w" sizes="(max-width: 660px) 100vw, 660px" />][1]

 [1]: https://naveensrinivasan.files.wordpress.com/2015/05/dynamicetw.jpg