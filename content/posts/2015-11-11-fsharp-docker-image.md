---
title: fsharp docker image
author: admin
type: post
date: 2015-11-11T02:41:14+00:00
url: /?p=2481
categories:
  - 'F#'
tags:
  - docker
  - fsharp

---
The fsharp project has a official docker image <a href="https://github.com/fsprojects/docker-fsharp" target="_blank">https://github.com/fsprojects/docker-fsharp</a> . The one issue with that is it is based on mono 4.0.4 which is buggy and fsharp does not work very well. The latest alpha release of the mono with which fsharp works well is 4.2.0. The 4.2.0 isn&#8217;t available in stable channels.

So I created a docker image with the latest mono from their alpha repo and using the latest fsharp from the github.

<a href="https://github.com/naveensrinivasan/fsharp-docker" target="_blank">https://github.com/naveensrinivasan/fsharp-docker</a>

[github file = &#8220;/naveensrinivasan/fsharp-docker/blob/master/Dockerfile&#8221;]

It is also available in the docker hub. You could get it by

docker pull naveensrinivasan/fsharp