---
title: Windbg trick – Having custom name for user-defined pseudo-registers
author: admin
type: post
date: 2010-11-23T01:07:53+00:00
url: /?p=1246
reddit:
  - 'a:2:{s:5:"count";i:0;s:4:"time";i:1433168819;}'
categories:
  - Windbg

---
There are 20 [user-defined pseudo-registers][1] (**$t0**, **$t1**, &#8230;, **$t19**) in windbg/cdb . To have scripts with variable names as @$t0 and @$t1 isn&#8217;t helpful for readability. The trick to avoid this is by using the &#8220;aS&#8221; command.

Here is an example, for a loop variable I would like to use a variable name like &#8220;i&#8221; instead of &#8220;@$t0&#8221; and to use &#8220;i&#8221;  as a variable  here is the command

> aS i &#8220;@$t0&#8221;

Now&#8221;i&#8221; is just an alias for &#8220;@$t0&#8221;.  Here is another example of using &#8220;i&#8221; in the comparison statement

> j (${i} =0) &#8216;.echo is zero&#8217; ; &#8216;.echo is not zero&#8217;

This is the command to remove the alias without evaluating it.

> ad ${/v:i}

 [1]: http://msdn.microsoft.com/en-us/library/ff553485(VS.85).aspx