---
title: Identify and Patch .NET Code using Windbg
author: admin
type: post
date: 2010-02-07T13:30:13+00:00
url: /?p=19
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1404940329;}'
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1404940329;}'
categories:
  - .NET
  - Windbg
tags:
  - .NET
  - SOS
  - Windbg

---
&#160;

The last week was really an interesting one with debugging production code. I was debugging a Winforms application which was using .NET framework 3.5 version. The real problem was with the latest release of the code, there was bug which caused certain elements on the UI not to be displayed. This is was High priority bug and very important to the business. 

The code that was causing this bug was an integer variable inside a class. 

<pre class="code"><span style="color:blue;">using </span>System;
<span style="color:blue;">using </span>System.Windows.Forms;

<span style="color:blue;">namespace </span>WindowsFormsApplication2
{
    <span style="color:blue;">public partial class </span><span style="color:#2b91af;">Form1 </span>: <span style="color:#2b91af;">Form
    </span>{
        <span style="color:blue;">public int </span>Foo;
        <span style="color:blue;">public </span>Form1()
        {
            InitializeComponent();
        }
        <span style="color:blue;">private void </span>Button1Click(<span style="color:blue;">object </span>sender, <span style="color:#2b91af;">EventArgs </span>e)
        {
            <span style="color:blue;">if </span>(Foo == <span style="color:brown;">100</span>) 
                <span style="color:blue;">this</span>.label1.Visible = <span style="color:blue;">true</span>;
        }
    }
}</pre>

[][1][][1]

The code was something like this. If Foo was equal 100 then the label was set to be visible. <a href="http://www.red-gate.com/products/reflector/" target="_blank">Reflector</a> came in handy to disassemble&#160; the code.With the latest code release the&#160; “<span style="color:blue;">if </span>(Foo == <span style="color:brown;">100</span>) “ condition was introduced.

The next step was to verify and validate that this condition was the reason for this bug. Though this looks very simple because i have shown a very contrived example which does not involve all the dependencies of the real world business application. 

Fired up windbg looked up the heap for object type Form1 using dumpheap. 

0:004> !dumpheap -type Form1
    
  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Address&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; MT&#160;&#160;&#160;&#160; Size

**0000000002cb2548** 000007ff004b7b68&#160;&#160;&#160;&#160;&#160; 480&#160;&#160;&#160;&#160;   
total 1 objects

Statistics:

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; MT&#160;&#160;&#160; Count&#160;&#160;&#160; TotalSize Class Name

000007ff004b7b68&#160;&#160;&#160;&#160;&#160;&#160;&#160; 1&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 480 WindowsFormsApplication2.Form1

Total 1 objects

The next was figure of the offset of Foo. So used the !do command on Form1 instance. **!do 0000000002cb2548** and here is the partial output of the command&#160;&#160;&#160; 

000007ff00107050&#160; 4001e9b&#160;&#160;&#160;&#160; 1780&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.Object&#160; 0&#160;&#160; static 0000000002cb2d48 EVENT_MAXIMIZEDBOUNDSCHANGED
    
  
0000000000000000&#160; 4000002&#160;&#160;&#160;&#160;&#160; 1b8&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 0 instance 0000000000000000 components 

000007ff00694f40&#160; 4000003&#160;&#160;&#160;&#160;&#160; 1c0 &#8230;dows.Forms.Button&#160; 0 instance 0000000002cdc040 button1 

000007ff006962c8&#160; 4000004&#160;&#160;&#160;&#160;&#160; 1c8 &#8230;ndows.Forms.Label&#160; 0 instance 0000000002cdc2f8 label1 

000007ff002683d8&#160; 4000005&#160;&#160;&#160;&#160;&#160; **1d0&#160;&#160;&#160;** &#160;&#160;&#160;&#160; System.Int32&#160; 1 instance&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 0 Foo

So from the result i could identify that the Foo variable was on the **1d0** offset of the Form1 object. 

After couple of test case runs i dumped the object and here was the output

000007ff00694f40&#160; 4000003&#160;&#160;&#160;&#160;&#160; 1c0 &#8230;dows.Forms.Button&#160; 0 instance 0000000002cdc040 button1
    
  
000007ff006962c8&#160; 4000004&#160;&#160;&#160;&#160;&#160; 1c8 &#8230;ndows.Forms.Label&#160; 0 instance 0000000002cdc2f8 label1

**000007ff002683d8&#160; 4000005&#160;&#160;&#160;&#160;&#160; 1d0&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.Int32&#160; 1 instance&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 23 Foo**

So the Foo’s value was now 23. I am sure most of us are used to updating the variable’s value in VS.NET using immediate window. I did something similar to that but instead used windbg.

In Windbg numbers are hex values, so to set the value as 100 it would have to be 64. You can fire up calc to figure this out or you use the command in windbg **?64** 

**Evaluate expression: 100 = 00000000\`00000064**

FYI “?” expression evaluator in windbg. Now the final step of updating the Foo in memory. The command to do that is 

**ed 0000000002cb2548+1d0 64** 

“e” command is enter values in memory. “e” has many flavors like eu,ed,ea. And ed command is for updating&#160; Double-word values. So ed is to update double-word value and the memory location is **0000000002cb2548+1d0** which is the Form1 memory location&#160; along with Foo is offset.&#160; 

Voila here is the output of !dumpobj after updating the memory 

**000007ff002683d8&#160; 4000005&#160;&#160;&#160;&#160;&#160; 1d0&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.Int32&#160; 1 instance&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 100 Foo**

Now we could make an emergency patch and be certain the patch would work with the fix already tested in production using the debugger. Having windbg in your toolbox is always saves a lot of time.&#160;&#160;

 [1]: http://11011.net/software/vspaste