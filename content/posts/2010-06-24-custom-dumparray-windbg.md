---
title: Custom DumpArray – Windbg
author: admin
type: post
date: 2010-06-25T00:59:42+00:00
url: /?p=885
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1350960686;}'
categories:
  - Windbg

---
The [The][1] has !dumparray for getting contents of the array. But it cannot be used for scripting or automation. Here is an example

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
namespace ConsoleApplication
  
{
   
class Program
  
{
   
Test[] arr = new[] { new Test() { ID = 1, Name = "Foo" }, new Test() { ID = 2, Name = "Bar" } };
   
static void Main(string[] args)
   
{
   
var p = new Program();
   
Console.WriteLine(p.arr);
   
Console.Read();
   
}
   
}
   
class Test
   
{
   
public int ID;
   
public string Name;
   
}
  
}
  
[/sourcecode]

And here is the output of the **<span style="color:#993366;">arr</span>** variable within the debugger

> 0:000> !da -details 021fbc6c
  
> Name:        ConsoleApplication.Test[]
  
> MethodTable: 64b56c28
  
> EEClass:     648d9698
  
> Size:        24(0x18) bytes
  
> Array:       Rank 1, Number of elements 2, Type CLASS
  
> Element Methodtable: 001938b0
  
> [0] 021fbc84
  
> Name:        ConsoleApplication.**<span style="color:#993366;">Test</span>**
  
> MethodTable: 001938b0
  
> EEClass:     00191488
  
> Size:        16(0x10) bytes
  
> File:        C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  4000002        8             System.Int32      1     instance            1     ID
  
> 64b9f9ac  4000003        4            System.String      0     instance     021fbc44     **<span style="color:#993366;">Name</span>**
  
> [1] 021fbc94
  
> Name:        ConsoleApplication.**<span style="color:#993366;">Test</span>**
  
> MethodTable: 001938b0
  
> EEClass:     00191488
  
> Size:        16(0x10) bytes
  
> File:        C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  4000002        8             System.Int32      1     instance            2     ID
  
> 64b9f9ac  4000003        4            System.String      0     instance     021fbc58     **<span style="color:#993366;">Name</span>**

From the output  we cannot see the values of the  **<span style="color:#993366;">Name</span>** variable within the **<span style="color:#993366;">Test</span>** class. To get the value we would have manually issue a !dumpobj on each one of these. In this post I will demonstrate how to automate this. In doing so we will also explore the array internals from raw memory perspective.

To start of lets dump the raw memory of the array. The  array address is <span style="color:#0000ff;"><strong>021fbc6c</strong></span>

[sourcecode]
  
dd 021fbc6c
  
[/sourcecode]

> 0:000> dd **<span style="color:#0000ff;">021fbc6c </span>**
  
> 021fbc6c  <span style="color:#0000ff;"><strong>64b56c28 </strong></span><span style="color:#0000ff;"><strong>00000002 </strong></span>**<span style="color:#0000ff;">001938b0 </span><span style="color:#0000ff;"><em>021fbc84</em></span>**
  
> 021fbc7c  <span style="color:#0000ff;"><strong><em>021fbc94 </em></strong></span>00000000 001938b0 021fbc44
  
> 021fbc8c  00000001 00000000 001938b0 021fbc58
  
> 021fbc9c  00000002 00000000 64ba7490 00000000
  
> 021fbcac  00000000 00000000 00000000 00000000
  
> 021fbcbc  00000000 64b9f5e8 00000000 40010000
  
> 021fbccc  64ba6034 00000007 00000004 00000100
  
> 021fbcdc  00000000 64ba6f40 00000000 00000000

Fields

  1. <span style="color:#00ff00;"><span style="color:#0000ff;"><strong>64b56c28</strong> </span><span style="color:#000000;">&#8211; Array&#8217;s Method table pointer </span></span>
  2. <span style="color:#ff9900;"><span style="color:#0000ff;"><strong>00000002 </strong></span><span style="color:#000000;">&#8211; Array&#8217;s length ( this will be used later)</span></span>
  3. <span style="color:#ff0000;"><span style="color:#0000ff;"><strong>001938b0 </strong></span><span style="color:#000000;">&#8211;</span><span style="color:#000000;">Array contents method table pointer ( Test class)</span></span>
  4. <span style="color:#ff0000;"><span style="color:#000080;"> </span><span style="color:#0000ff;"><strong>021fbc84</strong></span><span style="color:#000080;">,</span> <span style="color:#000080;"><span style="color:#0000ff;"><strong>021fbc94 </strong></span><span style="color:#000000;">&#8211; Contents of the array( 2 instances of the Test class)</span></span></span>

I would be using  $t0, $t1   [User-Defined Pseudo-Registers][2] within my script  as local variables to maintain state. Think of them as predefined variables that we can use. Here is the script to get just the **<span style="color:#993366;">Name </span>**from the array

[sourcecode]
  
.for (r $t0=0; @$t0 < poi(021fbc6c+0x4); r$t0=@$t0+1 ) { r$t1 = 0; .if(@$t0 = 0) { r$t1=10} .else { r$t1= 10+ @$t0\*4};.echo \*\***\***\*****;!do poi(poi((021fbc6c-0x4)+@$t1)+0x4) }
  
[/sourcecode]

Here is the explanation for the above script

  1. The [The [The][1] has !dumparray for getting contents of the array. But it cannot be used for scripting or automation. Here is an example

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
namespace ConsoleApplication
  
{
   
class Program
  
{
   
Test[] arr = new[] { new Test() { ID = 1, Name = "Foo" }, new Test() { ID = 2, Name = "Bar" } };
   
static void Main(string[] args)
   
{
   
var p = new Program();
   
Console.WriteLine(p.arr);
   
Console.Read();
   
}
   
}
   
class Test
   
{
   
public int ID;
   
public string Name;
   
}
  
}
  
[/sourcecode]

And here is the output of the **<span style="color:#993366;">arr</span>** variable within the debugger

> 0:000> !da -details 021fbc6c
  
> Name:        ConsoleApplication.Test[]
  
> MethodTable: 64b56c28
  
> EEClass:     648d9698
  
> Size:        24(0x18) bytes
  
> Array:       Rank 1, Number of elements 2, Type CLASS
  
> Element Methodtable: 001938b0
  
> [0] 021fbc84
  
> Name:        ConsoleApplication.**<span style="color:#993366;">Test</span>**
  
> MethodTable: 001938b0
  
> EEClass:     00191488
  
> Size:        16(0x10) bytes
  
> File:        C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  4000002        8             System.Int32      1     instance            1     ID
  
> 64b9f9ac  4000003        4            System.String      0     instance     021fbc44     **<span style="color:#993366;">Name</span>**
  
> [1] 021fbc94
  
> Name:        ConsoleApplication.**<span style="color:#993366;">Test</span>**
  
> MethodTable: 001938b0
  
> EEClass:     00191488
  
> Size:        16(0x10) bytes
  
> File:        C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9binDebugConsoleApplication.exe
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  4000002        8             System.Int32      1     instance            2     ID
  
> 64b9f9ac  4000003        4            System.String      0     instance     021fbc58     **<span style="color:#993366;">Name</span>**

From the output  we cannot see the values of the  **<span style="color:#993366;">Name</span>** variable within the **<span style="color:#993366;">Test</span>** class. To get the value we would have manually issue a !dumpobj on each one of these. In this post I will demonstrate how to automate this. In doing so we will also explore the array internals from raw memory perspective.

To start of lets dump the raw memory of the array. The  array address is <span style="color:#0000ff;"><strong>021fbc6c</strong></span>

[sourcecode]
  
dd 021fbc6c
  
[/sourcecode]

> 0:000> dd **<span style="color:#0000ff;">021fbc6c </span>**
  
> 021fbc6c  <span style="color:#0000ff;"><strong>64b56c28 </strong></span><span style="color:#0000ff;"><strong>00000002 </strong></span>**<span style="color:#0000ff;">001938b0 </span><span style="color:#0000ff;"><em>021fbc84</em></span>**
  
> 021fbc7c  <span style="color:#0000ff;"><strong><em>021fbc94 </em></strong></span>00000000 001938b0 021fbc44
  
> 021fbc8c  00000001 00000000 001938b0 021fbc58
  
> 021fbc9c  00000002 00000000 64ba7490 00000000
  
> 021fbcac  00000000 00000000 00000000 00000000
  
> 021fbcbc  00000000 64b9f5e8 00000000 40010000
  
> 021fbccc  64ba6034 00000007 00000004 00000100
  
> 021fbcdc  00000000 64ba6f40 00000000 00000000

Fields

  1. <span style="color:#00ff00;"><span style="color:#0000ff;"><strong>64b56c28</strong> </span><span style="color:#000000;">&#8211; Array&#8217;s Method table pointer </span></span>
  2. <span style="color:#ff9900;"><span style="color:#0000ff;"><strong>00000002 </strong></span><span style="color:#000000;">&#8211; Array&#8217;s length ( this will be used later)</span></span>
  3. <span style="color:#ff0000;"><span style="color:#0000ff;"><strong>001938b0 </strong></span><span style="color:#000000;">&#8211;</span><span style="color:#000000;">Array contents method table pointer ( Test class)</span></span>
  4. <span style="color:#ff0000;"><span style="color:#000080;"> </span><span style="color:#0000ff;"><strong>021fbc84</strong></span><span style="color:#000080;">,</span> <span style="color:#000080;"><span style="color:#0000ff;"><strong>021fbc94 </strong></span><span style="color:#000000;">&#8211; Contents of the array( 2 instances of the Test class)</span></span></span>

I would be using  $t0, $t1   [User-Defined Pseudo-Registers][2] within my script  as local variables to maintain state. Think of them as predefined variables that we can use. Here is the script to get just the **<span style="color:#993366;">Name </span>**from the array

[sourcecode]
  
.for (r $t0=0; @$t0 < poi(021fbc6c+0x4); r$t0=@$t0+1 ) { r$t1 = 0; .if(@$t0 = 0) { r$t1=10} .else { r$t1= 10+ @$t0\*4};.echo \*\***\***\*****;!do poi(poi((021fbc6c-0x4)+@$t1)+0x4) }
  
[/sourcecode]

Here is the explanation for the above script

  1. The][3] loop is used to iterate through the contents of the array :  &#8220;.for (r $t0=0; @$t0 < poi(021fbc6c+0x4); r$t0=@$t0+1 ) &#8221; 
      * The loop variable is $t0, which is initialized to zero  &#8220;r $t0=0;&#8221;,
      * Next is the loop condition check @$to < poi(021fbc6c+0x4) , the poi(021fbc6c+0x4) is the pointer deference to array length which is <span style="color:#0000ff;"><strong>00000002<br /> </strong></span>
      * <span style="color:#0000ff;"><span style="color:#000000;">And the last statement is the increment command of the loop variable </span></span>r$t0=@$t0+1
      * Tip :- I am using &#8220;@&#8221; before the &#8220;$&#8221; for increased speed within the debugger when accessing registers.
  2. The next statement is &#8220;r$t1 = 0&#8221; is initializing another pseudo register to zero
  3. After which the command &#8220;if(@$t0 = 0) { r$t1=10} .else { r$t1= 10+ @$t0\*4}&#8221; resets the value of $t1 register either &#8220;10&#8221; or $t0 \* 4, where $to is loop variable. I do this because the first instance of the **<span style="color:#993366;">Test</span>** class within the array is in the 10th offset and the rest of them would be on the next 4th offset. So for example the first time loop ,$t1 would be 10 , the second time  $t1 would 14 (10 + 1*4).
  4. The &#8220;.echo \***\***\***\***&#8221; is just for line separation
  5. The last command is the one which does most of the work 
      * The command poi((021fbc6c-0x4)+@$t1) would return the pointer of the each element in the array which is instance of Test class . The first time it would be poi((021fbc6c-0x4)+10) which would point <span style="color:#0000ff;"><strong>021fbc84<em> </em></strong><span style="color:#000000;">and the next time it would be </span></span>poi((021fbc6c-0x4)+14) which would be **<span style="color:#0000ff;">021fbc94</span>**
      * <span style="color:#0000ff;"><span style="color:#000000;">The outermost &#8220;poi 0x4</span><span style="color:#000000;">&#8221; is to get pointer of the member variable <strong><span style="color:#993366;">Name </span></strong></span><span style="color:#000000;">and dump its content using !do</span></span>

And here is the output from the script

> 0:000> .for (r $t0=0; @$t0 < poi(021fbc6c   +0x4); r$t0=@$t0+1 ) { r$t1 = 0; .if(@$t0 = 0) { r$t1=10} .else { r$t1= 10+ @$t0\*4};.echo \*\***\***\*****;  !do poi(poi((021fbc6c-0x4)+@$t1)+0x4) }
  
> \***\***\***\***
  
> Name:        System.String
  
> MethodTable: 64b9f9ac
  
> EEClass:     648d8bb0
  
> Size:        20(0x14) bytes
  
> File:        C:WindowsMicrosoft.NetassemblyGAC\_32mscorlibv4.0\_4.0.0.0__b77a5c561934e089mscorlib.dll
  
> String:      **<span style="color:#993300;">Foo</span>**
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  40000ed        4         System.Int32  1 instance        3 m_stringLength
  
> 64ba1dc8  40000ee        8          System.Char  1 instance       46 m_firstChar
  
> 64b9f9ac  40000ef        8        System.String  0   shared   static Empty
  
> >> Domain:Value  00745c28:021f1228 <<
  
> \***\***\***\***
  
> Name:        System.String
  
> MethodTable: 64b9f9ac
  
> EEClass:     648d8bb0
  
> Size:        20(0x14) bytes
  
> File:        C:WindowsMicrosoft.NetassemblyGAC\_32mscorlibv4.0\_4.0.0.0__b77a5c561934e089mscorlib.dll
  
> String:      **<span style="color:#993300;">Bar</span>**
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr    Value Name
  
> 64ba2978  40000ed        4         System.Int32  1 instance        3 m_stringLength
  
> 64ba1dc8  40000ee        8          System.Char  1 instance       42 m_firstChar
  
> 64b9f9ac  40000ef        8        System.String  0   shared   static Empty
  
> >> Domain:Value  00745c28:021f1228 <<

 [1]: http://msdn.microsoft.com/en-us/library/bb190764.aspx
 [2]: http://msdn.microsoft.com/en-us/library/ff553485(VS.85).aspx
 [3]: http://msdn.microsoft.com/en-us/library/ff563115(VS.85).aspx