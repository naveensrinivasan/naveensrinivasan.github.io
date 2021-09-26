---
title: 'Dumping Generic List  in .NET within Windbg'
author: admin
type: post
date: 2010-12-10T03:15:43+00:00
url: /?p=1281
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168818;}'
categories:
  - .NET
  - Windbg

---
Most of the code uses List<T> for storing items.  The present solutions don&#8217; t have a way to dump List<T> within windbg. Even though sosex has an option to dump the List<T> using !mdt it still doesn&#8217;t meet the scripting requirements. For example here is an output using sosex &#8220;!mdt -e 029a91c0&#8221;

> 0:000> !mdt -e 029a91c0
  
> 029a91c0 (System.Collections.Generic.List\`1[[Test.Foo, Test]])
  
> Count = 2
  
> [0] 029a9200 (Test.Foo)
  
> [1] 029a9210 (Test.Foo)

I would have preferred to get the contents of the &#8220;Foo&#8221; object instead of just the address of Foo. So wrote a script to do that.

[sourcecode]

$$ pointer to the array within the List
  
r @$t5 = poi(${$arg1}+@$ptrsize)

.if (@$ptrsize = 8 )
   
{
      
r @t7 = 20
   
}
   
.else
   
{
      
r @$t7 = 10
   
}

.for (r $t0=0; @$t0 < poi(@$t5+@$ptrsize); r$t0=@$t0+1 )
   
{
       
.if(@$t0 = 0)
       
{
           
$$ First occurence of the element in the array would be in the 20 offset for x64 and 10 offset for x86
           
r$t1=@$t7
       
}
       
.else
       
{
           
$$ the rest of the elements would be in the 8th offset for x64 and 4th offset for x86
           
r$t1= @$t7+(@$t0*@$ptrsize)
       
}
       
$$ Check for null before trying to dump
       
.if (poi((@$t5-@$ptrsize)+@$t1) = 0 )
       
{
       
.continue
       
}
       
.else
       
{
       
.printf &quot;%N n&quot; ,poi((@$t5-@$ptrsize)+@$t1)
       
}
   
} 

[/sourcecode]

This script should work in x86 and x64. To use the above script copy to a file and invoke it like this passing the address of List<T>

> $$>a<"d:Debuggersx86dumplist.txt" 029a91c0

Here is the output from the above command.

> 0:000> $$>a<"d:Debuggersx86dumplist.txt" 029a91c0
  
> 029A9200
  
> 029A9210 

Now with this script I can use !mdt to get the contents of the &#8220;Foo&#8221; object. 

[sourcecode]
  
.foreach ($obj {$$>a<"d:Debuggersx86dumplist.txt" 029a91c0}) {!mdt $obj}
  
[/sourcecode]

> 0:000> .foreach ($obj {$$>a<"d:Debuggersx86dumplist.txt" 029a91c0}) {!mdt $obj}
  
> 029a9200 (Test.Foo)
      
> counter:0x1 (System.Int32)
      
> Name:029a917c (System.String: "test")
  
> 029a9210 (Test.Foo)
      
> counter:0x2 (System.Int32)
      
> Name:029a9198 (System.String: "test2")

This is one of the scripts that I would use often. Hope it is useful to others also.