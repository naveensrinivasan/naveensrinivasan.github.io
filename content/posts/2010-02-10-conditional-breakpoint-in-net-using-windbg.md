---
title: Conditional Breakpoint in .NET using Windbg
author: admin
type: post
date: 2010-02-11T03:50:00+00:00
url: /?p=26
reddit:
  - 'a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1340284050";}'
  - 'a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1340284050";}'
  - 'a:2:{s:5:"count";s:1:"0";s:4:"time";s:10:"1340284050";}'
categories:
  - Windbg
tags:
  - Windbg

---
With my few years of production debugging .NET code ,one thing that has really helped me a lot is Windbg. Lot us of know that using sos, sosex and Windbg we should be able to troubleshoot most of the .NET Code. But certain tips / tricks makes us productive in those crucial moments. I am assuming that you are aware of basic usage of sos and windbg.

We know by using !bpmd command we can stick in a break-point on a method. But the issue is we would want to break in to the debugger only on a certain condition, very similar to VS.NET break-point condition.

Here is the sample code that i am going to be using to set the conditional break-point. This is a simple Winforms app.

<pre style="border:1px solid #cecece;background-color:#fbfbfb;min-height:40px;width:500px;overflow:auto;padding:5px;"><pre style="background-color:#fbfbfb;width:100%;">  1: <span style="color:#0000ff;">using</span> System;
</pre>


<pre style="background-color:#ffffff;width:100%;">  2: <span style="color:#0000ff;">using</span> System.Windows.Forms;
</pre>


<pre style="background-color:#fbfbfb;width:100%;">  3:
</pre>


<pre style="background-color:#ffffff;width:100%;">  4: <span style="color:#0000ff;">namespace</span> WindowsFormsApplication2
</pre>


<pre style="background-color:#fbfbfb;width:100%;">  5: {
</pre>


<pre style="background-color:#ffffff;width:100%;">  6:     <span style="color:#0000ff;">public</span> partial <span style="color:#0000ff;">class</span> Form1 : Form
</pre>


<pre style="background-color:#fbfbfb;width:100%;">  7:     {
</pre>


<pre style="background-color:#ffffff;width:100%;">  8:         <span style="color:#0000ff;">public</span> <span style="color:#0000ff;">int</span> Foo;
</pre>


<pre style="background-color:#fbfbfb;width:100%;">  9:
</pre>


<pre style="background-color:#ffffff;width:100%;"> 10:         <span style="color:#0000ff;">public</span> Form1()
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 11:         {
</pre>


<pre style="background-color:#ffffff;width:100%;"> 12:             InitializeComponent();
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 13:         }
</pre>


<pre style="background-color:#ffffff;width:100%;"> 14:
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 15:         <span style="color:#0000ff;">private</span> <span style="color:#0000ff;">void</span> Button1Click(<span style="color:#0000ff;">object</span> sender, EventArgs e)
</pre>


<pre style="background-color:#ffffff;width:100%;"> 16:         {
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 17:             Test(textBox1.Text);
</pre>


<pre style="background-color:#ffffff;width:100%;"> 18:         }
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 19:
</pre>


<pre style="background-color:#ffffff;width:100%;"> 20:         <span style="color:#0000ff;">void</span> Test(<span style="color:#0000ff;">string</span> s)
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 21:         {
</pre>


<pre style="background-color:#ffffff;width:100%;"> 22:             Console.WriteLine(s);
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 23:         }
</pre>


<pre style="background-color:#ffffff;width:100%;"> 24:
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 25:     }
</pre>


<pre style="background-color:#ffffff;width:100%;"> 26: }
</pre>


<pre style="background-color:#fbfbfb;width:100%;"> 27:</pre>


<p>
  In this sample code I would like to put a conditional break-point on the Test method. After attaching the application to windbg ,look for the object Form1 in the heap.
</p>


<blockquote>
  <p>
    0:000> !dumpheap -type WindowsFormsApplication2.Form1<br />
             Address               MT     Size<br />
    0000000002c92548 000007ff004b7b78      480<br />
    total 1 objects<br />
    Statistics:<br />
                  MT    Count    TotalSize Class Name<br />
    000007ff004b7b78        1          480 WindowsFormsApplication2.Form1<br />
    Total 1 objects
  </p>
</blockquote>


<p>
  The next step was to get the address of the function Test.
</p>


<blockquote>
  <p>
    0:000> .shell  -ci &#8220;!dumpmt -md  000007ff004b7b78&#8221; FIND &#8220;Test&#8221;<br />
    000007ff004a3508 000007ff004b7af0      JIT WindowsFormsApplication2.Form1.Test(System.String)<br />
    .shell: Process exited
  </p>
</blockquote>


<p>
  The reason to get address of the function is to use the native “bp” command to set the break-point. FYI the sos also uses only the built in bp command for setting the break-point. The difference is condition that we can pass to the break-point. In the above I use the .shell command to look for Test function address instead of manually looking for the Test function. The .shell command comes in very handy.
</p>


<p>
  In this exercise I would like to break into the debugger only if the argument “s” matches certain condition. I set the bp on the method test using the command “bp 000007ff004a3508”. And here is the result when the break-point hits.
</p>


<blockquote>
  <p>
    0:000> g<br />
    Breakpoint 0 hit<br />
    rax=0000000002d9fcd8 rbx=0000000000000000 rcx=0000000002c92548<br />
    rdx=0000000002d9fcd8 rsi=0000000000000001 rdi=0000000000000000<br />
    rip=000007ff004a3508 rsp=000000000028d3e8 rbp=000000000028d590<br />
    r8=000000000028cde0  r9=000007feed0b14c0 r10=000007feff469f20<br />
    r11=000007ff00060120 r12=00000000003a9460 r13=0000000000000202<br />
    r14=000000001b3e23b8 r15=0000000000030672<br />
    iopl=0         nv up ei pl nz na po nc<br />
    cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000206<br />
    000007ff`004a3508 e9f35a2c00      jmp     000007ff`00769000
  </p>
</blockquote>


<p>
  FYI I am using a 64-bit machine and that’s the reason my pointers are much bigger than usual x86. We are interested on the argument “s”  that is passed to the method which is @rdx register “rdx=0000000002d7ed00”. To verify that we setting the break-point on the correct argument we can test it by using command
</p>


<blockquote>
  <p>
    0:000> .printf &#8220;%mu&#8221;,@rdx+10<br />
    testa
  </p>
</blockquote>


<p>
  Not many of them are aware of how to get just the string from string object , instead of the all additions from !dumpobj. The above command would get just the string . The command .printf contains “%mu” because it is null terminated unicode string and @rdx is the register which contains the argument “s”. The <a href="mailto:“@rdx+10">“@rdx+10</a>” is the actual location of the string in memory and for the x86 it would be <a href="mailto:“@rdx+c">“@rdx+c</a>” for the actual string. Now that we are sure the @rdx is the register we can build condition for the argument. Here it is
</p>


<blockquote>
  <p>
    <strong>bp 000007ff004a3508 &#8220;.block {as /mu ${/v:cmp} @rdx+10; .if ( $spat( &#8220;${cmp}&#8221;, &#8220;*test*&#8221; )  ) { !clrstack; } .else { gc }}&#8221;</strong>
  </p>
</blockquote>


<p>
  <!--CRLF-->
</p>


<p>
  And here is the detailed explanation of the above condition within quotes. The .block command is used for alias evaluation. Alias is like variables within windbg. The “as /mu ${v:cmp} @rdx+10” command creates an string alias by name  of cmp which contains the value of argument “s”. This condition would be evaluated only when the functions first line of code is executed so @rdx will always have the value that is passed to the function. The next
</p>


<blockquote>
  <p>
    “.if ( $spat( &#8220;${cmp}&#8221;, &#8220;*test*&#8221; )  ) { !clrstack; } .else { gc }}”
  </p>
</blockquote>


<p>
  command is real crux where the code compares the alias cmp with “*test*”. Notice i am using a built-in function called $spat which is nothing but a string pattern function. So from the above condition i am instructing the break-point to give a callstack if the argument “s” has something like “*test*” . If not the command “gc” means go to the next conditional break-point,similar  F5 in VS.NET. So, for example if the function is called 40 times and of which we are interested only once when it is  something like “*test*” ,with the existing  !bpmd we would wasted our time 39 times.
</p>


<p>
  The “” is a escape character in windbg ,i am using it because i would have to a string compare.
</p>