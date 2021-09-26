---
title: Why isn’t the !bpmd in sos / windbg not working?
author: admin
type: post
date: 2010-12-06T00:16:53+00:00
url: /?p=1271
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168818;}'
categories:
  - .NET
  - SOS
  - SOSEX
  - Windbg

---
I recently noticed another blog <a title="http://bugslasher.net/2010/11/01/how-to-break-on-the-main-function-with-the-net-clr-4-0-and-windbg/" href="http://bugslasher.net/2010/11/01/how-to-break-on-the-main-function-with-the-net-clr-4-0-and-windbg/" target="_blank">post </a>refer to one of my <a title="Exploring undocumented SOS function in Windbg - .NET 4.0" href="http://naveensrinivasan.com/2010/02/25/exploring-undocumented-sos-function-in-windbg-net-4-0-2/" target="_blank">post</a>. The issue was, sos wasn&#8217;t enabling the break-points on non-jitted functions. The classic example being &#8220;Main&#8221;.  Thanks to <a title="http://www.stevestechspot.com/" href="http://www.stevestechspot.com/" target="_blank">Steve </a> I have been using sosex and not sos for setting break-points.

From my previous <a title="Exploring undocumented SOS function in Windbg - .NET 4.0" href="http://naveensrinivasan.com/2010/02/25/exploring-undocumented-sos-function-in-windbg-net-4-0-2/" target="_blank">post </a> you can understand how CLR is using clrn/CLRNotificationException to notify sos/sosex on JIT. With this information when I looked at the rotor <a title="dacnotify" href="http://www.koders.com/cpp/fidE4A1216FEC6BAA94CFA451D983819AE0CC78C073.aspx" target="_blank">code</a>, I noticed an interesting member variable &#8220;g_dacNotificationFlags&#8221;. So I decided to check the value of this variable when using !bpmd from sos and !mbm from sosex.

[sourcecode]

.if (dwo(mscorwks!g_dacNotificationFlags) = 0) {.echo bp not set } .else {.echo bp set}

[/sourcecode]

It was &#8220;0&#8221; when using sos and &#8220;1&#8221; when using sosex. Now I had to change the value to &#8220;1&#8221; and check if the break-point becomes active when using sos&#8217;s !bpmd.  FYI I don&#8217;t have private symbols and haven&#8217;t seen CLR Code. Here is the code to set the value to &#8220;1&#8221;.

[sourcecode]

ed mscorwks!g_dacNotificationFlags 00000001

[/sourcecode]

And not to my surprise the !bpmd seems to work for non-jitted function with the above hack. FYI we don&#8217;t have to resort to this to get !bpmd to work. If the !bpmd is set after load of mscorjit/clrjit it would work as expected.