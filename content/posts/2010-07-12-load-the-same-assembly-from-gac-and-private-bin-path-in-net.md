---
title: Load the same Assembly from GAC and private bin path in .NET
author: admin
type: post
date: 2010-07-12T21:31:40+00:00
url: /?p=966
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1406555122;}'
categories:
  - .NET
  - Loader

---
This post is all about exploring the CLR loader to load an assembly from GAC and from private bin path. This is not possible because, if an assembly is loaded from GAC and if the code uses Assembly.LoadFrom to load the same assembly, then the loader would return the assembly that has already been loaded from the GAC.

Here is the ClassLibrary2 code

[sourcecode language=&#8221;csharp&#8221;]

using System;

namespace ClassLibrary2
  
{
   
public class Class1
   
{
   
public string Bar()
   
{
   
return "bar";
   
}

}
  
}

[/sourcecode]

Here is the Winforms application to load the Class Library Code

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
using System.Reflection;
  
using System.Windows.Forms;
  
using ClassLibrary2;
  
namespace WindowsFormsApplication4
  
{
   
public partial class Form1 : Form
   
{
   
public Form1()
   
{
   
InitializeComponent();
   
var c = new Class1();
   
Console.WriteLine(c.Bar());
   
}

private void button1_Click(object sender, EventArgs e)
   
{
   
var  ad = AppDomain.CreateDomain("Test");
   
ad.AssemblyResolve += ((a1,e1) => Assembly.LoadFile(@"C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9ClassLibrary2binDebugClassLibrary2.dll"));
   
var a = ad.Load("ClassLibrary");
   
var c1 = a.CreateInstance("ClassLibrary2.Class1") as Class1;
   
c1.Bar();
   
}
   
}
  
}
  
[/sourcecode]

If you notice the code creates an instance of the “Class1” in the Form1 constructor, this will essentially load the ClassLibrary2 in to memory. FYI the ClassLibrary2 is GACed when the application launches. Here is the fusglovwr output , which shows the assembly being loaded from GAC.

> LOG: This bind starts in default load context.
> 
> LOG: Using application configuration file: C:UsersnaveenDocumentsVisual Studio 2010ProjectsWindowsFormsApplication4binDebugWindowsFormsApplication4.exe.Config
> 
> LOG: Using host configuration file:
> 
> LOG: Using machine configuration file from C:WindowsMicrosoft.NETFrameworkv4.0.30319configmachine.config.
> 
> LOG: Post-policy reference: ClassLibrary2, Version=1.0.0.0, Culture=neutral, PublicKeyToken=3a0a06f4596f3e16
> 
> LOG: Found assembly by looking in the GAC.
> 
> LOG: Binding succeeds. Returns assembly from C:WindowsMicrosoft.NetassemblyGAC\_MSILClassLibrary2v4.0\_1.0.0.0__3a0a06f4596f3e16ClassLibrary2.dll.
> 
> LOG: Assembly is loaded in default load context.

After verifying fuslogvwr output , I uninstall the assembly from gac using gacutil /u (without closing the app). After which I click the button1 ,which will call the Assembly.LoadFrom to load the same assembly from private bin path. And here fuslogvwr output of loading the assembly using the load from, notice GAC lookup was unsuccessful.

> LOG: This bind starts in default load context.
> 
> LOG: Using application configuration file: C:UsersnaveenDocumentsVisual Studio 2010ProjectsWindowsFormsApplication4binDebugWindowsFormsApplication4.exe.Config
> 
> LOG: Using host configuration file:
> 
> LOG: Using machine configuration file from C:WindowsMicrosoft.NETFrameworkv4.0.30319configmachine.config.
> 
> LOG: Post-policy reference: ClassLibrary2, Version=1.0.0.0, Culture=neutral, PublicKeyToken=3a0a06f4596f3e16
> 
> LOG: GAC Lookup was unsuccessful.
> 
> LOG: Attempting download of new URL file:///C:/Users/naveen/Documents/Visual Studio 2010/Projects/WindowsFormsApplication4/bin/Debug/ClassLibrary2.DLL.
> 
> LOG: Assembly download was successful. Attempting setup of file: C:UsersnaveenDocumentsVisual Studio 2010ProjectsWindowsFormsApplication4binDebugClassLibrary2.dll
> 
> LOG: Entering run-from-source setup phase.
> 
> LOG: Assembly Name is: ClassLibrary2, Version=1.0.0.0, Culture=neutral, PublicKeyToken=3a0a06f4596f3e16
> 
> LOG: Binding succeeds. Returns assembly from C:UsersnaveenDocumentsVisual Studio 2010ProjectsWindowsFormsApplication4binDebugClassLibrary2.dll.
> 
> LOG: Assembly is loaded in default load context.

And here is the !dumpdomain output from windbg for this process

> Assembly:           003ce1e0 [C:WindowsMicrosoft.NetassemblyGAC\_MSILClassLibrary2v4.0\_1.0.0.0__3a0a06f4596f3e16ClassLibrary2.dll]
  
> ClassLoader:        003ee5b8
  
> SecurityDescriptor: 003d2838
  
> Module Name
  
> 00306b28            C:WindowsMicrosoft.NetassemblyGAC\_MSILClassLibrary2v4.0\_1.0.0.0__3a0a06f4596f3e16ClassLibrary2.dll
> 
> Assembly:           003ceae0 [C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9ClassLibrary2binDebugClassLibrary2.dll]
  
> ClassLoader:        0044f418
  
> SecurityDescriptor: 00445ea8
  
> Module Name
  
> 011ee6b4            C:UsersnaveenDocumentsVisual Studio 2010ProjectsConsoleApplication9ClassLibrary2binDebugClassLibrary2.dll

Notice the first assembly is loaded from the GAC and the second one is loaded from the bin path.

This is not something that I would ever do. This was merely an exercise to bypass the loader.

FYI when the assembly is uninstalled from GAC, the gacutil.exe creates a temp copy and stores the assembly in the temp directory because it is being used by a process. And the temp copy of the assembly is deleted when the process ends. Here is the screen shot of Procmon creating an temp copy of the GACed assembly which I uninstalled.

[<img class="alignnone size-full wp-image-967" title="gacutil" src="http://104.197.135.42/wp-content/uploads/2010/07/gacutil1.jpg" alt="" width="700" height="418" />][1]

 [1]: http://104.197.135.42/wp-content/uploads/2010/07/gacutil1.jpg