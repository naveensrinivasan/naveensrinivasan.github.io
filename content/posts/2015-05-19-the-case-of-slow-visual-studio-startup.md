---
title: The case of slow Visual Studio startup
author: admin
type: post
date: 2015-05-20T01:23:35+00:00
url: /?p=1469
publicize_twitter_user:
  - snaveen
publicize_twitter_url:
  - http://t.co/JSLzj9zrX9
categories:
  - ETW
tags:
  - ETW
  - Perfview

---
In this post I would use Perfview /ETW to diagnose the delayed start-up of visual studio.

To analyze the problem start-up VS within Perfview as a run command

[gist https://gist.github.com/naveensrinivasan/5eb6406d6d38f2143acb]

This would launch visual studio and collect etw traces. I have also enabled CodeMarkers , which is ETW traces for Visual Studio in case if you want to trace any extensions performance.

[<img class="alignleft size-full wp-image-1474" src="https://naveensrinivasan.files.wordpress.com/2015/05/perfview-main.jpg" alt="perfview-main" width="660" height="353" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfview-main.jpg 1000w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfview-main-300x161.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfview-main-768x411.jpg 768w" sizes="(max-width: 660px) 100vw, 660px" />][1]After it completes I choose the CPU Stacks and filtered with devenv.exe process.

On the CPU window I choose call-tree tab which displays the threads.[<img class="alignleft size-full wp-image-1475" src="https://naveensrinivasan.files.wordpress.com/2015/05/perfviewcpuview.jpg" alt="perfviewcpuview" width="660" height="501" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfviewcpuview.jpg 961w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfviewcpuview-300x228.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/perfviewcpuview-768x583.jpg 768w" sizes="(max-width: 660px) 100vw, 660px" />][2]The most amount of time is spent on the start-up thread and that is what we want to zoom into.

[<img class="alignleft size-full wp-image-1477" src="https://naveensrinivasan.files.wordpress.com/2015/05/groupedcall-stacks.png" alt="groupedcall-stacks" width="660" height="426" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/groupedcall-stacks.png 835w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/groupedcall-stacks-300x194.png 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/groupedcall-stacks-768x496.png 768w" sizes="(max-width: 660px) 100vw, 660px" />][3]When I expanded it does not show the information and everything is grouped into OTHER which does not help me.

The reason for that is perfview groups call-stacks for better viewing. I cleared the &#8220;groupparts&#8221; textbox and then expanded the start-up thread.

[<img class="alignleft wp-image-1478 size-full" src="https://naveensrinivasan.files.wordpress.com/2015/05/actualcallstacks.jpg" alt="actualcallstacks" width="660" height="352" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/05/actualcallstacks.jpg 1366w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/actualcallstacks-300x160.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/actualcallstacks-768x409.jpg 768w, https://www.naveensrinivasan.com/wp-content/uploads/2015/05/actualcallstacks-1024x546.jpg 1024w" sizes="(max-width: 660px) 100vw, 660px" />][4]From the call-stacks I could make almost 42% of time is spent on Xamarin and DevExpress extensions within VS. Now I could turn them off and have a better performance.

Perfview is great tool for identifying where the time is being spent!

 [1]: https://naveensrinivasan.files.wordpress.com/2015/05/perfview-main.jpg
 [2]: https://naveensrinivasan.files.wordpress.com/2015/05/perfviewcpuview.jpg
 [3]: https://naveensrinivasan.files.wordpress.com/2015/05/groupedcall-stacks.png
 [4]: https://naveensrinivasan.files.wordpress.com/2015/05/actualcallstacks.jpg