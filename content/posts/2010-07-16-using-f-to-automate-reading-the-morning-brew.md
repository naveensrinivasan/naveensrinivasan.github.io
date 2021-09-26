---
title: 'Using F# to Automate Reading–The Morning Brew'
author: admin
type: post
date: 2010-07-16T04:27:24+00:00
url: /?p=997
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1431702072;}'
categories:
  - 'F#'

---
I guess most of the .NET Devs read [The Morning Brew][1], if not you should.  It is a morning newspaper for the dev, so I end up reading it first thing when I go to work. I like to  try and automate most of the stuff . So I thought why not write a script that reads the Morning Brew feed, filter the excluded content that I am not interested in and open the urls before I come in. The reason behind using F# is I don’t have to compile the code. I could use it with FSI.exe. Incase if the extraction logic changes and I don’t have to recompile , it is just fixing the script. The best part of writing in F# is I avoid all the ceremony. Here is the F# code, it is nothing fancy. And this code can easily be modified for other link collection sites.

[sourcecode]

#r @"C:WindowsMicrosoft.NETFrameworkv4.0.30319System.ServiceModel.dll"
  
#r @"C:WindowsMicrosoft.NETFrameworkv4.0.30319System.Xml.Linq.dll"

open System.IO
  
open System.ServiceModel.Syndication
  
open System.Web
  
open System.Xml
  
open System.Xml.Linq
  
open System.Diagnostics

let formatter = new Rss20FeedFormatter()
  
let sw = new StringWriter();
  
// Excluded Keywords
  
let exists (item:string) = File.ReadAllLines @"c:Usersnaveenexclude.txt" |>
                               
Seq.exists(fun e -> item.ToLower().Contains(e.ToLower()))

formatter.ReadFrom(XmlReader.Create "http://feeds.feedburner.com/ReflectivePerspective")
  
formatter.Feed.Items |> Seq.nth 0 |> fun i -> i.SaveAsRss20(new XmlTextWriter(sw))

let data = HttpUtility.HtmlDecode(sw.ToString())
  
XDocument.Parse(data).Descendants(XName.Get("li")) |>
             
Seq.filter(fun i -> not (exists i.Value)) |>
             
Seq.map(fun i -> i.Descendants(XName.Get("a"))) |>
             
Seq.iter(fun x -> x |> Seq.iter(fun y ->
              
try
                 
System.Diagnostics.Process.Start (y.Attribute(XName.Get("href")).Value) |> ignore
             
with
             
|ex -> printfn "Error : %s" ex.Message))
  
[/sourcecode]

My excluded file at the moment contains javascript and jquery. I am not much into these. Here is the command line

[sourcecode]
  
"C:Program Files (x86)Microsoft F#v4.0Fsi.exe" &#8211;quiet &#8211;exec "C:Usersnaveenmorningbrew.fsx"
  
[/sourcecode]

[<img class="alignnone size-full wp-image-1011" title="TheMorningBrew" src="http://104.197.135.42/wp-content/uploads/2010/07/themorningbrew2.jpg" alt="" width="700" height="418" />][2]

 [1]: http://blog.cwa.me.uk/
 [2]: http://104.197.135.42/wp-content/uploads/2010/07/themorningbrew2.jpg