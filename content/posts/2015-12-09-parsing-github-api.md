---
title: Parsing GitHub API
author: admin
type: post
date: 2015-12-09T16:46:57+00:00
url: /?p=2571
categories:
  - GitHub
tags:
  - GitHub

---
I have been contributing to <a href="https://github.com/octokit/octokit.net" target="_blank">https://github.com/octokit/octokit.net</a> project. It is the API for accessing GitHub. One of the recent questions that came up was to get the list of <a href="https://github.com/octokit/octokit.net/issues/968" target="_blank">https://github.com/octokit/octokit.net/issues/968</a>.

The API&#8217;s are published in HTML <a href="https://developer.github.com/v3/" target="_blank">https://developer.github.com/v3/</a>

There are about 63 + categories that have API. Wanted to parse all of these with least manual intervention.

[gist id = &#8220;58fa8eb57e61535f10db&#8221;]

The above goes to the main URL looks for all sub-categories and download&#8217;s each of the web pages and extracts <span class="pl-s"><span class="pl-pds">&#8220;</span><em>GET<span class="pl-pds">&#8220;</span></em></span>_, <span class="pl-s"><span class="pl-pds">&#8220;</span>DELETE<span class="pl-pds">&#8220;</span></span>, <span class="pl-s"><span class="pl-pds">&#8220;</span>PATCH<span class="pl-pds">&#8220;</span></span>, <span class="pl-s"><span class="pl-pds">&#8220;</span>POST<span class="pl-pds">&#8220;</span></span>_

This uses the <a href="https://www.nuget.org/packages/HtmlAgilityPack" target="_blank">HtmlAgility</a> for parsing.

And the output would look something like this.

![][1]

&nbsp;

&nbsp;

 [1]: https://camo.githubusercontent.com/03204325907aae77bb2f865117271bf28136d5ca/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4356766c6b5878564141456d7454332e706e67