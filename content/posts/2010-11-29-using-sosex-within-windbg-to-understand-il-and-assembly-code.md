---
title: Using sosex within windbg to understand IL and Assembly code
author: admin
type: post
date: 2010-11-30T01:33:25+00:00
url: /?p=1261
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168819;}'
categories:
  - SOS
  - SOSEX
  - Windbg

---
Sometimes when debugging managed code within the debugger I would like to see the C# code ,the IL translation for the managed code and the Assembly code for the IL. For example I recently learned that callvirt MSIL instruction must do the null-check before invoking method.

> C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication13Program.cs @ 18:
  
> 00bc26d8 8b4dec          mov     ecx,dword ptr [ebp-14h]
  
> 00bc26db 3909            cmp     dword ptr [ecx],ecx //NULL Check
  
> 00bc26dd ff1508a82900    call    dword ptr ds:\[29A808h\] (System.String.ToLower(), mdToken: 0600031d)
  
> 00bc26e3 8945e8          mov     dword ptr [ebp-18h],eax
  
> 00bc26e6 8b45e8          mov     eax,dword ptr [ebp-18h]
  
> 00bc26e9 8945ec          mov     dword ptr [ebp-14h],eax

I am not an assembly code expert. The above output is from &#8220;!u&#8221; sos command. It doesn&#8217;t show the c# code except the line number and it is missing IL translation.

The &#8220;!mu&#8221; from sosex does what I want. It is not yet documented because it is not yet stable as per the output of the command. Here is the output for the same call-stack using sosex&#8217;s !mu.

> 0:000> !mu
  
> THIS COMMAND IS UNDOCUMENTED AND NOT YET STABLE.
  
> test = test.ToLower();
  
> IL_001a: ldloc.0  (test)
  
> IL_001b: callvirt System.String::ToLower
  
> 00bc26d8 8b4dec          mov     ecx,dword ptr [ebp-14h]
  
> 00bc26db 3909            cmp     dword ptr [ecx],ecx
  
> 00bc26dd ff1508a82900    call    dword ptr ds:[29A808h]
  
> 00bc26e3 8945e8          mov     dword ptr [ebp-18h],eax
  
> IL_0020: stloc.0  (test)
  
> 00bc26e6 8b45e8          mov     eax,dword ptr [ebp-18h]
  
> 00bc26e9 8945ec          mov     dword ptr [ebp-14h],eax

The above output has c#,IL and assembly.