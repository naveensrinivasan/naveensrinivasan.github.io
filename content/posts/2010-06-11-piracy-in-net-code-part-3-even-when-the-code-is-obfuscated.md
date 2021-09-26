---
title: Piracy in .NET Code – Part 3 – Even when the code is obfuscated
author: admin
type: post
date: 2010-06-12T00:37:46+00:00
url: /?p=769
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659095;}'
categories:
  - .NET
  - Security

---
Continuing with my [series][1] on Piracy, in this post I am going to be  exploring how someone with little advanced knowledge in CLR / Windows can bypass important function calls like  license validation.

Most of the developers assume just because the code is obfuscated nobody can bypass the licensing logic. I am going to be demonstrating how to bypass certain function call,this is very similar to “Set Next Statement &#8221; in VS. I am not going to be discussing on how to fix this problem.

Here is a sample code.

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
namespace Conosole
  
{   class Program
   
{
   
static void Main(string[] args)
   
{
   
Console.WriteLine("Test");
   
Console.Read();
   
}
   
}
   
}
  
[/sourcecode]

The code has only two instructions. First one writes to console and the next to reads from the console. I would want to bypass the call to that WriteLine function.

Loaded the assembly within windbg and then issued the command

[sourcecode]
  
sxe ld: clrjit
  
[/sourcecode]

When the break-point hits ,issued the command to set a break-point on the Main Method

[sourcecode]
  
.loadby sos clr;.load sosex;!mbm *Program.Main;g
  
[/sourcecode]

Then when the break-point hits for the Main method , issued the following command to disassemble the Main method.

[sourcecode]
  
!u ($ip)
  
[/sourcecode]

> 0:000> !u ($ip)
  
> Normal JIT generated code
  
> Conosole.Program.Main(System.String[])
  
> Begin 00230070, size 2d
> 
> C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication6Program.cs @ 6:
  
> 00230070 55              push    ebp
  
> 00230071 8bec            mov     ebp,esp
  
> 00230073 50              push    eax
  
> 00230074 894dfc          mov     dword ptr [ebp-4],ecx
  
> 00230077 833d3c31180000  cmp     dword ptr ds:[18313Ch],0
  
> 0023007e 7405            je      00230085
  
> 00230080 e8ca5a6962      call    clr!JIT_DbgIsJustMyCode (628c5b4f)
  
> >>> 00230085 90              nop
> 
> C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication6Program.cs @ 7:
  
> 00230086 8b0d30204a03    mov     ecx,dword ptr ds:\[34A2030h\] (&#8220;Test&#8221;)
  
> 0023008c e81b707a61      call    mscorlib_ni+0x2570ac (619d70ac) (System.Console.WriteLine(System.String), mdToken: 06000919)
  
> 00230091 90              nop
> 
> C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication6Program.cs @ 8:
  
> 00230092 e8cdc1d761      call    mscorlib_ni+0x82c264 (61fac264) (System.Console.Read(), mdToken: 0600090a)
  
> 00230097 90              nop
> 
> C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication6Program.cs @ 9:
  
> 00230098 90              nop
  
> 00230099 8be5            mov     esp,ebp
  
> 0023009b 5d              pop     ebp
  
> 0023009c c3              ret
  
> 0:000> bp 0023008c
  
> 0:000> g

Because I have private symbols the line information is shown. So the function I want to bypass is “0023008c e81b707a61      call    mscorlib_ni+0x2570ac (619d70ac) (System.Console.WriteLine(System.String), mdToken: 06000919)” and the ip for this is 0023008c , so went ahead and set a break-point on the address

[sourcecode]
  
bp 0023008c
  
[/sourcecode]

When the break-point hits on 0023008c, I move pointer to the next instruction that I am interested in ,which is “00230092 e8cdc1d761      call    mscorlib_ni+0x82c264 (61fac264) (System.Console.Read(), mdToken: 0600090a)” to avoid the function being invoked and here is the command

[sourcecode]
  
r eip=00230092
  
[/sourcecode]

> 0:000> g
  
> Breakpoint 1 hit
  
> eax=001837f0 ebx=00000000 ecx=024abb50 edx=0041efd0 esi=008196c0 edi=0041ef20
  
> eip=0023008c esp=0041eef0 ebp=0041eef4 iopl=0         nv up ei pl zr na pe nc
  
> cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
  
> 0023008c e81b707a61      call    mscorlib_ni+0x2570ac (619d70ac)
  
> 0:000> r eip=00230092

Now we have managed to bypass the call to Console.WriteLine and here is my output .

[<img class="alignnone size-full wp-image-776" title="Console" src="http://104.197.135.42/wp-content/uploads/2010/06/console1.jpg" alt="" width="450" height="227" />][2]

So the key takeaway is to understand the working  of the platform closer to the metal, which can help us write better and secure code.

 [1]: http://naveensrinivasan.com/category/security/
 [2]: http://104.197.135.42/wp-content/uploads/2010/06/console1.jpg