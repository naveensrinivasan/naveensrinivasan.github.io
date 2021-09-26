---
title: Using Windows Error Reporting (WER) API in managed code to generate memory dump
author: admin
type: post
date: 2010-06-10T03:01:09+00:00
url: /?p=750
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1350960887;}'
categories:
  - .NET
  - Windbg

---
The WER is a pretty cool technology from Microsoft for collecting memory dumps on process crash/ hang. This can be extended to generate on demand when the application needs to. The usual reason for getting a memory dump could be based on certain conditions, for example, the customer feels the application is slow and would want to send the information to WinQual (WER server). If the application happens to be installed on hundreds / thousands of boxes then its not going to be possible to get from individual customers, the best bet is WER. To do this here is an [API][1]. But this is unmanaged API and I didn’t see one for managed code. FYI this would work only on Vista + systems, it will not work on XP.

Here is the basic PInvoke for creating dump and submitting a report. I am also using it along with the watsonbuckets that I had [blogged][2] about.

[sourcecode language=&#8221;csharp&#8221;]

internal enum WER_CONSENT
   
{
   
WerConsentAlwaysPrompt = 4,
   
WerConsentApproved = 2,
   
WerConsentDenied = 3,
   
WerConsentMax = 5,
   
WerConsentNotAsked = 1
   
}
   
internal enum WER\_DUMP\_TYPE
   
{
   
WerDumpTypeHeapDump = 3,
   
WerDumpTypeMax = 4,
   
WerDumpTypeMicroDump = 1,
   
WerDumpTypeMiniDump = 2
   
}
   
internal enum WER\_REPORT\_TYPE
   
{
   
WerReportNonCritical,
   
WerReportCritical,
   
WerReportApplicationCrash,
   
WerReportApplicationHange,
   
WerReportKernel,
   
WerReportInvalid
   
}

internal static class Unmanaged
   
{
   
[DllImport("wer.dll", CharSet = CharSet.Unicode, SetLastError = true)]
   
internal static extern int WerReportAddDump(IntPtr hReportHandle,
   
IntPtr hProcess, IntPtr hThread, WER\_DUMP\_TYPE dumpType, IntPtr pExceptionParam, IntPtr pDumpCustomOptions, int dwFlags);
   
[DllImport("wer.dll", CharSet = CharSet.Unicode, SetLastError = true)]
   
internal static extern int WerReportCreate(string pwzEventType,
   
WER\_REPORT\_TYPE repType, IntPtr pReportInformation, ref IntPtr phReportHandle);
   
[DllImport("wer.dll", CharSet = CharSet.Unicode, SetLastError = true)]
   
internal static extern int WerReportSetParameter(IntPtr hReportHandle, int dwparamID, string pwzName, string pwzValue);
   
[DllImport("wer.dll", CharSet = CharSet.Unicode, SetLastError = true)]
   
internal static extern int WerReportSubmit(IntPtr hReportHandle, WER_CONSENT consent, int dwFlags, ref IntPtr pSubmitResult);
   
}

[/sourcecode]

I have shown only few functions in the wer api, there are few more.  Here is the total implementation

[sourcecode language=&#8221;csharp&#8221;]

using System;
  
using System.Windows.Forms;
  
using System.Runtime.InteropServices;
  
namespace WER
  
{
   
public partial class Form1 : Form
   
{
   
static bool Failed(int result)
   
{
   
return (result < 0);
   
}
   
public Form1()
   
{
   
InitializeComponent();
   
button1.Click += (s, b) =>
   
{
   
try
   
{
   
throw new NullReferenceException("Test");
   
}
   
catch (Exception ex)
   
{

var bucket = GetWatsonBuckets();
   
var zero = IntPtr.Zero;

if ((!Failed(Unmanaged.WerReportCreate("CrashingApp",
   
WER\_REPORT\_TYPE.WerReportCritical, IntPtr.Zero, ref zero)) &&
   
(zero != IntPtr.Zero)) && ((((!Failed(Unmanaged.WerReportSetParameter(zero, 0, "AppName", bucket.param0))
   
&& !Failed(Unmanaged.WerReportSetParameter(zero, 1, "AppVer", bucket.param1))) &&
   
(!Failed(Unmanaged.WerReportSetParameter(zero, 2, "AppStamp", bucket.param2)) &&
   
!Failed(Unmanaged.WerReportSetParameter(zero, 3, "AsmAndModName", bucket.param3)))) &&
   
((!Failed(Unmanaged.WerReportSetParameter(zero, 4, "AsmVer", bucket.param4)) &&
   
!Failed(Unmanaged.WerReportSetParameter(zero, 5, "ModStamp", bucket.param5))) &&
   
(!Failed(Unmanaged.WerReportSetParameter(zero, 6, "MethodDef", bucket.param6)) &&
   
!Failed(Unmanaged.WerReportSetParameter(zero, 7, "Offset", bucket.param7))))) &&
   
!Failed(Unmanaged.WerReportSetParameter(zero, 8, "ExceptionType", bucket.param8))))
   
{

var currentProcess = System.Diagnostics.Process.GetCurrentProcess().Handle;
   
if (!Failed(Unmanaged.WerReportAddDump(zero, currentProcess,
   
IntPtr.Zero, WER\_DUMP\_TYPE.WerDumpTypeHeapDump, IntPtr.Zero, IntPtr.Zero, 0)))
   
{
   
var pSubmitResult = IntPtr.Zero;
   
Unmanaged.WerReportSubmit(zero, WER_CONSENT.WerConsentNotAsked, 4, ref pSubmitResult);
   
}
   
}
   
}
   
};
   
}
   
private static WatsonBuckets GetWatsonBuckets()
   
{
   
var pParams = new WatsonBuckets();
   
IClrRuntimeHost host = null;
   
host = Activator.CreateInstance(Type.GetTypeFromCLSID(ClrGuids.ClsIdClrRuntimeHost)) as IClrRuntimeHost;
   
if (host != null)
   
{
   
var clrControl = host.GetCLRControl();
   
if (clrControl == null)
   
{
   
return pParams;
   
}
   
var clrErrorReportingManager =
   
clrControl.GetCLRManager(ref ClrGuids.IClrErrorReportingManager) as IClrErrorReportingManager;
   
if (clrErrorReportingManager == null)
   
{
   
return pParams;
   
}
   
clrErrorReportingManager.GetBucketParametersForCurrentException(out pParams);
   
}
   
return pParams;
   
}
   
}
   
// BucketParameters Structure to get watson buckets back from CLR
   
//http://msdn.microsoft.com/en-us/library/ms404466(v=VS.100).aspx
   
[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
   
internal struct WatsonBuckets
   
{
   
internal int fInited;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string pszEventTypeName;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param0;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param1;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param2;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param3;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param4;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param5;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param6;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param7;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param8;
   
[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 0xff)]
   
internal string param9;
   
}
   
internal static class ClrGuids
   
{
   
internal static readonly Guid ClsIdClrRuntimeHost = new Guid("90F1A06E-7712-4762-86B5-7A5EBA6BDB02");
   
internal static Guid IClrErrorReportingManager = new Guid("980D2F1A-BF79-4c08-812A-BB9778928F78");
   
internal static readonly Guid IClrRuntimeHost = new Guid("90F1A06C-7712-4762-86B5-7A5EBA6BDB02");
   
}
   
[Guid("90F1A06C-7712-4762-86B5-7A5EBA6BDB02"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
   
internal interface IClrRuntimeHost
   
{
   
void Start();
   
void Stop();
   
void SetHostControl(IntPtr pHostControl);
   
IClrControl GetCLRControl();
   
void UnloadAppDomain(int dwAppDomainId, bool fWaitUntilDone);
   
void ExecuteInAppDomain(int dwAppDomainId, IntPtr pCallback, IntPtr cookie);
   
int GetCurrentAppDomainId();

int ExecuteApplication(string pwzAppFullName, int dwManifestPaths, string[] ppwzManifestPaths,
   
int dwActivationData, string[] ppwzActivationData);

int ExecuteInDefaultAppDomain(string pwzAssemblyPath, string pwzTypeName, string pwzMethodName,
   
string pwzArgument);
   
}
   
[Guid("9065597E-D1A1-4fb2-B6BA-7E1FCE230F61"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
   
internal interface IClrControl
   
{
   
[return: MarshalAs(UnmanagedType.IUnknown)]
   
object GetCLRManager([In] ref Guid riid);

void SetAppDomainManagerType(string pwzAppDomainManagerAssembly, string pwzAppDomainManagerType);
   
}
   
// IClrErrorReportingManager to get watson bukets back from CLR
   
//http://msdn.microsoft.com/en-us/library/ms164367(v=VS.100).aspx
   
[Guid("980D2F1A-BF79-4c08-812A-BB9778928F78"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
   
internal interface IClrErrorReportingManager
   
{
   
[PreserveSig]
   
int GetBucketParametersForCurrentException(out WatsonBuckets pParams);
   
}
  
}
  
[/sourcecode]

In the above code ,I am using this within a windows form to create a memory dump when an exception is thrown. The GetWatson bucket and the clr hosting interface is just to get the watson bucket information from clr. This can even be extended to attach a custom log file. How cool is this. Here is the call-stack for the above exception

[<img class="alignnone size-full wp-image-762" title="wer1" src="http://104.197.135.42/wp-content/uploads/2010/06/wer12.jpg" alt="" width="450" height="268" />][3]

 [1]: http://msdn.microsoft.com/en-us/library/bb513635%28VS.85%29.aspx
 [2]: http://naveensrinivasan.com/2010/04/24/exploring-unhandledexception-in-net-and-watson-buckets/
 [3]: http://104.197.135.42/wp-content/uploads/2010/06/wer12.jpg