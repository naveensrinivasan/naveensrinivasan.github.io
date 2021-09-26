---
title: Saving Dynamic Assembly in .NET 4.0 using Windbg
author: admin
type: post
date: 2010-12-23T23:37:45+00:00
url: /?p=1298
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168818;}'
categories:
  - .NET
  - Windbg

---
I recently had to debug a .NET 4.0 process which was loading the dependent assemblies using the <a href="http://msdn.microsoft.com/en-us/library/system.appdomain.assemblyresolve.aspx" target="_blank">AppDomain.AssemblyResolve</a> event. The dependent assemblies were stored within the executable. I couldn’t disassemble the code to look for the dependent assembly because the exe was obfuscated. FYI the dynamic assembly cannot be saved using !SaveModule and here is the reason for [I recently had to debug a .NET 4.0 process which was loading the dependent assemblies using the <a href="http://msdn.microsoft.com/en-us/library/system.appdomain.assemblyresolve.aspx" target="_blank">AppDomain.AssemblyResolve</a> event. The dependent assemblies were stored within the executable. I couldn’t disassemble the code to look for the dependent assembly because the exe was obfuscated. FYI the dynamic assembly cannot be saved using !SaveModule and here is the reason for][1] read the comments especially from Evian. Unlike psscor2.dll the sos for .NET 4.0 does not have a !dumpdynamicassembly with a save option.

Here is the sample code to demonstrate this.

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
using System.Reflection;
  
using TestLib;
  
namespace Test
  
{
   
class Foo1
   
{
   
int[] s = new int[2];
   
int v = 100;
   
public Foo1()
   
{
   
Console.WriteLine(new Class1().Foo());
   
}
   
static void Main(string[] args1)
   
{
   
AppDomain.CurrentDomain.AssemblyResolve += (sender, args) =>
   
{
   
String resourceName = "ConsoleApplication13." +new AssemblyName(args.Name).Name + ".dll";
   
using (var stream = Assembly.GetExecutingAssembly().GetManifestResourceStream(resourceName))
   
{
   
Byte[] assemblyData = new Byte[stream.Length];
   
stream.Read(assemblyData, 0, assemblyData.Length);
   
return Assembly.Load(assemblyData);
   
}
   
};
   
System.IO.File.Delete(@"C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication13binDebugTestLib.dll");
   
var s = new Foo1();
   
Console.Read();
   
}
   
}
  
}

[/sourcecode]

I knew the Assembly had to be loaded using  System.Reflection.Assembly.Load(Byte[]) ,so ended setting a break-point on the method using command !mbm \*Assembly.Load\* on the launch of the executable.

Here are the output of args and local variables for the above break-point

> 0:000> !mdv
  
> Frame 0x0: (System.Reflection.Assembly.Load(Byte[])):
  
> [A0]:rawAssembly:0x025fc374 (System.Byte[])
  
> [L0]:<?>

Notice the &#8220;rawAssembly&#8221; argument which has the assembly contents.  Here are the raw memory contents of the address using dd 0x025fc374

> 0:000> dd 0x025fc374
  
> 025fc374  <span style="color:#ff0000;">6a764994 00001000</span> 00905a4d 00000003
  
> 025fc384  00000004 0000ffff 000000b8 00000000
  
> 025fc394  00000040 00000000 00000000 00000000
  
> 025fc3a4  00000000 00000000 00000000 00000000
  
> 025fc3b4  00000000 00000080 0eba1f0e cd09b400
  
> 025fc3c4  4c01b821 685421cd 70207369 72676f72
  
> 025fc3d4  63206d61 6f6e6e61 65622074 6e757220
  
> 025fc3e4  206e6920 20534f44 65646f6d 0a0d0d2e

  1. <span style="color:#ff0000;">6a764994 <span style="color:#000000;">:- Is the Array&#8217;s  Method Table</span></span>
  2. <span style="color:#ff0000;"><span style="color:#000000;"><span style="color:#ff0000;">00001000</span> : &#8211; Is the Array size</span></span>
  3. <span style="color:#ff0000;"><span style="color:#000000;">The rest are the array contents.<br /> </span></span>

Unlike the reference type arrays, the value type arrays  don&#8217;t have a DWORD for Method table of its contents. With this information I could dump the contents from memory in to disk using .writemem command.

[sourcecode]

.writemem c:tempassembly.bin @ecx+8 L?(poi(@ecx+@$ptrsize)*@$ptrsize)

[/sourcecode]

In x86 @ecx register contains argument for rawAssembly. The  @ecx+8 is the start  position of the first byte and that is the reason for using this as the start position for .writemem. The poi(@ecx+@$ptrsize) contains the array size which in our case is 0001000 and multiply it by @$ptrsize which is 4 in x86. The expression (poi(@ecx+@$ptrsize)*@$ptrsize) would in our case result to 4000 bytes.

The assembly.bin would contain data in hex format which has to be converted in to binary format. Here is the code to convert from Hex to Binary format.

[sourcecode]
  
Assembly.Load( File.ReadAllBytes(@"c:tempassembly.bin")
   
.Select(x =>
   
Convert.ToByte(
   
int.Parse((x.ToString("X")),NumberStyles.HexNumber)
   
)).ToArray())
  
.FullName.Dump();
  
[/sourcecode]

 [1]: http://stackoverflow.com/questions/1872502/how-to-save-a-dynamically-generated-assembly-that-is-stored-in-memory/1872588#1872588