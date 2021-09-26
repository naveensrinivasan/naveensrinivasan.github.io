---
title: dumpstring – windbg
author: admin
type: post
date: 2010-06-30T01:55:42+00:00
url: /?p=913
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1359378520;}'
categories:
  - Windbg

---
Viewing strings inside the debugger has never been pretty, especially if you are using sos extension.  Here is a sample !dumpobj on a string

> 0:000> !do 00000000025f2280
  
> Name:        System.String
  
> MethodTable: 000007fef6e26960
  
> EEClass:     000007fef69aeec8
  
> Size:        32(0x20) bytes
  
> String:     <span style="background-color:#ffff00;">Foo</span>
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef6e2c848  40000ed        8         System.Int32  1 instance                3 m_stringLength
  
> 000007fef6e2b388  40000ee        c          System.Char  1 instance               46 m_firstChar
  
> 000007fef6e26960  40000ef       10        System.String  0   shared           static Empty
  
> >> Domain:Value  00000000002ae900:00000000025e1420 <<

Some of the devs like to use the du command

[sourcecode]
  
du 00000000025f2280+c
  
[/sourcecode]

> 0:000> du 00000000025f2280+c
> 
> 00000000\`025f228c  <span style="background-color:#ffff00;">&#8220;Foo&#8221;</span>

My choice is to use the .printf command and here is my alias for printing string

[sourcecode]
  
as !ds .printf "%mu n", c+
  
[/sourcecode]

> 0:000> !ds 00000000025f2280
> 
> <span style="background-color:#ffff00;">Foo</span>

I prefer .printf over du because I am not interested in looking at the memory address often especially dumping strings within a script.