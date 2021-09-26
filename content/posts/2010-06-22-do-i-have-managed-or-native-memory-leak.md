---
title: Do I have Managed or Native memory leak?
author: admin
type: post
date: 2010-06-22T22:38:52+00:00
url: /?p=868
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659084;}'
twitter_cards_summary_img_size:
  - 'a:7:{i:0;i:1437;i:1;i:851;i:2;i:2;i:3;s:25:"width="1437" height="851"";s:4:"bits";i:8;s:8:"channels";i:3;s:4:"mime";s:10:"image/jpeg";}'
categories:
  - Windbg

---
I noticed someone who couldn’t figure out the cause of memory leak in  managed application within the debugger. This person had basic debugging skills and was comfortable with sos.  FYI the leak wasn’t in the managed code, but in the native code. The managed code was using native code via PInvoke.

Here is how I figured out the cause. Every time I have to debug a memory leak in managed code ,the first command I run is !vmstat. The !vmstat is available in psscor2.dll for .net 3.5 and for .net 4.0 it is available in sos. This command provides summary of VM. The output is similar to the [VMMap][1] tool and here is the output from the command.

[<img class="alignnone size-full wp-image-869" title="vmstat" src="http://104.197.135.42/wp-content/uploads/2010/06/vmstat1.jpg" alt="" width="450" height="266" />][2]

Now that I know there is a memory leak, I always start of with running these commands

[sourcecode]

.shell -ci "!EEHeap -loader" findstr  "LoaderHeap"

.shell -ci "!EEHeap -gc" findstr /B "Total Size"

!heapstat  &#8211; iu

[/sourcecode]

And here is the output from the above commands

> 0:004> .shell -ci &#8220;!EEHeap -loader&#8221; findstr  &#8220;LoaderHeap&#8221;
  
> Total LoaderHeap size: 0x10000(65,536)bytes
  
> .shell: Process exited
  
> 0:004> .shell -ci &#8220;!EEHeap -gc&#8221; findstr /B &#8220;Total Size&#8221;
  
> Total Size   0x3df74(253,812)
  
> .shell: Process exited
  
> 0:004> !heapstat -iu
  
> Heap     Gen0         Gen1         Gen2         LOH
  
> Heap0    8204         182020       54820        8768
> 
> Free space:                                                 Percentage
  
> Heap0    12           149212       36           48          SOH: 60% LOH:  0%
> 
> Unrooted objects:                                           Percentage
  
> Heap0    1188         0            0            0           SOH:  0% LOH:  0%
  
> 0:004>

And from the output I know there isn’t a managed memory leak. The total heap size is only 253,812 bytes and my loader heap is 65,536 bytes, so it has to be native code.  Now I can focus my efforts on the native code debugging.

 [1]: http://technet.microsoft.com/en-us/sysinternals/dd535533.aspx
 [2]: http://104.197.135.42/wp-content/uploads/2010/06/vmstat1.jpg