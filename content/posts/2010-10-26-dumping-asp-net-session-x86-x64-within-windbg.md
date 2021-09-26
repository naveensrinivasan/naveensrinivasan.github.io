---
title: Dumping ASP.NET Session (x86 /x64) within Windbg
author: admin
type: post
date: 2010-10-27T01:18:21+00:00
url: /?p=1071
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1422208545;}'
categories:
  - .NET
  - ASP.NET
  - SOS
  - Windbg

---
This post is going to be about dumping ASP.NET session objects using Windbg. I had recently answered a stackoverflow question in which someone wanted to dump ASP.NET session objects for 64-bit IIS (x64). I thought why not blog about the same which might be useful to others.  The challenge is to write one script that should work in both x86/x64.  FYI there is a script from [Tess][1] that does dump out the session contents, AFAIK it will not work on x64 and my script iterates through the array using the array length instead of using “.foreach /pS 2 /ps 99” which is somewhat cleaner.

Here is the script for dumping ASP.NET session objects within Windbg / CDB

[sourcecode wraplines=&#8221;true&#8221;]

$$$ Dump the ASP.NET Session objects within windbg/cdb
  
$$$ Platform : x86 / x64
  
$$$ Naveen Srinivasan http://naveensrinivasan.com
  
$$$ Usage: $$>a<"c:Debuggersx86dumpsession.txt" 000007fef4115c20
  
$$$ where 000007fef4115c20 is the MethodTable pointer System.Web.SessionState.HttpSessionState

r @$t9 = @$ptrsize
  
$$ $t9 register contains pointer size
  
$$ $t8 register contains the next offset of the variable
  
$$ $t7 register contains array start address

.if (@$ptrsize = 8 )
  
{
   
$$$ x64
   
r @$t8 = 10
   
r @$t7 = 20
   
r @$t6 = 10
  
}
  
.else
  
{
   
$$$ x86
   
r @$t8 = 6
   
r @$t6 = 8
   
r @$t7 = 10
  
}
  
.foreach ($obj {!dumpheap -mt ${$arg1} -short})
  
{
   
$$ The !dumpheap -short option has last result as &#8212;&#8212;&#8212;&#8212;&#8212; and
   
$$ this .if is to avoid this
   
.if ($spat ("${$obj}","&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;"))
   
{}
   
.else
   
{
   
$$ $t5 contains refernce to the array which has key and value for the
   
$$ session contents

r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)
   
$$$ Iterating through the array elements
   
.for (r $t0=0; @$t0 < poi(@$t5+@$t9); r$t0=@$t0+1 )
   
{
   
.if(@$t0 = 0)
   
{
   
$$ First occurence of the element in the array would be in the 20 offset for x64 and 10 offset for x86
   
r$t1=@$t7
   
}
   
.else
   
{
   
$$ the rest of the elements would be in the 8th offset for x64 and 4th offset for x86
   
r$t1= @$t7+(@$t0*@$t9)
   
}
   
$$ Check for null before trying to dump
   
.if (poi((@$t5-@$t9)+@$t1) = 0 )
   
{
   
.continue
   
}
   
.else
   
{
   
.echo \***\***\***\***
   
$$ Session Key
   
.printf /ow "Session Key is :- "; !ds poi(poi((@$t5-@$t9)+@$t1)+@$t9)
   
$$ Session value
   
.printf /ow "Session value is :- ";!ds poi(poi((@$t5-@$t9)+@$t1)+@$t6)
   
}
   
}
   
}
  
}
  
[/sourcecode]

Copy the above script in to a file and invoke the script like this within Windbg

> $$>a<&#8220;c:Debuggersx86dumpsession.txt&#8221; 000007fef4115c20

Passing the MT of `System.Web.SessionState.HttpSessionState` as the script argument.

Within the script I am using the alias `!ds` for dumping strings instead of using `!dumpobj`.
  
To create the alias use this command in x64

[sourcecode]
  
as !ds .printf "%mu n", 10+
  
[/sourcecode]

and in x86

[sourcecode]
  
as !ds .printf "%mu n", C+
  
[/sourcecode]

Replace !ds with !do for dumping regular objects instead of strings.

Here is the output from the above script

> 0:022> $$>a<&#8220;c:Debuggersx86dumpsession.txt&#8221; 000007fef4115c20
  
> \***\***\***\***
  
> Session Key is :- Name
  
> Session value is :- Test
  
> \***\***\***\***
  
> Session Key is :- Name1
  
> Session value is :- Test1

If you are only interested in getting the session contents the above script should get you the answer you are looking for. The rest of the post is an explanation of how the script works.

I am going to start by explaining one of the important statement in the script “r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)” which gets the contents of the array that contains the session key and value.

The $obj is the loop variable that contains the object address for each Http Session object  “.foreach ($obj {!dumpheap -mt ${$arg1} -short})”

If I dump the Http session object using !do

> 0:022> !do 000000013fe20c30
  
> Name: System.Web.SessionState.HttpSessionState
  
> MethodTable: 000007fef4115c20
  
> EEClass: 000007fef3d73e00
  
> Size: 24(0x18) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef40b59e8  4001f59        <span style="color:#ff0000;"><strong>8</strong></span> &#8230;IHttpSessionState  0 instance 000000013fe20bc0 _container

We can see the 8th offset contains the pointer to the &#8220;_container&#8221; object in x64 and in x86 it will be the 4th offset and that&#8217;s the reason we use poi(${$obj}+@$t9) which should work for both x86 and x64 because the value of @$t9 is the pointer size which will be 4 in x86 and 8 in x64.

The next step is to dump the &#8220;_container&#8221; which is equal to poi(${$obj}+@$t9)

> 0:022> !do poi(000000013fe20c30+8)
  
> Name: System.Web.SessionState.HttpSessionStateContainer
  
> MethodTable: 000007fef411e868
  
> EEClass: 000007fef3d77348
  
> Size: 64(0x40) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001f5a        8        System.String  0 instance 0000000000000000 _id
  
> 000007fef4086508  4001f5b       <span style="color:#ff0000;"><strong>10 &#8230;ateItemCollection  0 instance 000000013fe20458 _sessionItems</strong></span>
  
> 000007fef4115390  4001f5c       18 &#8230;ObjectsCollection  0 instance 000000013fe209d0 _staticObjects
  
> 000007fef7b7ecf0  4001f5d       28         System.Int32  1 instance               20 _timeout
  
> 000007fef7b76c50  4001f5e       34       System.Boolean  1 instance                1 _newSession
  
> 000007fef411f8c8  4001f5f       2c         System.Int32  1 instance                1 _cookieMode
  
> 000007fef411f798  4001f60       30         System.Int32  1 instance                1 _mode
  
> 000007fef7b76c50  4001f61       35       System.Boolean  1 instance                0 _abandon
  
> 000007fef7b76c50  4001f62       36       System.Boolean  1 instance                0 _isReadonly
  
> 000007fef411e7b0  4001f63       20 &#8230;essionStateModule  0 instance 000000013fce1bd8 _stateModule

Now that we have the SessionContainer, we would have to get the contents of &#8220;_sessionItems&#8221; which is in the 10th offset in x64.

Next step is to dump &#8220;_sessionitems&#8221; using !do poi(poi(000000013fe20c30+8)+10) and this is equal to poi(poi(${$obj}+@$t9)+@$t6).In the starting of the script @$t6 is set to 10 or 8 based on platform.

> 0:022> !do poi(poi(000000013fe20c30+8)+10)
  
> Name: System.Web.SessionState.SessionStateItemCollection
  
> MethodTable: 000007fef4086650
  
> EEClass: 000007fef3d2fcf0
  
> Size: 112(0x70) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b76c50  400117b       44       System.Boolean  1 instance                0 _readOnly
  
> 000007fef7b7e968  400117c        <span style="color:#ff0000;"><strong>8 &#8230;ections.ArrayList  0 instance 000000013fe20830 _entriesArray</strong></span>
  
> 000007fef7b7fd88  400117d       10 &#8230;IEqualityComparer  0 instance 000000013fc6b270 _keyComparer
  
> 000007fef7b7f3d8  400117e       18 &#8230;ections.Hashtable  0 instance 000000013fe20858 _entriesTable
  
> 000007fef6f6f938  400117f       20 &#8230;e+NameObjectEntry  0 instance 0000000000000000 _nullKeyEntry
  
> 000007fef6f479b8  4001180       28 &#8230;se+KeysCollection  0 instance 0000000000000000 _keys
  
> 000007fef7b66840  4001181       30 &#8230;SerializationInfo  0 instance 0000000000000000 _serializationInfo
  
> 000007fef7b7ecf0  4001182       40         System.Int32  1 instance                3 _version
  
> 000007fef7b77370  4001183       38        System.Object  0 instance 0000000000000000 _syncRoot
  
> 000007fef7bbd028  4001184      a70 &#8230;em.StringComparer  0   shared           static defaultComparer
  
> >> Domain:Value  00000000010e2690:NotInit  0000000002e0a0a0:00000000ffae8cb8 <<
  
> 000007fef7b76c50  4001f67       45       System.Boolean  1 instance                1 _dirty
  
> 000007fef4108b20  4001f68       48 &#8230;n+KeyedCollection  0 instance 0000000000000000 _serializedItems
  
> 000007fef7b7aa30  4001f69       50     System.IO.Stream  0 instance 0000000000000000 _stream
  
> 000007fef7b7ecf0  4001f6a       60         System.Int32  1 instance                0 _iLastOffset
  
> 000007fef7b77370  4001f6b       58        System.Object  0 instance 000000013fe20818 _serializedItemsLock
  
> 000007fef7b7f3d8  4001f66     18e0 &#8230;ections.Hashtable  0   shared           static s_immutableTypes
  
> >> Domain:Value  00000000010e2690:NotInit  0000000002e0a0a0:000000013fe204c8 <<

Next field that we are interested in is &#8220;_entriesArray&#8221; which is in the 8th offset in x64. To dump its contents here is the command !do
  
poi(poi(poi(000000013fe20c30+8)+10)+8) which is equal to poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9

> 0:022> !do poi(poi(poi(000000013fe20c30+8)+10)+8)
  
> Name: System.Collections.ArrayList
  
> MethodTable: 000007fef7b7e968
  
> EEClass: 000007fef7781ee0
  
> Size: 40(0x28) bytes
  
> (C:WindowsassemblyGAC\_64mscorlib2.0.0.0\__b77a5c561934e089mscorlib.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b65870  400094c       <span style="color:#ff0000;"><strong> 8      System.Object[]  0 instance 000000013fe3ddb8 _items</strong></span>
  
> 000007fef7b7ecf0  400094d       18         System.Int32  1 instance                2 _size
  
> 000007fef7b7ecf0  400094e       1c         System.Int32  1 instance                2 _version
  
> 000007fef7b77370  400094f       10        System.Object  0 instance 0000000000000000 _syncRoot
  
> 000007fef7b65870  4000950      388      System.Object[]  0   shared           static emptyArray
  
> >> Domain:Value  00000000010e2690:00000000ffac6110 0000000002e0a0a0:00000000ffad19e0 <<

The next field we are interested  is &#8220;\_items&#8221; which is in the 8th offset in x64. Notice &#8220;\_items&#8221; is an array and cannot be dumped using !dumpobj or !do. So this command “r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)&#8221; will set the array pointer to$t5.

Now that we have array containing the session items, we could have used !da to
  
dump the array contents with details using !da -details poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)

> 0:022> !da -details poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)
  
> Name: System.Object[]
  
> MethodTable: 000007fef7b65870
  
> EEClass: 000007fef777eb58
  
> Size: 64(0x40) bytes
  
> Array: Rank 1, Number of elements 4, Type CLASS
  
> Element Methodtable: 000007fef7b77370
  
> [0] 000000013fe3dd98
  
> Name: System.Collections.Specialized.NameObjectCollectionBase+NameObjectEntry
  
> MethodTable: 000007fef6f6f938
  
> EEClass: 000007fef6ce90b0
  
> Size: 32(0x20) bytes
  
> (C:WindowsassemblyGAC\_MSILSystem2.0.0.0\__b77a5c561934e089System.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001185        <span style="color:#ff0000;"><strong>8        System.String  0 instance 000000013fe3dcf8 Key</strong></span>
  
> 000007fef7b77370  4001186       <span style="color:#ff0000;"><strong>10        System.Object  0 instance 000000013fe3dd20 Value</strong></span>
  
> [1] 000000013fe3ddf8
  
> Name: System.Collections.Specialized.NameObjectCollectionBase+NameObjectEntry
  
> MethodTable: 000007fef6f6f938
  
> EEClass: 000007fef6ce90b0
  
> Size: 32(0x20) bytes
  
> (C:WindowsassemblyGAC\_MSILSystem2.0.0.0\__b77a5c561934e089System.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001185        8        System.String  0 instance 000000013fe3dd48 Key
  
> 000007fef7b77370  4001186       10        System.Object  0 instance 000000013fe3dd70 Value
  
> [2] null
  
> [3] null

But notice it does not help much because we still cannot see the actual key and value. That is the reason for using a nested &#8220;.for&#8221; loop in the script which will iterate through the array contents. FYI @$t5 contains reference to the array.

The statement  &#8220;.for (r $t0=0; @$t0 < poi(@$t5+@$t9); r$t0=@$t0+1 )&#8221;  is standard for loop with one thing that is special which is poi(@$t5+@$t9).  The poi(@$t5+@$t9) contains the reference to the size of the array. How do I know that? The answer is dd poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)

> 0:022> dd poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)
  
> 00000001\`3fe3ddb8  f7b65870 000007fe <span style="color:#ff0000;"><strong>00000004</strong></span> 00000000
  
> 00000001\`3fe3ddc8  f7b77370 000007fe 3fe3dd98 00000001
  
> 00000001\`3fe3ddd8  3fe3ddf8 00000001 00000000 00000000
  
> 00000001\`3fe3dde8  00000000 00000000 00000000 00000000
  
> 00000001\`3fe3ddf8  f6f6f938 000007fe 3fe3dd48 00000001
  
> 00000001\`3fe3de08  3fe3dd70 00000001 00000000 00000000
  
> 00000001\`3fe3de18  f7b77370 000007fe 00000000 00000000
  
> 00000001\`3fe3de28  00000000 80000000 f7b77a80 000007fe

Notice the 8th offset value is 00000004 which is the size of the array and for
  
more information look at the post on custom dump array [This post is going to be about dumping ASP.NET session objects using Windbg. I had recently answered a stackoverflow question in which someone wanted to dump ASP.NET session objects for 64-bit IIS (x64). I thought why not blog about the same which might be useful to others.  The challenge is to write one script that should work in both x86/x64.  FYI there is a script from [Tess][1] that does dump out the session contents, AFAIK it will not work on x64 and my script iterates through the array using the array length instead of using “.foreach /pS 2 /ps 99” which is somewhat cleaner.

Here is the script for dumping ASP.NET session objects within Windbg / CDB

[sourcecode wraplines=&#8221;true&#8221;]

$$$ Dump the ASP.NET Session objects within windbg/cdb
  
$$$ Platform : x86 / x64
  
$$$ Naveen Srinivasan http://naveensrinivasan.com
  
$$$ Usage: $$>a<"c:Debuggersx86dumpsession.txt" 000007fef4115c20
  
$$$ where 000007fef4115c20 is the MethodTable pointer System.Web.SessionState.HttpSessionState

r @$t9 = @$ptrsize
  
$$ $t9 register contains pointer size
  
$$ $t8 register contains the next offset of the variable
  
$$ $t7 register contains array start address

.if (@$ptrsize = 8 )
  
{
   
$$$ x64
   
r @$t8 = 10
   
r @$t7 = 20
   
r @$t6 = 10
  
}
  
.else
  
{
   
$$$ x86
   
r @$t8 = 6
   
r @$t6 = 8
   
r @$t7 = 10
  
}
  
.foreach ($obj {!dumpheap -mt ${$arg1} -short})
  
{
   
$$ The !dumpheap -short option has last result as &#8212;&#8212;&#8212;&#8212;&#8212; and
   
$$ this .if is to avoid this
   
.if ($spat ("${$obj}","&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;"))
   
{}
   
.else
   
{
   
$$ $t5 contains refernce to the array which has key and value for the
   
$$ session contents

r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)
   
$$$ Iterating through the array elements
   
.for (r $t0=0; @$t0 < poi(@$t5+@$t9); r$t0=@$t0+1 )
   
{
   
.if(@$t0 = 0)
   
{
   
$$ First occurence of the element in the array would be in the 20 offset for x64 and 10 offset for x86
   
r$t1=@$t7
   
}
   
.else
   
{
   
$$ the rest of the elements would be in the 8th offset for x64 and 4th offset for x86
   
r$t1= @$t7+(@$t0*@$t9)
   
}
   
$$ Check for null before trying to dump
   
.if (poi((@$t5-@$t9)+@$t1) = 0 )
   
{
   
.continue
   
}
   
.else
   
{
   
.echo \***\***\***\***
   
$$ Session Key
   
.printf /ow "Session Key is :- "; !ds poi(poi((@$t5-@$t9)+@$t1)+@$t9)
   
$$ Session value
   
.printf /ow "Session value is :- ";!ds poi(poi((@$t5-@$t9)+@$t1)+@$t6)
   
}
   
}
   
}
  
}
  
[/sourcecode]

Copy the above script in to a file and invoke the script like this within Windbg

> $$>a<&#8220;c:Debuggersx86dumpsession.txt&#8221; 000007fef4115c20

Passing the MT of `System.Web.SessionState.HttpSessionState` as the script argument.

Within the script I am using the alias `!ds` for dumping strings instead of using `!dumpobj`.
  
To create the alias use this command in x64

[sourcecode]
  
as !ds .printf "%mu n", 10+
  
[/sourcecode]

and in x86

[sourcecode]
  
as !ds .printf "%mu n", C+
  
[/sourcecode]

Replace !ds with !do for dumping regular objects instead of strings.

Here is the output from the above script

> 0:022> $$>a<&#8220;c:Debuggersx86dumpsession.txt&#8221; 000007fef4115c20
  
> \***\***\***\***
  
> Session Key is :- Name
  
> Session value is :- Test
  
> \***\***\***\***
  
> Session Key is :- Name1
  
> Session value is :- Test1

If you are only interested in getting the session contents the above script should get you the answer you are looking for. The rest of the post is an explanation of how the script works.

I am going to start by explaining one of the important statement in the script “r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)” which gets the contents of the array that contains the session key and value.

The $obj is the loop variable that contains the object address for each Http Session object  “.foreach ($obj {!dumpheap -mt ${$arg1} -short})”

If I dump the Http session object using !do

> 0:022> !do 000000013fe20c30
  
> Name: System.Web.SessionState.HttpSessionState
  
> MethodTable: 000007fef4115c20
  
> EEClass: 000007fef3d73e00
  
> Size: 24(0x18) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef40b59e8  4001f59        <span style="color:#ff0000;"><strong>8</strong></span> &#8230;IHttpSessionState  0 instance 000000013fe20bc0 _container

We can see the 8th offset contains the pointer to the &#8220;_container&#8221; object in x64 and in x86 it will be the 4th offset and that&#8217;s the reason we use poi(${$obj}+@$t9) which should work for both x86 and x64 because the value of @$t9 is the pointer size which will be 4 in x86 and 8 in x64.

The next step is to dump the &#8220;_container&#8221; which is equal to poi(${$obj}+@$t9)

> 0:022> !do poi(000000013fe20c30+8)
  
> Name: System.Web.SessionState.HttpSessionStateContainer
  
> MethodTable: 000007fef411e868
  
> EEClass: 000007fef3d77348
  
> Size: 64(0x40) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001f5a        8        System.String  0 instance 0000000000000000 _id
  
> 000007fef4086508  4001f5b       <span style="color:#ff0000;"><strong>10 &#8230;ateItemCollection  0 instance 000000013fe20458 _sessionItems</strong></span>
  
> 000007fef4115390  4001f5c       18 &#8230;ObjectsCollection  0 instance 000000013fe209d0 _staticObjects
  
> 000007fef7b7ecf0  4001f5d       28         System.Int32  1 instance               20 _timeout
  
> 000007fef7b76c50  4001f5e       34       System.Boolean  1 instance                1 _newSession
  
> 000007fef411f8c8  4001f5f       2c         System.Int32  1 instance                1 _cookieMode
  
> 000007fef411f798  4001f60       30         System.Int32  1 instance                1 _mode
  
> 000007fef7b76c50  4001f61       35       System.Boolean  1 instance                0 _abandon
  
> 000007fef7b76c50  4001f62       36       System.Boolean  1 instance                0 _isReadonly
  
> 000007fef411e7b0  4001f63       20 &#8230;essionStateModule  0 instance 000000013fce1bd8 _stateModule

Now that we have the SessionContainer, we would have to get the contents of &#8220;_sessionItems&#8221; which is in the 10th offset in x64.

Next step is to dump &#8220;_sessionitems&#8221; using !do poi(poi(000000013fe20c30+8)+10) and this is equal to poi(poi(${$obj}+@$t9)+@$t6).In the starting of the script @$t6 is set to 10 or 8 based on platform.

> 0:022> !do poi(poi(000000013fe20c30+8)+10)
  
> Name: System.Web.SessionState.SessionStateItemCollection
  
> MethodTable: 000007fef4086650
  
> EEClass: 000007fef3d2fcf0
  
> Size: 112(0x70) bytes
  
> (C:WindowsassemblyGAC\_64System.Web2.0.0.0\__b03f5f7f11d50a3aSystem.Web.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b76c50  400117b       44       System.Boolean  1 instance                0 _readOnly
  
> 000007fef7b7e968  400117c        <span style="color:#ff0000;"><strong>8 &#8230;ections.ArrayList  0 instance 000000013fe20830 _entriesArray</strong></span>
  
> 000007fef7b7fd88  400117d       10 &#8230;IEqualityComparer  0 instance 000000013fc6b270 _keyComparer
  
> 000007fef7b7f3d8  400117e       18 &#8230;ections.Hashtable  0 instance 000000013fe20858 _entriesTable
  
> 000007fef6f6f938  400117f       20 &#8230;e+NameObjectEntry  0 instance 0000000000000000 _nullKeyEntry
  
> 000007fef6f479b8  4001180       28 &#8230;se+KeysCollection  0 instance 0000000000000000 _keys
  
> 000007fef7b66840  4001181       30 &#8230;SerializationInfo  0 instance 0000000000000000 _serializationInfo
  
> 000007fef7b7ecf0  4001182       40         System.Int32  1 instance                3 _version
  
> 000007fef7b77370  4001183       38        System.Object  0 instance 0000000000000000 _syncRoot
  
> 000007fef7bbd028  4001184      a70 &#8230;em.StringComparer  0   shared           static defaultComparer
  
> >> Domain:Value  00000000010e2690:NotInit  0000000002e0a0a0:00000000ffae8cb8 <<
  
> 000007fef7b76c50  4001f67       45       System.Boolean  1 instance                1 _dirty
  
> 000007fef4108b20  4001f68       48 &#8230;n+KeyedCollection  0 instance 0000000000000000 _serializedItems
  
> 000007fef7b7aa30  4001f69       50     System.IO.Stream  0 instance 0000000000000000 _stream
  
> 000007fef7b7ecf0  4001f6a       60         System.Int32  1 instance                0 _iLastOffset
  
> 000007fef7b77370  4001f6b       58        System.Object  0 instance 000000013fe20818 _serializedItemsLock
  
> 000007fef7b7f3d8  4001f66     18e0 &#8230;ections.Hashtable  0   shared           static s_immutableTypes
  
> >> Domain:Value  00000000010e2690:NotInit  0000000002e0a0a0:000000013fe204c8 <<

Next field that we are interested in is &#8220;_entriesArray&#8221; which is in the 8th offset in x64. To dump its contents here is the command !do
  
poi(poi(poi(000000013fe20c30+8)+10)+8) which is equal to poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9

> 0:022> !do poi(poi(poi(000000013fe20c30+8)+10)+8)
  
> Name: System.Collections.ArrayList
  
> MethodTable: 000007fef7b7e968
  
> EEClass: 000007fef7781ee0
  
> Size: 40(0x28) bytes
  
> (C:WindowsassemblyGAC\_64mscorlib2.0.0.0\__b77a5c561934e089mscorlib.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b65870  400094c       <span style="color:#ff0000;"><strong> 8      System.Object[]  0 instance 000000013fe3ddb8 _items</strong></span>
  
> 000007fef7b7ecf0  400094d       18         System.Int32  1 instance                2 _size
  
> 000007fef7b7ecf0  400094e       1c         System.Int32  1 instance                2 _version
  
> 000007fef7b77370  400094f       10        System.Object  0 instance 0000000000000000 _syncRoot
  
> 000007fef7b65870  4000950      388      System.Object[]  0   shared           static emptyArray
  
> >> Domain:Value  00000000010e2690:00000000ffac6110 0000000002e0a0a0:00000000ffad19e0 <<

The next field we are interested  is &#8220;\_items&#8221; which is in the 8th offset in x64. Notice &#8220;\_items&#8221; is an array and cannot be dumped using !dumpobj or !do. So this command “r$t5 = poi(poi(poi(poi(${$obj}+@$t9)+@$t6)+@$t9)+@$t9)&#8221; will set the array pointer to$t5.

Now that we have array containing the session items, we could have used !da to
  
dump the array contents with details using !da -details poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)

> 0:022> !da -details poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)
  
> Name: System.Object[]
  
> MethodTable: 000007fef7b65870
  
> EEClass: 000007fef777eb58
  
> Size: 64(0x40) bytes
  
> Array: Rank 1, Number of elements 4, Type CLASS
  
> Element Methodtable: 000007fef7b77370
  
> [0] 000000013fe3dd98
  
> Name: System.Collections.Specialized.NameObjectCollectionBase+NameObjectEntry
  
> MethodTable: 000007fef6f6f938
  
> EEClass: 000007fef6ce90b0
  
> Size: 32(0x20) bytes
  
> (C:WindowsassemblyGAC\_MSILSystem2.0.0.0\__b77a5c561934e089System.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001185        <span style="color:#ff0000;"><strong>8        System.String  0 instance 000000013fe3dcf8 Key</strong></span>
  
> 000007fef7b77370  4001186       <span style="color:#ff0000;"><strong>10        System.Object  0 instance 000000013fe3dd20 Value</strong></span>
  
> [1] 000000013fe3ddf8
  
> Name: System.Collections.Specialized.NameObjectCollectionBase+NameObjectEntry
  
> MethodTable: 000007fef6f6f938
  
> EEClass: 000007fef6ce90b0
  
> Size: 32(0x20) bytes
  
> (C:WindowsassemblyGAC\_MSILSystem2.0.0.0\__b77a5c561934e089System.dll)
  
> Fields:
  
> MT    Field   Offset                 Type VT     Attr            Value Name
  
> 000007fef7b77a80  4001185        8        System.String  0 instance 000000013fe3dd48 Key
  
> 000007fef7b77370  4001186       10        System.Object  0 instance 000000013fe3dd70 Value
  
> [2] null
  
> [3] null

But notice it does not help much because we still cannot see the actual key and value. That is the reason for using a nested &#8220;.for&#8221; loop in the script which will iterate through the array contents. FYI @$t5 contains reference to the array.

The statement  &#8220;.for (r $t0=0; @$t0 < poi(@$t5+@$t9); r$t0=@$t0+1 )&#8221;  is standard for loop with one thing that is special which is poi(@$t5+@$t9).  The poi(@$t5+@$t9) contains the reference to the size of the array. How do I know that? The answer is dd poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)

> 0:022> dd poi(poi(poi(poi(000000013fe20c30+8)+10)+8)+8)
  
> 00000001\`3fe3ddb8  f7b65870 000007fe <span style="color:#ff0000;"><strong>00000004</strong></span> 00000000
  
> 00000001\`3fe3ddc8  f7b77370 000007fe 3fe3dd98 00000001
  
> 00000001\`3fe3ddd8  3fe3ddf8 00000001 00000000 00000000
  
> 00000001\`3fe3dde8  00000000 00000000 00000000 00000000
  
> 00000001\`3fe3ddf8  f6f6f938 000007fe 3fe3dd48 00000001
  
> 00000001\`3fe3de08  3fe3dd70 00000001 00000000 00000000
  
> 00000001\`3fe3de18  f7b77370 000007fe 00000000 00000000
  
> 00000001\`3fe3de28  00000000 80000000 f7b77a80 000007fe

Notice the 8th offset value is 00000004 which is the size of the array and for
  
more information look at the post on custom dump array][2] 
  
The first element in the array  would be in the 20th offset in x64 and 10th
  
offset in x86 and that is the reason for the &#8220;.if(@$t0=0)&#8221;

> .if(@$t0 = 0)
  
> {
  
> $$ First occurence of the element in the array would be in the 20 offset for x64 and 10 offset for x86
  
> r$t1=@$t7
  
> }

So the first time the value @$t1 would be 20. And the rest of the elements would be in the 8th offset in x64 and 4th offset inx86

> .else
  
> {
  
> $$ the rest of the elements would be in the 8th offset for x64 and 4th offset for x86
  
> r$t1= @$t7+(@$t0*@$t9)
  
> }

So the second time it would be 20+(1\*8) = @$t7+(@$t0\*@$t9) which will be 28th offset.

The next statement &#8220;.if (poi((@$t5-@$t9)+@$t1) = 0 )&#8221; is null check , this would avoid dumping an object which has not be initialized. This is because not all elements in the array could have been initialized.

The !ds poi(poi((@$t5-@$t9)+@$t1)+@$t9) gets the session key which is in the 8th offset (look at the previous output from dumparray) and the !ds poi(poi((@$t5-@$t9)+@$t1)+@$t6) gets the session value which is in the 10th offset.

Here is my initial x64 specific script that I wrote.

[sourcecode]
  
foreach ($obj {!dumpheap -mt ${$arg1} -short})
  
{
  
$$ The !dumpheap -short option has last result as &#8212;&#8212;&#8212;&#8212;&#8212; and
  
$$ this .if is to avoid this
  
.if ($spat ("${$obj}","&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;"))
  
{}
  
.else
  
{
  
$$ $t5 contains reference to the array which has key and value for the
  
$$ session contents

r$t5 = poi(poi(poi(poi(${$obj}+0x8)+0x10)+0x8)+0x8);
  
r$t1 = 0
  
.for (r $t0=0; @$t0 < poi(@$t5+0x8); r$t0=@$t0+1 )
  
{
  
.if(@$t0 = 0)
  
{
  
$$ First occurrence of the element in the array would be in the 20 offset
  
r$t1=20
  
}
  
.else
  
{
  
$$ the rest of the elements would be in the 8th offset
  
r$t1= 20+(@$t0*8)
  
};

$$ Check for null before trying to dump

.if (poi((@$t5-0x8)+@$t1) = 0 )
  
{
  
.continue
  
}
  
.else
  
{
  
.echo \***\***\***\***;
  
? @$t0
  
$$ Session Key
  
.printf "Session Key is :- "; !ds poi(poi((@$t5-0x8)+@$t1)+0x8);
  
$$ Session value
  
.printf "Session value is :- ";!ds poi(poi((@$t5-0x8)+@$t1)+0x10)
  
}
  
}
  
}
  
}
  
[/sourcecode]

I had fun writing this script. Let me know if there is a better way to write this.

 [1]: http://blogs.msdn.com/b/tess/archive/2007/09/18/debugging-script-dumping-out-asp-net-session-contents.aspx
 [2]: http://naveensrinivasan.com/2010/06/24/custom-dumparray-windbg/