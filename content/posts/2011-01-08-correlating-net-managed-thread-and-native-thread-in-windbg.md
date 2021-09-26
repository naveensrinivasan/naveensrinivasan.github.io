---
title: Correlating between .NET and native thread in Windbg
author: admin
type: post
date: 2011-01-09T01:46:06+00:00
url: /?p=1324
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168817;}'
categories:
  - Windbg

---
I recently saw a stackoverflow question where  someone wanted to know how they could correlate between  managed and native threads within Windbg.

Here is the managed thread object within the debugger

> 0:004> !do 02a1d6c4
  
> Name:        System.Threading.Thread
  
> MethodTable: 672e001c
  
> EEClass:     67018ed8
  
> Size:        48(0x30) bytes
  
> File:        C:WindowsMicrosoft.NetassemblyGAC\_32mscorlibv4.0\_4.0.0.0__b77a5c561934e089mscorlib.dll
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 672c8a78  4000720        4 &#8230;.Contexts.Context  0 instance 00000000 m_Context
  
> 672db4b8  4000721        8 &#8230;.ExecutionContext  0 instance 00000000 m_ExecutionContext
  
> 672df9fc  4000722        c        System.String  0 instance 02a1a220 m_Name
  
> 672dfed0  4000723       10      System.Delegate  0 instance 00000000 m_Delegate
  
> 672e63f4  4000724       14 &#8230;ation.CultureInfo  0 instance 00000000 m_CurrentCulture
  
> 672e63f4  4000725       18 &#8230;ation.CultureInfo  0 instance 00000000 m_CurrentUICulture
  
> 672df638  4000726       1c        System.Object  0 instance 00000000 m_ThreadStartArg
  
> <span style="color:#ff0000;"><strong>672daa7c  4000727       20        System.IntPtr  1 instance   542560 DONT_USE_InternalThread</strong></span>
  
> 672e29c8  4000728       24         System.Int32  1 instance        2 m_Priority
  
> 672e29c8  4000729       28         System.Int32  1 instance        3 m_ManagedThreadId
  
> 672cb76c  400072a      18c &#8230;LocalDataStoreMgr  0   shared   static s_LocalDataStoreMgr
  
> >> Domain:Value  0049f148:NotInit  <<
  
> 672ce328  400072b        c &#8230;alDataStoreHolder  0   shared TLstatic s_LocalDataStore
  
> >> Thread:Value <<

The present thread’s @$teb (Thread Environment Block) is <span style="color:#ff0000;"><strong>7efac000</strong></span>

0:004> ? @$teb Evaluate expression: 2130374656 = 7efac000

The DONT\_USE\_InternalThread is pointer to the native thread. Dumping the raw memory of the pointer should give us more information we are looking for.

> 0:004> dd poi(02a1d6c4+20)
  
> 00542560  67e9ee88 0000b220 00000000 056ef42c
  
> 00542570  00000000 00000000 00000000 00000003
  
> 00542580  00000000 00542588 00542588 00542588
  
> 00542590  00000000 00000000 baad0000 004a4f30
  
> 005425a0  <span style="color:#ff0000;"><strong>7efac000</strong></span> baadf00d 00000000 00000000
  
> 005425b0  00024dac 00000000 00000000 00000000
  
> 005425c0  00000000 baadf00d 00541ba0 00544290
  
> 005425d0  00544298 00000200 00544290 00544580

The pointer to teb is in the 40th offset of the  DONT\_USE\_InternalThread and here is the script that would get teb for each managed thread.

[sourcecode]
  
.foreach ($thread {!dumpheap -mt 672e001c -short}) { .if ( poi(${$thread}+20) != 0) {.printf "%p n",dwo(poi(${$thread}+20)+40)}}
  
[/sourcecode]

> 0:004> .foreach ($thread {!dumpheap -mt 672e001c -short}) { .if ( poi(${$thread}+20) != 0) {.printf &#8220;%p n&#8221;,dwo(poi(${$thread}+20)+40) }}
  
> 7efdd000
  
> 7efac000
  
> 7ef9a000
  
> 7ef97000
  
> 7ef2f000
  
> 7ef26000
  
> 7efd7000
  
> 7ef20000
  
> 7ef1d000
  
> 7ef1d000
  
> 7ef0e000
  
> 7efa3000
  
> 7ef2c000

So with the above we could dump the teb structure using dt ntdll!_TEB command. In the next post I will demonstrate how this can be used to debug some cool stuff 🙂<!--more-->