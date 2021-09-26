---
title: Who is is blocking my UI Thread? Diagnosing the cause using Windbg
author: admin
type: post
date: 2011-04-01T18:59:06+00:00
url: /?p=1344
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168816;}'
categories:
  - Windbg

---
It so happens most of the applications block the UI thread and do sync I/O, which is most common reason for “Not Responding” window. Here is a post <http://blogs.msdn.com/b/nathannesbit/archive/2010/12/28/detecting-ui-thread-misuse.aspx> that tries helping in detecting this. I like to handle this from bottom of the stack because we have a cool tool called debugger.

The approach is simple as having a break-point on a function like “KERNEL32!WaitFor*” and checking if the current thread is a UI thread. This could also be done for other functions like Sleep on UI thread by having a break-point on “KERNELBASE!SleepEx”.

Here are steps to determine if a thread is a STA thread / UI Thread. This is information is stored in TEB structure (thread environment block). Here is the output of the teb on the UI Thread

> 0:000> dt ntdll!_TEB @$teb
  
> +0x000 NtTib            : \_NT\_TIB
  
> +0x01c EnvironmentPointer : (null)
  
> +0x020 ClientId         : \_CLIENT\_ID
  
> +0x028 ActiveRpcHandle  : (null)
  
> +0x02c ThreadLocalStoragePointer : 0x7efdd02c
  
> +0x030 ProcessEnvironmentBlock : 0x7efde000 _PEB
  
> +0x034 LastErrorValue   : 0
  
> +0x038 CountOfOwnedCriticalSections : 0
  
> +0x03c CsrClientThread  : (null)
  
> +0x040 Win32ThreadInfo  : (null)
  
> +0x044 User32Reserved   : [26] 0
  
> +0x0ac UserReserved     : [5] 0
  
> +0x0c0 WOW32Reserved    : 0x751b2320
  
> +0x0c4 CurrentLocale    : 0x409
  
> +0x0c8 FpSoftwareStatusRegister : 0
  
> +0x0cc SystemReserved1  : \[54\] (null)
  
> +0x1a4 ExceptionCode    : 0
  
> +0x1a8 ActivationContextStackPointer : 0x001a07d0 \_ACTIVATION\_CONTEXT_STACK
  
> +0x1ac SpareBytes       : [36]  &#8220;&#8221;
  
> +0x1d0 TxFsContext      : 0xfffe
  
> +0x1d4 GdiTebBatch      : \_GDI\_TEB_BATCH
  
> +0x6b4 RealClientId     : \_CLIENT\_ID
  
> +0x6bc GdiCachedProcessHandle : (null)
  
> +0x6c0 GdiClientPID     : 0
  
> +0x6c4 GdiClientTID     : 0
  
> +0x6c8 GdiThreadLocalInfo : (null)
  
> +0x6cc Win32ClientInfo  : [62] 0
  
> +0x7c4 glDispatchTable  : \[233\] (null)
  
> +0xb68 glReserved1      : [29] 0
  
> +0xbdc glReserved2      : (null)
  
> +0xbe0 glSectionInfo    : (null)
  
> +0xbe4 glSection        : (null)
  
> +0xbe8 glTable          : (null)
  
> +0xbec glCurrentRC      : (null)
  
> +0xbf0 glContext        : (null)
  
> +0xbf4 LastStatusValue  : 0xc0000139
  
> +0xbf8 StaticUnicodeString : \_UNICODE\_STRING &#8220;&#8221;
  
> +0xc00 StaticUnicodeBuffer : [261]  &#8220;&#8221;
  
> +0xe0c DeallocationStack : 0x00320000
  
> +0xe10 TlsSlots         : \[64\] (null)
  
> +0xf10 TlsLinks         : \_LIST\_ENTRY [ 0x0 &#8211; 0x0 ]
  
> +0xf18 Vdm              : (null)
  
> +0xf1c ReservedForNtRpc : 0x001d8c70
  
> +0xf20 DbgSsReserved    : \[2\] (null)
  
> +0xf28 HardErrorMode    : 0
  
> +0xf2c Instrumentation  : \[9\] (null)
  
> +0xf50 ActivityId       : _GUID {00000000-0000-0000-0000-000000000000}
  
> +0xf60 SubProcessTag    : (null)
  
> +0xf64 EtwLocalData     : (null)
  
> +0xf68 EtwTraceData     : (null)
  
> +0xf6c WinSockData      : (null)
  
> +0xf70 GdiBatchCount    : 0x7efdb000
  
> +0xf74 CurrentIdealProcessor : \_PROCESSOR\_NUMBER
  
> +0xf74 IdealProcessorValue : 0x1010000
  
> +0xf74 ReservedPad0     : 0 &#8221;
  
> +0xf75 ReservedPad1     : 0 &#8221;
  
> +0xf76 ReservedPad2     : 0x1 &#8221;
  
> +0xf77 IdealProcessor   : 0x1 &#8221;
  
> +0xf78 GuaranteedStackBytes : 0x1000
  
> +0xf7c ReservedForPerf  : (null)
  
> +0xf80 ReservedForOle   : 0x001ffd50

I have shown only the partial output because we are interested only in &#8220;**ReservedForOle**&#8221; member which is in the **oxf80** offset. Within this structure in &#8220;**0xc**&#8221; offset contains the information on whether it is STA / MTA / Unkown and here is a write up on this from John Robbins <http://www.microsoft.com/msj/1099/bugslayer/bugslayer1099.aspx>.  Though the posts mentions STA as **0x80** and MTA as **0x140** with current version of windows value of STA is **81** <span style="font-size:x-small;"><span style="font-size:x-small;"><strong><span style="font-size:x-small;"> </span></strong></span></span>and MTA is **141**.
  
With this information it was pretty easy to create a script which will give us a call-stack if a UI Thread is blocking.

[sourcecode]
  
bm KERNEL32!WaitFor* ".if (poi(@$teb+0xf80) != 0) { .if (poi(poi(@$teb+0xf80)+0xc) = 81) {!clrstack;g} .else {g}} .else {gh}"
  
bp KERNELBASE!SleepEx ".if (poi(@$teb+0xf80) != 0) { .if (poi(poi(@$teb+0xf80)+0xc) = 81) {!clrstack;g} .else {g}} .else {gh}"
  
[/sourcecode]

Here is a example call-stack from the above break-point which indicates that we are blocking on the UI Thread

> OS Thread Id: 0x76c (0)
  
> Child SP IP       Call Site
  
> 0029e74c 775c118e [InlinedCallFrame: 0029e74c]
  
> 0029e748 5affbc00 DomainBoundILStubClass.IL\_STUB\_PInvoke(System.Net.Sockets.AddressFamily, System.Net.Sockets.SocketType, System.Net.Sockets.ProtocolType, IntPtr, UInt32, System.Net.SocketConstructorFlags)
  
> 0029e74c 5afa72e4 [InlinedCallFrame: 0029e74c] System.Net.UnsafeNclNativeMethods+OSSOCK.WSASocket(System.Net.Sockets.AddressFamily, System.Net.Sockets.SocketType, System.Net.Sockets.ProtocolType, IntPtr, UInt32, System.Net.SocketConstructorFlags)
  
> 0029e7a4 5afa72e4 System.Net.Sockets.Socket.InitializeSockets()
  
> 0029e7f4 5afcc3ca System.Net.NetworkAddressChangePolled..ctor()
  
> 0029e808 5afcc326 System.Net.AutoWebProxyScriptEngine+AutoDetector.Initialize()
  
> 0029e838 5af7534d System.Net.AutoWebProxyScriptEngine+AutoDetector.get_CurrentAutoDetector()
  
> 0029e83c 5af75263 System.Net.AutoWebProxyScriptEngine..ctor(System.Net.WebProxy, Boolean)
  
> 0029e858 5af75202 System.Net.WebProxy.UnsafeUpdateFromRegistry()
  
> 0029e868 5af751c8 System.Net.WebProxy..ctor(Boolean)
  
> 0029e86c 5af74c79 System.Net.Configuration.DefaultProxySectionInternal..ctor(System.Net.Configuration.DefaultProxySection)
  
> 0029e8b0 5af748d2 System.Net.Configuration.DefaultProxySectionInternal.GetSection()
  
> 0029e8e4 5afcbf76 System.Net.WebRequest.get_InternalDefaultWebProxy()
  
> 0029e914 5afcbc86 System.Net.HttpWebRequest..ctor(System.Uri, System.Net.ServicePoint)
  
> 0029e92c 5afcbbcb System.Net.HttpRequestCreator.Create(System.Uri)
  
> 0029e938 5afcb772 System.Net.WebRequest.Create(System.Uri, Boolean)
  
> 0029e95c 5af94cad System.Net.WebRequest.Create(System.String)
  
> 0029e96c 0087056e WindowsFormsApplication1.Form1.<.ctor>b__0(System.Object, System.EventArgs) [C:UsersnaveenDocumentsVisual Studio 2010ProjectsWindowsFormsApplication1WindowsFormsApplication1Form1.cs @ 15]
  
> 0029e9ac 592b4ae8 System.Windows.Forms.Control.OnClick(System.EventArgs)
  
> 0029e9c4 592b70a2 System.Windows.Forms.Button.OnClick(System.EventArgs)
  
> 0029e9dc 59846174 System.Windows.Forms.Button.OnMouseUp(System.Windows.Forms.MouseEventArgs)
  
> 0029e9f8 598195b5 System.Windows.Forms.Control.WmMouseUp(System.Windows.Forms.Message ByRef, System.Windows.Forms.MouseButtons, Int32)
  
> 0029ea8c 59bda1bf System.Windows.Forms.Control.WndProc(System.Windows.Forms.Message ByRef)
  
> 0029ea90 59be18dd [InlinedCallFrame: 0029ea90]
  
> 0029eae4 59be18dd System.Windows.Forms.ButtonBase.WndProc(System.Windows.Forms.Message ByRef)
  
> 0029eb28 5931de00 System.Windows.Forms.Button.WndProc(System.Windows.Forms.Message ByRef)
  
> 0029eb34 593070f3 System.Windows.Forms.Control+ControlNativeWindow.OnMessage(System.Windows.Forms.Message ByRef)
  
> 0029eb3c 59307071 System.Windows.Forms.Control+ControlNativeWindow.WndProc(System.Windows.Forms.Message ByRef)
  
> 0029eb50 59306fb6 System.Windows.Forms.NativeWindow.Callback(IntPtr, Int32, IntPtr, IntPtr)
  
> 0029ecf4 01010a35 [InlinedCallFrame: 0029ecf4]