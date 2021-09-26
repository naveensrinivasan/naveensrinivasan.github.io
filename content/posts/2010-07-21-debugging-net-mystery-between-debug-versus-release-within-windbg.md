---
title: Debugging .NET – mystery between DEBUG versus RELEASE within windbg
author: admin
type: post
date: 2010-07-22T02:48:17+00:00
url: /?p=1015
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1431702071;}'
categories:
  - .NET
  - Windbg

---
I am sure most of us have debugged applications that are build with debug turned on, which is obviously much easier compared to debugging release build (optimized code). In this post I am going to share one of my experiences of debugging release build code. I will demonstrate this with a simple Console Application.

Here is the code

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
namespace ConsoleApplication
  
{
  
class Program
  
{
  
static void Main(string[] args)
  
{
  
var x = 10;
  
var name = "naveen";
  
Console.WriteLine(name);
  
Console.Read();
  
}
  
}
  
}
  
[/sourcecode]

I compiled it under release mode and launched it within the debugger. The goal is to have a break-point on Console.WriteLine(name). First thing was to set up a sx notification on load for mscorlib.

[sourcecode]

sxe ld:mscorlib

[/sourcecode]

And when the break-point hits for the above event, then issued the following command to load sosex , sos and set an bp on Console.WriteLine which is nothing fancy

[sourcecode]

.load sosex;.loadby sos clr;!mbm \*System.Console.WriteLine\* "!mk";g

[/sourcecode]

I would imagine that the break-point would hit and I would get a call-stack, but to my surprise this was the output

> 0:000> .load sosex;.loadby sos clr;!mbm \*System.Console.WriteLine\* &#8220;!mk&#8221;;g
  
> The breakpoint could not be resolved immediately.
  
> Further attempts will be made as modules are loaded.
  
> (1090.11fc): CLR notification exception &#8211; code e0444143 (first chance)
  
> Breakpoint set at System.Console.WriteLine().
  
> Breakpoint set at System.Console.WriteLine(Boolean).
  
> Breakpoint set at System.Console.WriteLine(Char).
  
> Breakpoint set at System.Console.WriteLine(Char[]).
  
> Breakpoint set at System.Console.WriteLine(Char[], Int32, Int32).
  
> Breakpoint set at System.Console.WriteLine(System.Decimal).
  
> Breakpoint set at System.Console.WriteLine(Double).
  
> Breakpoint set at System.Console.WriteLine(Single).
  
> Breakpoint set at System.Console.WriteLine(Int32).
  
> Breakpoint set at System.Console.WriteLine(UInt32).
  
> Breakpoint set at System.Console.WriteLine(Int64).
  
> Breakpoint set at System.Console.WriteLine(UInt64).
  
> Breakpoint set at System.Console.WriteLine(System.Object).
  
> Breakpoint set at System.Console.WriteLine(System.String).
  
> Breakpoint set at System.Console.WriteLine(System.String, System.Object).
  
> Breakpoint set at System.Console.WriteLine(System.String, System.Object, System.Object).
  
> Breakpoint set at System.Console.WriteLine(System.String, System.Object, System.Object, System.Object).
  
> Breakpoint set at System.Console.WriteLine(System.String, System.Object, System.Object, System.Object, System.Object, &#8230;).
  
> Breakpoint set at System.Console.WriteLine(System.String, System.Object[]).
  
> (1090.11fc): CLR notification exception &#8211; code e0444143 (first chance)

That&#8217;s it. And I never got an hit for the break-point. I checked to make sure there was an actual breakpoint set by issuing a &#8220;bl&#8221; command. I could see there were break-points for Console.WriteLine. The next step was to  disassemble the code. So got the instruction pointer from the !mk call-stack. Here is the output of !mk. FYI this is when the code is blocked on Console.Read

> 00:U 003def90 75d273ea KERNEL32!ReadConsoleInternal+0x15
  
> 01:U 003def98 75d27041 KERNEL32!ReadConsoleA+0x40
  
> 02:U 003df020 75caf489 KERNEL32!ReadFileImplementation+0x75
  
> 03:M 003df068 65651c8b DomainNeutralILStubClass.IL\_STUB\_PInvoke(Microsoft.Win32.SafeHandles.SafeFileHandle, Byte*, Int32, Int32 ByRef, IntPtr)(+0x0 IL)(+0x0 Native)
  
> 04:M 003df0e8 65cbf7e8 System.IO.\_\_ConsoleStream.ReadFileNative(Microsoft.Win32.SafeHandles.SafeFileHandle, Byte[], Int32, Int32, Int32, Int32 ByRef)(+0x53 IL)(+0x8c Native) [f:ddndpclrsrcBCLSystemIO\_\_ConsoleStream.cs, @ 16707566,0]
  
> 05:M 003df110 65cbf6d0 System.IO.\_\_ConsoleStream.Read(Byte[], Int32, Int32)(+0x5d IL)(+0x9c Native) [f:ddndpclrsrcBCLSystemIO\_\_ConsoleStream.cs, @ 131,13]
  
> 06:M 003df138 65608bfb System.IO.StreamReader.ReadBuffer()(+0xa0 IL)(+0x3b Native) [f:ddndpclrsrcBCLSystemIOStreamReader.cs, @ 488,21]
  
> 07:M 003df154 65bcacc3 System.IO.StreamReader.Read()(+0x1b IL)(+0x23 Native) [f:ddndpclrsrcBCLSystemIOStreamReader.cs, @ 302,17]
  
> 08:M 003df160 65cc5e9d System.IO.TextReader+SyncTextReader.Read()(+0x0 IL)(+0x19 Native) [f:ddndpclrsrcBCLSystemIOTextReader.cs, @ 244,17]
  
> 09:M 003df170 <span style="background-color:#ffff00;">0066009a</span> \*** WARNING: Unable to verify checksum for ConsoleApplication.exe
  
> ConsoleApplication.Program.Main(System.String[])(+0x0 IL)(+0x2a Native) [c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs, @ 10,13]
  
> 0a:U 003df17c 661621db clr!CallDescrWorker+0x33

Next disassemble Main Method using the ip which is 0066009a

[sourcecode]

!u 0066009a

[/sourcecode]

Here is the output

> 0:000> !u 0066009a
  
> Normal JIT generated code
  
> ConsoleApplication.Program.Main(System.String[])
  
> Begin 00660070, size 2d
  
> c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs @ 10:
  
> 00660070 55 push ebp
  
> 00660071 8bec mov ebp,esp
  
> 00660073 56 push esi
  
> 00660074 8b3530206b03 mov esi,dword ptr ds:\[36B2030h\] (&#8220;naveen&#8221;)
  
> c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs @ 11:
  
> 0066007a e85170f864 call mscorlib_ni+0x2570d0 (655e70d0) (<span style="background-color:#ffff00;">System.Console.get_Out()</span>, mdToken: 060008cd)
  
> 0066007f 8bc8 mov ecx,eax
  
> 00660081 8bd6 mov edx,esi
  
> 00660083 8b01 mov eax,dword ptr [ecx]
  
> 00660085 8b403c mov eax,dword ptr [eax+3Ch]
  
> 00660088 ff5010 call dword ptr [eax+10h]
  
> 0066008b e8f0a55565 call mscorlib\_ni+0x82a680 (65bba680) (System.Console.get\_In(), mdToken: 060008cc)
  
> 00660090 8bc8 mov ecx,eax
  
> 00660092 8b01 mov eax,dword ptr [ecx]
  
> 00660094 8b402c mov eax,dword ptr [eax+2Ch]
  
> 00660097 ff500c call dword ptr [eax+0Ch]
  
> c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs @ 13:
  
> >>> 0066009a 5e pop esi
  
> 0066009b 5d pop ebp
  
> 0066009c c3 ret

And I see System.Console.get_Out instead of System.Console.WriteLine which I was totally surprised. This was  the reason the break-point never hit. Next I wanted check the IL which was compiled , what we see above is jitted x86 mixed with IL. Here is the command to check the compiled IL. First I had to get the methodesc from the ip using !ip2md

[sourcecode]

!ip2md 0066009a

[/sourcecode]

> 0:000> !ip2md 0066009a
  
> MethodDesc: <span style="background-color:#ffff00;">002237f0</span>
  
> Method Name: ConsoleApplication.Program.Main(System.String[])
  
> Class: 002213f8
  
> MethodTable: 00223804
  
> mdToken: 06000001
  
> Module: 00222e9c
  
> IsJitted: yes
  
> CodeAddr: 00660070
  
> Transparency: Critical
  
> Source file: c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs @ 13

Here is from the methoddesc to IL

[sourcecode]

!dumpil 002237f0

[/sourcecode]

> 0:000> !dumpil 002237f0
  
> ilAddr = 012a2050
  
> IL_0000: ldstr &#8220;naveen&#8221;
  
> IL_0005: stloc.0
  
> IL_0006: ldloc.0
  
> IL_0007: call System.Console::WriteLine
  
> IL_000c: call System.Console::Read
  
> IL_0011: pop
  
> IL_0012: ret

Which looks very similar to my C# code. So looks like CLR optimized the code ,converted the Console.WriteLine to Console.get_Out. To validate it restarted the app with this as the command for break-point

[sourcecode]
  
.load sosex;.loadby sos clr;!mbm \*System.Console.get_Out\* "!mk";g
  
[/sourcecode]

And here is the output

> 00:M 0032ed58 655e70d1 <span style="background-color:#ffff00;">System.Console.get_Out()</span>(+0x0 IL)(+0x1 Native) [f:ddndpclrsrcBCLSystemConsole.cs, @ 193,17]
  
> 01:M 0032ed60 003a007f \*** WARNING: Unable to verify checksum for ConsoleApplication.exe
  
> ConsoleApplication.Program.Main(System.String[])(+0x6 IL)(+0xf Native) [c:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication11Program.cs, @ 11,13]
  
> 02:U 0032ed6c 661621db clr!CallDescrWorker+0x33

Now that I have solved this I wanted to check the same on the debug build (optimized -) . To validate if it was Console.get_Out or Console.WriteLine. So when mscorlib loaded here was my command to check this

[sourcecode]
  
.load sosex;.loadby sos clr;!mbm \*Program.Main\* "!u @eip";g
  
[/sourcecode]

In the above command I am setting a break-point on Main method and when the break-point hits &#8220;!u @eip&#8221; will disassemble the ip, the @eip register will have the address of the current function. Here is the output from !u @eip

> \*** WARNING: Unable to verify checksum for ConsoleApplication11.exe
> 
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 11:
  
> 01f10070 55 push ebp
  
> 01f10071 8bec mov ebp,esp
  
> 01f10073 83ec0c sub esp,0Ch
  
> 01f10076 894dfc mov dword ptr [ebp-4],ecx
  
> 01f10079 833d3c31360000 cmp dword ptr ds:[36313Ch],0
  
> 01f10080 7405 je 01f10087
  
> 01f10082 e8c85a5064 call clr!JIT_DbgIsJustMyCode (66415b4f)
  
> 01f10087 33d2 xor edx,edx
  
> 01f10089 8955f4 mov dword ptr [ebp-0Ch],edx
  
> 01f1008c 33d2 xor edx,edx
  
> 01f1008e 8955f8 mov dword ptr [ebp-8],edx
  
> >>> 01f10091 90 nop
  
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 12:
  
> 01f10092 c745f80a000000 mov dword ptr [ebp-8],0Ah
  
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 13:
  
> 01f10099 8b0530200f03 mov eax,dword ptr ds:\[30F2030h\] (&#8220;naveen&#8221;)
  
> 01f1009f 8945f4 mov dword ptr [ebp-0Ch],eax
  
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 14:
  
> 01f100a2 8b4df4 mov ecx,dword ptr [ebp-0Ch]
  
> \*** WARNING: Unable to verify checksum for C:WindowsassemblyNativeImages\_v4.0.30319\_32mscorlib246f1a5abb686b9dcdf22d3505b08ceamscorlib.ni.dll
  
> 01f100a5 e802706d63 call mscorlib_ni+0x2570ac (655e70ac) (<span style="background-color:#ffff00;">System.Console.WriteLine(System.String) </span>, mdToken: 06000919)
  
> 01f100aa 90 nop
  
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 15:
  
> 01f100ab e8b4c1ca63 call mscorlib_ni+0x82c264 (65bbc264) (System.Console.Read(), mdToken: 0600090a)
  
> 01f100b0 90 nop
  
> c:usersnaveendocumentsvisual studio 2010ProjectsConsoleApplication11Program.cs @ 17:
  
> 01f100b1 90 nop
  
> 01f100b2 8be5 mov esp,ebp
  
> 01f100b4 5d pop ebp
  
> 01f100b5 c3 ret
  
> eax=003637f0 ebx=00000000 ecx=020fbc7c edx=00000000 esi=004bb2d0 edi=0016f400
  
> eip=01f10091 esp=0016f3c8 ebp=0016f3d4 iopl=0 nv up ei pl zr na pe nc
  
> cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00000246
  
> 01f10091 90 nop

Notice in the above code it is Console.WriteLine and not Console.get_Out.

Here is one of gotchas of debugging optimized code.