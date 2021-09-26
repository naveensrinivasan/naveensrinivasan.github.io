---
title: Debugging base class method with conditional break point in .NET using Windbg
author: admin
type: post
date: 2010-06-15T04:40:17+00:00
url: /?p=803
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659088;}'
categories:
  - Windbg

---
In this post I am going to be demonstrating how to have a conditional break-point on the base class method where it has been used by multiple derived classes. A classic example is Winform UI.

The Control base class has got methods like set\_Enabled , set\_Visible which could be consumed by multiple derived controls. The goal is to debug only the control instance that we are interested in. Here is the sample code.

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
using System.Windows.Forms;
  
namespace WindowsFormsApplication1
  
{
  
public partial class Form1 : Form
  
{
   
public Form1()
   
{
   
InitializeComponent();
   
button3.Click += (s, b) => button1.Enabled = button1.Enabled ? false : true;
   
button4.Click += (s, b) => button2.Enabled = button2.Enabled ? false : true;
   
}
   
}
  
}
  
[/sourcecode]

[<img style="display:inline;border-width:0;" title="image" src="http://104.197.135.42/wp-content/uploads/2010/06/image_thumb2.png" border="0" alt="image" width="244" height="244" />][1]

This app that has four buttons. On the click of button3 it toggles button1’s enabled property and the button 4 does the same for button2. The goal is to break only when button1&#8217;s enabled property is set.

Launched the app and attached it to windbg and loaded sosex, issued the command

[sourcecode]
  
!mbm *Control.set_Enabled
  
[/sourcecode]

to set a break-point on enabled method. FYI set_Enabled is not available in the button class because it is derived from the base class. The goal is to break only when the button1’s enabled property is changed. With the above command it will break every time and here is the output when the break-point hits

> rax=000007fede335c40 rbx=0000000000060b28 rcx=000000000242ed18
  
> rdx=0000000000000000 rsi=0000000000000000 rdi=000000000242ed18
  
> rip=000007fede335c4d rsp=000000000015dde0 rbp=000000000015df80
  
> r8=0000000002464500  r9=0000000000000000 r10=000007fffff10018
  
> r11=000000000015dd20 r12=00000000002f5980 r13=000000000015dea8
  
> r14=000000000015e380 r15=0000000000100000
  
> iopl=0         nv up ei pl nz na pe nc
  
> cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000202
  
> System\_Windows\_Forms_ni+0x2b5c4d:
  
> 000007fe\`de335c4d 488bcf          mov     rcx,rdi

The rcx register contains the reference of the instance of the class ,  example button1 / button2. And here is the command to get the offset of the button text

[sourcecode]
  
.shell -ci "!do @rcx" findstr /E text
  
[/sourcecode]

and the output of the command is

> 0:000> .shell -ci&#8221;!do @ecx&#8221; findstr /E text
  
> 000007fef6db6960  40001c2       40        System.String  0 instance 000000000242ec50 text
  
> 000007fef6db5ab8  400016d      280        System.Object  0   static 0000000002403d80 EventBindingContext
  
> .shell: Process exited

The reason behind getting the button text offset is to use the text as the property for conditional break-point. The text member is available in the 40 offset of the button class. Now that we have the offset of the text property lets try and reset the break-point with a condition. To do this  get the address of the break-point using the bl command and the output is

> 0:000> bl
  
> 0 e 000007fe\`de335c4d     0001 (0001)  0:\**** System\_Windows\_Forms_ni+0x2b5c4d

The address of the set_Enabled function is 000007fe\`de335c4d.   Here is the conditional break-point for button1

[sourcecode]
  
bp 000007fe\`de335c4d "as /mu ${/v:name} (poi(@rcx+40)+c);.block{ .if (0== $scmp( "${name}", "button1") ) { .echo &#8216;in button1&#8217;;gc } .else { gc}}"
  
[/sourcecode]

The “as /mu ${/v:name} (poi(@rcx+40)+c)” will set the value of text property  to the variable “name”. The .block command is used to evaluate the variable  “name” and the rest is a simple .if .else command. Every time the button1 is clicked it would output  “in button1” and continue. Now we have managed to break-in only on the button1 click.

 [1]: http://104.197.135.42/wp-content/uploads/2010/06/image2.png