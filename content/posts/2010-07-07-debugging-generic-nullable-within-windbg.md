---
title: Debugging Generic System.Nullable within Windbg
author: admin
type: post
date: 2010-07-08T03:58:51+00:00
url: /?p=947
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422207766;}'
categories:
  - SOS
  - SOSEX
  - Windbg

---
In this post I am going to unravel the mystery of debugging the Nullable<T> within Windbg in .NET 3.5 and also compare it with .NET 4.0. Here is the sample code and it is compiled in .NET 3.5

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
namespace ConsoleApplication
  
{
   
class Program
   
{
   
Int32? test;
   
int i = 10;
   
static void Main(string[] args)
   
{
   
Nullable<T>
   
Int32? i = 10;
   
Object o = 10;
   
var p = new Program() { test = 20 };
   
Console.Read();
   
p.test = (Int32?) o ;
   
Console.WriteLine(p.test.HasValue);
   
}
   
}
  
}
  
[/sourcecode]

Attached to the debugger on the Console.Read. FYI I always load sos and sosex extensions to debug managed code. Here is !mdt 0x0253c11c output

> 0:000> !mdt 0x0253c11c
  
> 0253c11c (ConsoleApplication.Program)
  
> test:ERROR (0x80070057).
  
> i:0xa (System.Int32)

Notice that &#8220;test&#8221; does not have a value and has an error. Next issued !dumpobj

[sourcecode]
  
!do 0x0253c11c
  
[/sourcecode]

> 0:000> !do 0x0253c11c
  
> Name: ConsoleApplication.Program
  
> MethodTable: 002932f0
  
> EEClass: 00291360
  
> Size: 20(0x14) bytes
  
> (C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> <span style="background-color:#ffff00;">00000000</span> 4000001        8                       1 instance 0280c124 test
  
> 7776ab0c  4000002        4         System.Int32  1 instance       10 i

My fault ,I thought sos should be able to get the MethodTable of Nullable<Int32> for &#8220;test&#8221; when sosex couldn&#8217;t.  To my surprise the MT output was <span style="background-color:#ffff00;">00000000</span> . To view the contents of the &#8220;test&#8221; I would have to use  the !dumpvc which requires methodtable. I know I could use the dd command. And here is the output from the dd 0280c124

> 0:000> dd 0280c124
  
> 0280c124  00000001 <span style="background-color:#ffff00;">00000014</span> 00000000 77767c70
  
> 0280c134  00000000 00000000 00000000 00000000
  
> 0280c144  00000000 00000000 777684dc 00000000
  
> 0280c154  40010000 7776d7ec 00000003 00000008
  
> 0280c164  00000100 00000000 77767cc4 00000000
  
> 0280c174  00000000 00000000 00000000 00000001
  
> 0280c184  0280c158 00000001 00000000 7776841c
  
> 0280c194  00000000 00000000 00000000 00000000

The second field <span style="background-color:#ffff00;">00000014</span> is the actual value of test and here is the actual output

> 0:000> ? poi(0280c124+0x4)
  
> Evaluate expression: 20 = 00000014

But this does not solve the real issue of figuring out the methodtable to use it in !dumpvc. I could have used !mx System.Nullable* to get the MethodTable,  because I knew the type is Nullable<Int> ,what if I didn&#8217;t know the type information.

To get the mt information I had to disassemble the code. First step is to get the !clrstack

> 0:000> !CLRStack
  
> OS Thread Id: 0x94c (0)
  
> ESP       EIP
  
> 0021f1dc 769d73ea [NDirectMethodFrameStandaloneCleanup: 0021f1dc] System.IO.__ConsoleStream.ReadFile(Microsoft.Win32.SafeHandles.SafeFileHandle, Byte*, Int32, Int32 ByRef, IntPtr)
  
> 0021f1f8 77c8ae67 System.IO.__ConsoleStream.ReadFileNative(Microsoft.Win32.SafeHandles.SafeFileHandle, Byte[], Int32, Int32, Int32, Int32 ByRef)
  
> 0021f224 77c8ad86 System.IO.__ConsoleStream.Read(Byte[], Int32, Int32)
  
> 0021f244 776f9fbb System.IO.StreamReader.ReadBuffer()
  
> 0021f258 77c677fc System.IO.StreamReader.Read()
  
> 0021f264 77c8dd81 System.IO.TextReader+SyncTextReader.Read()
  
> 0021f270 77bd328b System.Console.Read()
  
> 0021f278 <span style="background-color:#ffff00;">00320115</span> ConsoleApplication.Program.Main(System.String[])
  
> 0021f4d0 59781b6c [GCFrame: 0021f4d0]

The next is to !u 00320115 and here is the partial ouput

> 00320116 8b45d8          mov     eax,dword ptr [ebp-28h]
  
> 00320119 3a4008          cmp     al,byte ptr [eax+8]
  
> 0032011c 8d4008          lea     eax,[eax+8]
  
> 0032011f 8945c8          mov     dword ptr [ebp-38h],eax
  
> 00320122 ff75dc          push    dword ptr [ebp-24h]
  
> 00320125 8b4dc8          mov     ecx,dword ptr [ebp-38h]
  
> 00320128 bae8397777      mov     edx,offset mscorlib_ni+0x2739e8 (<span style="background-color:#ffff00;">777739e8</span>) (MT: System.Nullable\`1[[System.Int32, mscorlib]])
  
> 0032012d e8d6a74c59      call    mscorwks!JIT\_Unbox\_Nullable (597ea908)

Notice the Method table <span style="background-color:#ffff00;">777739e8</span> for System.Nullable\`1[[System.Int32, mscorlib]] and here is the output from !dumpmt -md 777739e8

> 0:000> !dumpmt -md 777739e8
  
> EEClass: 7752e7c8
  
> Module: 77501000
  
> Name: System.Nullable\`1[[System.Int32, mscorlib]]
  
> mdToken: 0200026d  (C:WindowsassemblyGAC\_32mscorlib2.0.0.0\__b77a5c561934e089mscorlib.dll)
  
> BaseSize: 0x10
  
> ComponentSize: 0x0
  
> Number of IFaces in IFaceMap: 0
  
> Slots in VTable: 14
  
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;
  
> MethodDesc Table
  
> Entry MethodDesc      JIT Name
  
> 77697028   775eace0     NONE System.Nullable\`1[[System.Int32, mscorlib]].ToString()
  
> 77697020   775eacc0     NONE System.Nullable\`1[[System.Int32, mscorlib]].Equals(System.Object)
  
> 77697018   775eacd0     NONE System.Nullable\`1[[System.Int32, mscorlib]].GetHashCode()
  
> 777374c0   775412a4   PreJIT System.Object.Finalize()
  
> 77d010a0   775eac98   PreJIT System.Nullable\`1[[System.Int32, mscorlib]]..ctor(Int32)
  
> 77d01100   775eaca0   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].get_HasValue()
  
> 77d01120   775eaca8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].get_Value()
  
> 77d01030   775eacb0   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].GetValueOrDefault()
  
> 77d01140   775eacb8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].GetValueOrDefault(Int32)
  
> 77d0100c   775eacf0   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].op_Implicit(Int32)
  
> 77d00fe8   775eacf8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].op_Explicit(System.Nullable\`1<Int32>)
  
> 77d01040   775eacc8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].Equals(System.Object)
  
> 77d01078   775eacd8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].GetHashCode()
  
> 77d010c0   775eace8   PreJIT System.Nullable\`1[[System.Int32, mscorlib]].ToString()

Now that I have confirmed the mt and here is the output from !dumpvc 777739e8 0280c124

> 0:000> !dumpvc 777739e8 0280c124
  
> Name: System.Nullable\`1[[System.Int32, mscorlib]]
  
> MethodTable 777739e8
  
> EEClass: 7752e7c8
  
> Size: 16(0x10) bytes
  
> (C:WindowsassemblyGAC\_32mscorlib2.0.0.0\__b77a5c561934e089mscorlib.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 7776eadc  40009a8        0       System.Boolean  1 instance        1 hasValue
  
> 7776ab0c  40009a9        4         System.Int32  1 instance       20 value

I decided to test this in .NET 4.0 and here is the output of !mdt for the Program object

> 0:000> !mdt 0x0236bc44
  
> 0236bc44 (ConsoleApplication.Program)
  
> test:(System.Nullable\`1[[System.Int32, mscorlib]]) VALTYPE (MT=6229f60c, ADDR=0236bc50)
  
> i:0xa (System.Int32)

Notice the &#8220;test&#8221; which is Nullable<Int32> is now recognized by sosex and it also provides the method table.