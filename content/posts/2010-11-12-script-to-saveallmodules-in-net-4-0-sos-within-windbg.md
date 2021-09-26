---
title: Script to !SaveAllModules in .NET 4.0 SOS within Windbg
author: admin
type: post
date: 2010-11-12T19:29:06+00:00
url: /?p=1202
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422204733;}'
categories:
  - SOS
  - Windbg

---
The .NET 4.0 sos doesn’t have save all modules (!SaveAllModules) command. It only has !SaveModule. Recently I was debugging a .NET 4.0 process for which I had to save all the modules. Here is a script that does !SaveAllModules.

[sourcecode]

!for\_each\_module .if ($spat ("${@#ImageName}","*.exe")) { !SaveModule ${@#Base} c:temp${@#ModuleName}.exe } .else { !SaveModule ${@#Base} c:temp${@#ModuleName}.dll }

[/sourcecode]