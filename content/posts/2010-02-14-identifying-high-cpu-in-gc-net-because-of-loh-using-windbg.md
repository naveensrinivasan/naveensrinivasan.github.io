---
title: Identifying High CPU in GC (.NET) because of LOH – using Windbg
author: admin
type: post
date: 2010-02-15T04:57:22+00:00
url: /?p=40
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422172701;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422172701;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422172701;}'
categories:
  - .NET
  - Windbg
tags:
  - GC

---
I am sure most of us are aware that one of the common reasons for High CPU usage .NET is because of, percentage time spent on GC is high. There are lot of write up about this. <a href="http://blogs.msdn.com/tess/archive/2006/06/22/643309.aspx" target="_blank">Tess</a> has amazing blog post  on this specifically, which explains in detail how to identify the symptoms. But one thing that I want share was the experience I had ,where in i could identify the real call stack which was causing allocation on LOH by have a break-point on the CLR Garbage collector itself. I am going to assume that you are aware of CLR GC and LOH and basic debugging using windbg.

FYI the Perfmon counter like “% Time spent in GC”, “Large Object Heap size” can identify we have an issue with allocation in LOH. But this only indicates there is a problem in High allocations that’s causing  GC to work hard and in turn causing high CPU usage . But does not point where the exact problem is .If there is a large code base and all of sudden if there is an issue like this it is hard to pinpoint where the root cause of the problem.

I am going to walk through an example on a button click the code allocates objects on LOH and identify it using windbg.

<span style="color:blue;">using </span>System;
  
<span style="color:blue;">using</span>System.Collections.Generic;
  
<span style="color:blue;">using</span>System.Linq;
  
<span style="color:blue;">using</span>System.Windows.Forms;

<span style="color:blue;">namespace</span>WindowsFormsApplication1
  
{
  
<span style="color:blue;">public partial class</span><span style="color:#2b91af;">Form1</span>: <span style="color:#2b91af;">Form<br /> </span>{
  
<span style="color:blue;">public</span><span style="color:#2b91af;">List</span><<span style="color:blue;">byte</span>[]> buffer = <span style="color:blue;">new</span><span style="color:#2b91af;">List</span><<span style="color:blue;">byte</span>[]>();

<p style="padding-left:30px;">
  <span style="color:blue;">public</span>Form1()<br /> {<br /> InitializeComponent();<br /> }
</p>

<p style="padding-left:30px;">
  <span style="color:blue;">private void</span>Button1Click(<span style="color:blue;">object</span>sender, <span style="color:#2b91af;">EventArgs </span>e)<br /> {<br /> AllocateinLOH();<br /> }
</p>

<p style="padding-left:30px;">
  <span style="color:blue;">void</span>AllocateinLOH()<br /> {<br /> <span style="color:blue;">var </span>b = <span style="color:blue;">new byte</span>[85001];<br /> b.ToList().ForEach(<br /> (x) => x = <span style="color:blue;">new byte</span>());
</p>

<p style="padding-left:30px;">
  buffer.Add(<span style="color:blue;">new byte</span>[85001]);<br /> }
</p>

}
  
}

[][1]In the above code AllocateinLOH will cause the heap allocations to LOH because the size is more than 85000 bytes. The next step is to launch the application and attach it to the debugger. To figure out the cause I could have got multiple memory dumps  and checked at every call stack when GC was happening . That is the harder way and we could be spending lot of time on that.

To get to solve the problem i always like to approach from the bottom of the stack. I was certain there should be a function within the CLR / GC which should be doing this and I was certain it has to be within MSCORWKS.dll. The next step was examine symbols using “x” command and here is the output

> 0:000> x mscorwks!wks::gc_heap::*
> 
> 000007fe\`e90d1340 mscorwks!WKS::gc\_heap::alloc\_allocated = <no type information>
  
> 000007fe\`e89fd380 mscorwks!WKS::gc\_heap::limit\_from_size = <no type information>
  
> 000007fe\`e8907e20 mscorwks!WKS::gc\_heap::fix\_older\_allocation\_area = <no type information>
  
> 000007fe\`e90d1310 mscorwks!WKS::gc\_heap::max\_free\_space\_items = <no type information>
  
> 000007fe\`e90cba38 mscorwks!WKS::gc\_heap::gc\_low = <no type information>
  
> 000007fe\`e8dae150 mscorwks!WKS::gc\_heap::grow\_brick\_card\_tables = <no type information>
  
> 000007fe\`e8a12d40 mscorwks!WKS::gc\_heap::fix\_generation_bounds = <no type information>
  
> 000007fe\`e8a13040 mscorwks!WKS::gc\_heap::fix\_large\_allocation\_area = <no type information>
  
> 000007fe\`e8a2c680 mscorwks!WKS::gc\_heap::allocate\_large_object = <no type information>
  
> 000007fe\`e90c0478 mscorwks!WKS::gc_heap::slow = <no type information>
  
> 000007fe\`e8ed8ec0 mscorwks!WKS::gc\_heap::c\_promote_callback = <no type information>

The above results are only partial because it was not necessary to get all the methods. The command “x mscorwks!wks::gc\_heap::\*” causes the debugger to get all the functions that are within the mscorwks dll with class name gc\_heap. I knew it is gc_heap because prior to this I issued the command “x mscorwks!\*gc*”. Because we don’t have private symbols for mscorwks we cannot see the type information. But we don’t need that for our purpose. The public symbols has FPO information for us to have break-point.

Form the above result the method name should be “allocate\_large\_object”

The next step was to put a break-point on the method

> bp mscorwks!wks::gc\_heap::allocate\_large_object &#8220;!CLRStack&#8221;

And then click button and here is the callstack output

> 0:004> g
  
> OS Thread Id: 0xf6f4 (0)
  
> Child-SP         RetAddr          Call Site
  
> 000000000027e000 000007ff00190684 WindowsFormsApplication1.Form1.AllocateinLOH()
  
> 000000000027e060 000007feeab00555 WindowsFormsApplication1.Form1.Button1Click(System.Object, System.EventArgs)
  
> 000000000027e090 000007feeb1f5873 System.Windows.Forms.Control.OnClick(System.EventArgs)
  
> 000000000027e0d0 000007feeb1aadf7 System.Windows.Forms.Button.OnMouseUp(System.Windows.Forms.MouseEventArgs)
  
> 000000000027e130 000007feeb702b7d System.Windows.Forms.Control.WmMouseUp(System.Windows.Forms.Message ByRef, System.Windows.Forms.MouseButtons, Int32)
  
> 000000000027e200 000007feeab49b5a System.Windows.Forms.Control.WndProc(System.Windows.Forms.Message ByRef)
  
> 000000000027e3b0 000007feeab49954 System.Windows.Forms.ButtonBase.WndProc(System.Windows.Forms.Message ByRef)
  
> 000000000027e430 000007feeab53aa6 System.Windows.Forms.Button.WndProc(System.Windows.Forms.Message ByRef)
  
> 000000000027e460 000007feeab53945 System.Windows.Forms.Control+ControlNativeWindow.WndProc(System.Windows.Forms.Message ByRef)
  
> 000000000027e4b0 000007feeab52244 System.Windows.Forms.NativeWindow.Callback(IntPtr, Int32, IntPtr, IntPtr)
  
> 000000000027e560 000007fee8aab07a DomainBoundILStubClass.IL_STUB(Int64, Int32, Int64, Int64)
  
> 000000000027e810 000007feeab6e883 DomainBoundILStubClass.IL_STUB(MSG ByRef)
  
> 000000000027e990 000007feeab6e0f8 System.Windows.Forms.Application+ComponentManager.System.Windows.Forms.
> 
> UnsafeNativeMethods.IMsoComponentManager.FPushMessageLoop(Int32, Int32, Int32)
  
> 000000000027ebe0 000007feeab6db65 System.Windows.Forms.Application+ThreadContext.RunMessageLoopInner(Int32, System.Windows.Forms.ApplicationContext)
  
> 000000000027ed30 000007ff00190171 System.Windows.Forms.Application+ThreadContext.RunMessageLoop(Int32, System.Windows.Forms.ApplicationContext)
  
> 000000000027ed90 000007fee8aad502 WindowsFormsApplication1.Program.Main()
  
> mscorwks!WKS::gc\_heap::allocate\_large_object:
  
> 000007fe\`e8a2c680 4053            push    rbx

Bingo now at the bottom of the stack we can see the “allocate\_large\_object” and on the top of the stack it is the managed code that we wrote “AllocateinLOH”.  Now we have solved the reason behind the High CPU usage in GC because of LOH.

The “allocate\_large\_object” is not documented by Microsoft as a public API and I don’t know whether the name would be same going forward from .NET framework 4.0.  This holds good until .NET framework 3.5. The idea behind this is just digging in to the framework can give us some information which has saved us valuable time and effort.

 [1]: http://11011.net/software/vspaste