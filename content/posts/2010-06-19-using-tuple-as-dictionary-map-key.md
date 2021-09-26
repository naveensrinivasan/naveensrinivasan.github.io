---
title: Using Tuple as Dictionary / Map key
author: admin
type: post
date: 2010-06-19T12:37:53+00:00
url: /?p=829
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1349659088;}'
categories:
  - .NET

---
I  recently had to create a dictionary which needed a multipart key like <string,int> . To do this I would have to create a custom class  override equals and gethashcode. That’s when someone told I could use Tuple, but weren’t sure it was possible.  Here was the quick sample to try it in F#

[sourcecode]
  
let x = [("naveen",1),1;("naveen",1),2] |> Map.ofList
  
[/sourcecode]

as expected only one item in the Map

> val x : Map<(string * int),int> = map [((&#8220;naveen&#8221;, 1), 2)]

And the same in C#

[sourcecode language=&#8221;csharp&#8221;]
   
Console.WriteLine(Tuple.Create("Naveen",1).Equals( Tuple.Create("Naveen",1)));
  
[/sourcecode]

The next step was to actually disassemble Tuple in reflector

[<img class="alignnone size-full wp-image-830" title="Tuple" src="http://104.197.135.42/wp-content/uploads/2010/06/tuple2.jpg" alt="" width="450" height="268" />][1]

The code implements IStructuralComparable.CompareTo and does the comparison for each item. This is one of the reasons why generics is cool.

 [1]: http://104.197.135.42/wp-content/uploads/2010/06/tuple2.jpg