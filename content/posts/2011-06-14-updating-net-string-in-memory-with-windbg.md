---
title: Updating .NET String in memory with Windbg
author: admin
type: post
date: 2011-06-14T16:42:24+00:00
url: /?p=1353
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168816;}'
tagazine-media:
  - 'a:7:{s:7:"primary";s:69:"http://naveensrinivasan.files.wordpress.com/2011/06/updatestring1.jpg";s:6:"images";a:2:{s:69:"http://naveensrinivasan.files.wordpress.com/2011/06/updatestring1.jpg";a:6:{s:8:"file_url";s:69:"http://naveensrinivasan.files.wordpress.com/2011/06/updatestring1.jpg";s:5:"width";s:3:"301";s:6:"height";s:3:"300";s:4:"type";s:5:"image";s:4:"area";s:5:"90300";s:9:"file_path";s:0:"";}s:59:"http://naveensrinivasan.files.wordpress.com/2011/06/bad.jpg";a:6:{s:8:"file_url";s:59:"http://naveensrinivasan.files.wordpress.com/2011/06/bad.jpg";s:5:"width";s:3:"301";s:6:"height";s:3:"300";s:4:"type";s:5:"image";s:4:"area";s:5:"90300";s:9:"file_path";s:0:"";}}s:6:"videos";a:0:{}s:11:"image_count";s:1:"2";s:6:"author";s:8:"11476521";s:7:"blog_id";s:8:"11168708";s:9:"mod_stamp";s:19:"2011-06-14 16:42:24";}'
categories:
  - .NET
  - Windbg

---
In this post I would show a simple trick to update .NET strings in memory with Windbg. The caveat is make sure the string that you&#8217;re updating is long enough to fit into the string buffer. If not there would be a memory corruption.

Here is a simple windows form application with title &#8220;Good&#8221;

[<img class="alignnone size-full wp-image-1356" title="updatestring" src="http://104.197.135.42/wp-content/uploads/2011/06/updatestring13.jpg" alt="" width="301" height="300" />][1]

The goal is to update the title from &#8220;Good&#8221; to &#8220;Bad&#8221;.

[sourcecode language=&#8221;csharp&#8221;]

button1.Click += (s,b) => Text = _caption;

[/sourcecode]

<span class="Apple-style-span" style="font-family:Georgia, 'Times New Roman', 'Bitstream Charter', Times, serif;font-size:13px;line-height:19px;white-space:normal;">I am updating the title in the button click.</span>

Here is the actual string object within the debugger

<pre>0:006&gt; !do 0294d0a0
Name:        System.String
MethodTable: 59b9fb64
EEClass:     598d8bb0
Size:        22(0x16) bytes
File:        C:WindowsMicrosoft.NetassemblyGAC_32mscorlib
v4.0_4.0.0.0__b77a5c561934e089mscorlib.dll
String:      Good
Fields:
      MT    Field   Offset                 Type VT     Attr    Value Name
59ba2b30  40000ed        4         System.Int32  1 instance        4 m_stringLength
59ba1f80  40000ee        8          System.Char  1 instance       47 m_firstChar
59b9fb64  40000ef        8        System.String  0   shared   static Empty
    &gt;&gt; Domain:Value  004b0308:02941228 &lt;&lt;</pre>

I would be using the <a title="e" href="http://msdn.microsoft.com/en-us/library/ff545308(VS.85).aspx" target="_blank">e</a>  command to update the memory. The **ezu **command is used for updating  Null-terminated Unicode string .

Notice the first character starts in the 8th offset from the above. So we would have start updating the string only from the 8th offset. The first 8 bytes of object are for syncblock index and method table pointer.

Here is the command to update the string memory.

> ezu 0294d0a0+8 &#8220;Bad&#8221;

And the updated form title.

[<img class="alignnone size-full wp-image-1358" title="bad" src="http://104.197.135.42/wp-content/uploads/2011/06/bad2.jpg" alt="" width="301" height="300" />][2]

 [1]: http://104.197.135.42/wp-content/uploads/2011/06/updatestring13.jpg
 [2]: http://104.197.135.42/wp-content/uploads/2011/06/bad2.jpg