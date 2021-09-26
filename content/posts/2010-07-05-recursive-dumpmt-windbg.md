---
title: Recursive !dumpmt – Windbg
author: admin
type: post
date: 2010-07-06T01:04:53+00:00
url: /?p=928
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659079;}'
categories:
  - Windbg

---
In this post I will be demonstrating how we could use CLR internal data-structures to recursively get the methodtable’s of an object and its base classes. The idea behind this is to understand the CLR data structure.

Here is the sample code

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
namespace ConsoleApplication
  
{
   
class Program : B
   
{
   
string test = "cw";
   
static void Main(string[] args)
   
{
   
var p = new Program();
   
Console.Read();
   
}
   
}
   
class B : A
   
{
   
public void TestB()
   
{
   
}
   
}
   
class A
   
{
   
public void TestA()
   
{
   
}
   
}
  
}
  
[/sourcecode]

The “Program” object address  is <span style="background-color:#ffff00;">0x0254bc38 </span> and here  is the output of !dumpobj

> 0:000> !do 0x0254bc38
  
> Name:        ConsoleApplication.Program
  
> MethodTable: <span style="background-color:#ffff00;">001c3904</span>
  
> EEClass:     001c1508
  
> Size:        12(0xc) bytes
  
> File:        C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 6335f9ac  4000001        4        System.String  0 instance 0254bc44 test

The method table pointer for the Program class is <span style="background-color:#ffff00;">001c3904 </span> . Let&#8217;s dump the raw memory instead of using !dumpobj

[sourcecode]
  
dd 0x0254bc38
  
[/sourcecode]

> 0:000> dd 0x0254bc38
  
> 0254bc38  <span style="background-color:#ffff00;">001c3904</span> 0254bc44 80000000 6335f9ac
  
> 0254bc48  00000002 00770063 00000000 00000000
  
> 0254bc58  63367490 00000000 00000000 00000000
  
> 0254bc68  00000000 00000000 00000000 6335f5e8
  
> 0254bc78  00000000 40010000 63366034 00000003
  
> 0254bc88  00000008 00000100 00000000 63366f40
  
> 0254bc98  00000000 00000000 00000000 00000000
  
> 0254bca8  00000001 0254bc80 00000001 00000000

Now that we can see the MethodTable pointer is the first field we can get the Methods by using

[sourcecode]

!dumpmt -md poi(0x0254bc38)

[/sourcecode]

> 0:000> !dumpmt -md poi(0x0254bc38)
  
> EEClass:      001c1508
  
> Module:       001c2e9c
  
> Name:         ConsoleApplication.Program
  
> mdToken:      02000004
  
> File:         C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> BaseSize:        0xc
  
> ComponentSize:   0x0
  
> Slots in VTable: 6
  
> Number of IFaces in IFaceMap: 0
  
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;
  
> MethodDesc Table
  
> Entry MethodDesc      JIT Name
  
> 6326a7e0   63044934   PreJIT System.Object.ToString()
  
> 6326e2e0   6304493c   PreJIT System.Object.Equals(System.Object)
  
> 6326e1f0   6304495c   PreJIT System.Object.GetHashCode()
  
> 632f1600   63044970   PreJIT System.Object.Finalize()
  
> 002200d0   001c38f0      JIT ConsoleApplication.Program..ctor()
  
> 00220070   001c38e4      JIT ConsoleApplication.Program.Main(System.String[])

So next time when we have an object we don&#8217;t have to go look for method table pointer address.

The goal is to get every method of class  Program, B, A and System.Object automatically. To get this, lets dump the raw memory of the method table

[sourcecode]
  
dd poi(0x0254bc38)
  
[/sourcecode]

> 0:000> dd poi(0x0254bc38)
  
> 001c3904  00080000 0000000c 00050011 00000004
  
> 001c3914  <span style="background-color:#ffff00;">001c3890 </span> 001c2e9c 001c3934 001c1508
  
> 001c3924  00000000 00000000 001c3854 002200d0
  
> 001c3934  00000080 00000000 00000000 00000000
  
> 001c3944  00000000 00000000 00000000 00000000
  
> 001c3954  00000000 00000000 00000000 00000000
  
> 001c3964  00000000 00000000 00000000 00000000
  
> 001c3974  00000000 00000000 00000000 00000000

The 10th offset contains the address of its base class method table pointer, So in the above output it is  <span style="background-color:#ffff00;">001c3890 </span>.

Now that we know it is the 10th offset, here is the script to get every method table for a class and its parents. FYI if the 10th offset is 00000000 then it means it is the super class which is System.Object.

[sourcecode]
  
r$t0 =poi(0x0254bc38);.while(@$t0) {!dumpmt -md @$t0;.echo \***\***\***\***\*****;r$t0=poi(@$t0+10)}
  
[/sourcecode]

And here is the explanation for the above script

  1. r$t0 =poi(0x0254bc38) &#8211; Using a pseudo register $t0 to assign the value of mt of the 0x0254bc38
  2. The .while loop will terminate when the value is 0. The  &#8220;!dumpmt -md @$t0&#8221; will dump the Method Table of the $t0 and the  &#8220;r$t0=poi(@$t0+10)&#8221; will reset $t0 its parent object method table.

Here is the partial output from the above script with method tables from Program and B

> 0:000> r$t0 =poi(0x0254bc38);.while(@$t0) {!dumpmt -md @$t0;.echo \***\***\***\***\*****;r$t0=poi(@$t0+10)}
  
> EEClass:      001c1508
  
> Module:       001c2e9c
  
> Name:         ConsoleApplication.Program
  
> mdToken:      02000004
  
> File:         C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> BaseSize:        0xc
  
> ComponentSize:   0x0
  
> Slots in VTable: 6
  
> Number of IFaces in IFaceMap: 0
  
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;
  
> MethodDesc Table
  
> Entry MethodDesc      JIT Name
  
> 6326a7e0   63044934   PreJIT System.Object.ToString()
  
> 6326e2e0   6304493c   PreJIT System.Object.Equals(System.Object)
  
> 6326e1f0   6304495c   PreJIT System.Object.GetHashCode()
  
> 632f1600   63044970   PreJIT System.Object.Finalize()
  
> 002200d0   001c38f0      JIT ConsoleApplication.Program..ctor()
  
> 00220070   001c38e4      JIT ConsoleApplication.Program.Main(System.String[])
  
> \***\***\***\***\*****
  
> EEClass:      001c149c
  
> Module:       001c2e9c
  
> Name:         ConsoleApplication.B
  
> mdToken:      02000003
  
> File:         C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> BaseSize:        0xc
  
> ComponentSize:   0x0
  
> Slots in VTable: 6
  
> Number of IFaces in IFaceMap: 0
  
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;
  
> MethodDesc Table
  
> Entry MethodDesc      JIT Name
  
> 6326a7e0   63044934   PreJIT System.Object.ToString()
  
> 6326e2e0   6304493c   PreJIT System.Object.Equals(System.Object)
  
> 6326e1f0   6304495c   PreJIT System.Object.GetHashCode()
  
> 632f1600   63044970   PreJIT System.Object.Finalize()
  
> 00220120   001c3888      JIT ConsoleApplication.B..ctor()
  
> 001cc02d   001c387c     NONE ConsoleApplication.B.TestB()
  
> \***\***\***\***\*****