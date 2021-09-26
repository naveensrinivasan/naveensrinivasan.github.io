---
title: Using Managed Code to debug Memory Dumps
author: admin
type: post
date: 2010-11-11T22:21:02+00:00
url: /?p=1168
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1359015220;}'
categories:
  - Windbg

---
I happened to notice the new [I happened to notice the new][1] and it had COM based API for dbgeng. The sample code were in VB Script. I much comfortable writing managed code compared to VB script. So I  decided to use COM based API in managed code.

Here are couple of ways to solve certain problems using this

  1. **Parallel GC Roots** :- Getting GC Roots from memory dump is the most time consuming because SOS is single threaded. I use PFX to do them in parallel.
  2. **Reconstructing manged objects** :- Creating an instance of an object by reading data from the memory dump.

Need to add reference to the COM Library

[<img class="alignnone size-full wp-image-1171" title="dbghost" src="http://104.197.135.42/wp-content/uploads/2010/11/dbghost2.png" alt="" width="700" height="403" />][2]

And in VS2010 (.NET 4.0) by default  COM Interop types have  Embed Interop Types turned on. I couldn&#8217;t compile the code with this option. I had to turn off Embed Interop types.

Few extension methods for the DbgObj

[sourcecode language=&#8221;csharp&#8221;]
  
static class DbgExtensions {
   
public static DbgObj OpenDump(this DbgControlClass dbg, string dumpPath) {
   
var path = Environment.GetEnvironmentVariable("\_NT\_SYMBOL_PATH");
   
return dbg.OpenDump(dumpPath, path, path, null);
   
}
   
public static void LoadSOS(this DbgObj dbg){
   
// By default it only loads psscor2.dll and It will not work for .NET 4.0
   
dbg.UnloadExtensions();
   
// Will load sos based on the framework version
   
var sos = dbg.GetModuleByModuleName("clr") == null ? ".loadby sos mscorwks" : ".loadby sos clr";
   
dbg.Execute(sos);
   
}
   
public static IEnumerable<string> DumpHeap(this DbgObj dbg, string typeorMT, bool isMT = false) {
   
var parameter = isMT ? "-MT " : "-type ";
   
return dbg.Execute("!dumpheap -short " + parameter + typeorMT).Split(new[] { "n" },
   
StringSplitOptions.RemoveEmptyEntries);
   
}
   
public static string GCRoot(this DbgObj dbg, string address) {
   
return dbg.Execute("!GcRoot" + address);
   
}
   
public static double ReadDouble(this DbgObj dbg, string address, string offset) {
   
return (double)Int32.Parse(
   
dbg.Execute(string.Format("dd {0}+{1} L1", address, offset)).Replace("n", "")
   
.Split(new[] { " " }, StringSplitOptions.RemoveEmptyEntries)
   
.ElementAt(1),
   
NumberStyles.AllowHexSpecifier);
   
}
   
public static string ReadString(this DbgObj dbg, string address, string offset) {
   
// The managed string in x86 starts at 8th offset
   
return dbg.ReadUnicodeString(ReadDouble(dbg, address, offset) + 8);
   
}
   
}
  
[/sourcecode]

### Parallel GC Roots

Anybody who is debugged memory dumps for leaks understands the pain of running gcroots within a loop. AFAIK sos is single threaded.  I have had customers who had 24 way CPU&#8217;s who wanted to use all the CPU&#8217;s to debug memory leaks, but it wasn&#8217;t possible.

Here is a code that would make parallel gc roots possible

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
using System.Collections.Generic;
  
using System.Linq;
  
using System.Globalization;
  
using DbgHostLib;
  
namespace ConsoleApplication1 {
  
class Program {
  
static void Main(string[] args) {
  
var dump = new DbgControlClass().OpenDump(@"C:TestClass.dmp").LoadSOS();
  
var testInstances = dump.DumpHeap("Test.TestClass");
  
var roots = testInstances.AsParallel().Select(testclass =>
  
new DbgControlClass().OpenDump(@"C:TestClass.dmp").LoadSOS().GCRoot(testclass)).ToList();
  
Console.Read();
  
}
  
}
  
static class DbgExtensions {
  
public static DbgObj OpenDump(this DbgControlClass dbg, string dumpPath) {
  
var path = Environment.GetEnvironmentVariable("\_NT\_SYMBOL_PATH");
  
return dbg.OpenDump(dumpPath, path, path, null);
  
}
  
public static DbgObj LoadSOS(this DbgObj dbg){
  
// By default it loads psscor2
  
dbg.UnloadExtensions();
  
// Will load sos based on the framework version
  
var sos = dbg.GetModuleByModuleName("clr") == null ? ".loadby sos mscorwks" : ".loadby sos clr";
  
dbg.Execute(sos);
  
return dbg;
  
}
  
public static IEnumerable<string> DumpHeap(this DbgObj dbg, string typeorMT, bool isMT = false) {
  
var parameter = isMT ? "-MT " : "-type ";
  
return dbg.Execute("!dumpheap -short " + parameter + typeorMT).Split(new[] { "n" },
  
StringSplitOptions.RemoveEmptyEntries);
  
}
  
public static string GCRoot(this DbgObj dbg, string address) {
  
var s = dbg.IsClrExtensionMissing;
  
return dbg.Execute("!gcroot " + address);
  
}
  
public static double ReadDouble(this DbgObj dbg, string address, string offset) {
  
return (double)Int32.Parse(
  
dbg.Execute(string.Format("dd {0}+{1} L1", address, offset)).Replace("n", "")
  
.Split(new[] { " " }, StringSplitOptions.RemoveEmptyEntries)
  
.ElementAt(1),
  
NumberStyles.AllowHexSpecifier);
  
}
  
public static string ReadString(this DbgObj dbg, string address, string offset) {
  
// The managed string in x86 starts at 8th offset
  
return dbg.ReadUnicodeString(ReadDouble(dbg, address, offset) + 8);
  
}
  
}
  
}
  
[/sourcecode]

The above code loads a memory dump and looks for object type &#8220;Test.TestClass&#8221; and gets its addresses. Then gets GCRoots in parallel using the AsParallel option.

[<img class="alignnone size-medium wp-image-1175" title="ParallelGCRoots" src="http://104.197.135.42/wp-content/uploads/2010/11/parallelgcroots2.png?w=300" alt="" width="300" height="96" />][3]

### Reconstructing manged objects

Using the same API it is pretty easy to create an actual instance of a class from a memory dump.  Here is the code for which I dumped the memory.

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
namespace Test {
  
class Program {
  
static Foo[] foo= new Foo[5];
  
static void Main(string[] args) {
  
for (int i = 0; i < 5; i++)
  
foo[i] = new Foo() { counter = i, Name = "Name " + i.ToString() };
  
Console.WriteLine(foo);
  
Console.Read();
  
}
  
}
  
class Foo {
  
public int counter;
  
public string Name;
  
public override string ToString() {
  
return string.Format("Counter :- {0} , Name :-  {1} ", counter, Name);
  
}
  
}
  
}
  
[/sourcecode]

Here is the memory structure of Foo

> 0:005> !do 00f1c660
  
> Name:        Test.Foo
  
> MethodTable: 009b38bc
  
> EEClass:     009b14a4
  
> Size:        16(0x10) bytes
  
> File:        C:FoobinDebugFoo.exe
  
> Fields:
  
> MT                  Field          Offset                    Type  VT     Attr    Value Name
  
> 79ba2978  4000002        8         System.Int32  1 instance        2 counter
  
> 79b9f9ac  4000003        4        System.String  0 instance 00f1c680 Name

Notice the variable &#8220;Name&#8221; is in the 4th offset and counter is in the 8th offset. I use these offsets to read its contents from the dump.Here is the code that recreates instances of Foo from the memory dump.

[sourcecode language=&#8221;csharp&#8221;]
  
class Program {
  
static void Main(string[] args) {
  
var dump = new DbgControlClass().OpenDump(@"C:tempFoo.dmp").LoadSOS();
  
var foos = dump.DumpHeap(@"Test.Foo");
  
foos.Select(s => new Foo() { counter = (int)dump.ReadDouble(s, "0x8"), Name = dump.ReadString(s, "0x4") }).
  
ToList().ForEach(Console.WriteLine);
  
Console.Read();
  
}
  
}
  
[/sourcecode]

And here is the output from the above code.

> Counter :- 0 , Name :-  Name 0
  
> Counter :- 1 , Name :-  Name 1
  
> Counter :- 2 , Name :-  Name 2
  
> Counter :- 3 , Name :-  Name 3
  
> Counter :- 4 , Name :-  Name 4

There is lot more to explore than what I have shown above. Happy debugging  🙂

 [1]: http://translate.google.com/translate?hl=en&sl=pt&u=http://blogs.technet.com/b/carnevale/archive/2010/10/04/debugdiag-1-2-beta1.aspx&ei=dwbcTOe4DIGglAfc6uWuCQ&sa=X&oi=translate&ct=result&resnum=5&ved=0CDkQ7gEwBA&prev=/search%3Fq%3Ddebugdiag%2B1.2%26hl%3Den%26client%3Dfirefox-a%26hs%3DNUZ%26rls%3Dorg.mozilla:en-US:official%26prmd%3Div
 [2]: http://104.197.135.42/wp-content/uploads/2010/11/dbghost2.png
 [3]: http://104.197.135.42/wp-content/uploads/2010/11/parallelgcroots2.png