---
title: Function hit count using Pseudo-Register in Windbg
author: admin
type: post
date: 2010-02-13T01:41:01+00:00
url: /?p=30
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1404673871;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1404673871;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1404673871;}'
categories:
  - .NET
  - Windbg
tags:
  - Pseudo-Register
  - Windbg

---
What if we want to know the number of times a function was invoked. We can have “.echo” or “.printf” on break-point of a function and count the output manually. The better way to do this is using pseudo-registers.

In my <a href="http://naveensrinivasan.com/2010/02/10/conditional-breakpoint-in-net-using-windbg/" target="_blank">previous</a> post I had mentioned about alias inside the debugger. The debugger also provides User defined Pseudo-Registers for scripting inside the debugger. We can use them to manipulate values within our scripts. There are 20 of them from $t0,$t1.. $t19. These are pre-defined names that we cannot change.
  
The syntax to assign value to the register is “r”

> r @$t0 = 0

FYI the default value is 0.

> bp 000007ff0003c050  &#8221;  r @$t0 =@$t0 + 1;gc &#8220;

The above command will increment the $t0 register by one more, every time the break-point is hit. And gc is to go to next conditional break-point. To get the output of the register we can use the expression evaluator command “?”

> 0:004> ? @$t0
  
> Evaluate expression: 5 = 00000000\`00000005

There quite a few places we can use these inside the debugger. Example we could calculate the total size of multiple instances of an object type inside the debugger. That is we could get !objsize of each instance for type DataTable and then total it, to get the total memory consumption of datatable within our process.