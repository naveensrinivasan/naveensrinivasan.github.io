---
title: Conditional BreakPoint based on callstack within Windbg – .NET
author: admin
type: post
date: 2010-12-29T02:00:12+00:00
url: /?p=1314
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168817;}'
categories:
  - .NET
  - Windbg

---
Someone recently asked me &#8220;How to have a break-point on a method based on certain function in the call-stack?&#8221;

Here is the sample code to demonstrate this

[sourcecode language=&#8221;csharp&#8221;]
  
using System;
  
using System.Threading.Tasks;
  
using System.Data.SqlClient;
  
namespace Test
  
{
      
class Program
      
{
          
string connectionString = @"Data Source=.sqlexpress;Initial Catalog=Tfs_Configuration;Integrated Security=True";
          
public void Bar()
          
{
              
using (var c = new SqlConnection(connectionString))
              
{
                  
c.Open();
                  
var command = new SqlCommand(@"update [tbl_AccessMapping] set [DisplayName] = @param", c);
                  
command.Parameters.Add(new SqlParameter("param", "Bar"));
                  
command.ExecuteNonQuery();
              
}
          
}
          
public void Foo()
          
{
              
using (var c = new SqlConnection(connectionString))
              
{
                  
c.Open();
                  
var command = new SqlCommand(@"update [tbl_AccessMapping] set [DisplayName] = @param", c);
                  
command.Parameters.Add(new SqlParameter("param", "Foo"));
                  
command.ExecuteNonQuery();
              
}
          
}
          
static void Main(string[] args1)
          
{
              
var s = new Program();
              
Parallel.For(0, 2, (i) => s.Bar());
              
Parallel.For(0, 2, (i) => s.Foo());
              
Console.Read();
          
}
      
}
  
}

[/sourcecode]

The requirement is to have a break-point on &#8220;ExecuteNonQuery&#8221; but it should break only if it is invoked from &#8220;Foo&#8221; and not from &#8220;Bar&#8221;.

Launched the exe within windbg and loaded sos,sosex and set a bp on System.Data.SqlClient.SqlCommand.ExecuteNonQuery suing !mbm

And when the break-point hits the first time updated the bp using

> bs 0  $$>a<&#8220;d:Debuggersx86ConditionalBP.txt&#8221; Foo

Here are the contents of ConditionalBP.txt

[sourcecode]
  
ad /q Contains
  
aS /c Contains .shell -ci "!CLRStack" FINDSTR $arg1
  
.block {
              
.if ($spat("${Contains}","\*${$arg1}\*"))
                  
{
                   
!CLRStack
                  
}
             
.else
                  
{
                  
g
                  
}
       
}
  
ad /q Contains
  
[/sourcecode]