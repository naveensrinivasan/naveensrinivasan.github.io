---
title: "Stunning Tribble"
date: 2021-09-27T00:37:38Z
tags:
  - supplychain
---

## Defending from including OSV/CVE in go dependencies

tldr: If you want to avoid including OSV/CVE in your `go.mod/go.sum` you can utilize this https://github.com/naveensrinivasan/stunning-tribble tool do that. Here is an example of how this is being used in scorecard https://github.com/ossf/scorecard/blob/9df865c4f83cfb36bae487125e5ccbc6aef448c6/Makefile#L64-L72

I contribute to a project that is primarily focused on supply chain security https://github.com/ossf/scorecard which is part of Open Source Security Foundation, which is also part of Linux Foundation.

We happened to realize that between version 1.2.0 and 2.0.0 we have included a library that happens to have OSV(https://osv.dev) https://deps.dev/go/github.com%2Fossf%2Fscorecard/v1.2.0/comparev2=v2.0.0%2Bincompatible. This was accidental, and we wanted to check how to avoid this in the future. 

The goal was to make this process part of every PR to check to be aware of any new OSV that we could be introducing. We also wanted an option to ignore any pre-existing OSV until we fix them.

So this led to https://github.com/naveensrinivasan/stunning-tribble.  Stunning-tribble takes input as stream and looks for OSV from osv.dev.
The stream is based on the format `go list -m -f '{{if not (or .Main)}}{{.Path}}@{{.Version}}_{{.Replace}}{{end}}' all`. This will provide all the dependencies, including the replace directives. 

The tool is specifically built with the idea of not having any external dependency to avoid it having any OSV.

To ignore existing OSV pass them as CSV `stunning-tribble GO-2020-0018,GO-2020-0016` 

Supply chain security is one of the new ways in which lots of attacks are happening https://github.com/cncf/tag-security/tree/main/supply-chain-security/compromises




