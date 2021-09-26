---
title: Managed Stack Explorer using ClrMD
author: admin
type: post
date: 2015-05-14T23:14:45+00:00
url: /?p=1457
publicize_twitter_user:
  - snaveen
publicize_twitter_url:
  - http://t.co/uzGzKiErwh
categories:
  - .NET

---
How often we run into an issue in the field where we just want to see the managed call-stack where the exception is or where the thread is hung. One of the options is debugger or something like ETW.

So I built a managed stack explorer

<a title="http://naveensrinivasan.github.io/ManagedStackExplorer/" href="http://naveensrinivasan.github.io/ManagedStackExplorer/" target="_blank">http://naveensrinivasan.github.io/ManagedStackExplorer/</a>

[<img class="alignleft" src="https://raw.githubusercontent.com/naveensrinivasan/ManagedStackExplorer/master/screenshot.jpg" alt="" width="1204" height="1560" />][1]

Managed Stack Explorer provides call stack for .NET applications using managed code with thread local variables. The API as of now does not provide values for the local variables. When it is available we can update it.

It is a single executable without any other dll&#8217;s. It uses [Costura][2] to embed dll.

The debug shim works specific to processor and would not be able to get call-stacks if it is not. So x86 exe cannot get call-stacks of x64. That&#8217;s reason for x86 and x64 specific exe&#8217;s. There is no difference in code other than how it is compiled.

It is work in progress and I would love some feedback and code contributions.

The github site also has link to download the pre-complied executables.

 [1]: https://raw.githubusercontent.com/naveensrinivasan/ManagedStackExplorer/master/screenshot.jpg
 [2]: https://github.com/Fody/Costura