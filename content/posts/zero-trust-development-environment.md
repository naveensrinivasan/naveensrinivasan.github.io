---
title: "Zero Trust Development Environment"
date: 2021-09-30T23:48:46Z
images:
---
![](https://i.imgur.com/eHwmG9V.jpg)
*[Photo by novia wu on Unsplash](https://unsplash.com/photos/JyepCoDqics?utm_source=unsplash&utm_medium=referral&utm_content=creditShareLink)*

I am paranoid about running unknown code on my machine. I have been using a MacBook for some years now, but the way I used to install any software was `brew` like most developers.

Later I asked the question, "How do I trust my `brew installs`?" But like most of us, I had to try new packages and deploy software to do my work.

I contribute to many different [OSS projects](https://github.com/naveensrinivasan), and I also wanted to keep the environments separate. For example, one of the OSS projects requires `go 1.15` whereas the other one needs `go 1.17`.

The question I get asked is, aren't you overblowing the situation? No, I am not, and here are few examples of supply chain issues https://github.com/cncf/tag-security/tree/main/supply-chain-security/compromises and a great article by Paulo Gomes, [Golang: stop trusting your dependencies!](https://itnext.io/golang-stop-trusting-your-dependencies-a4c916533b04)

Here are the things that I wanted for my new ENV
1. I wanted an automatable environment.
1. It has to be on a Linux box. 
1. It would be nice to have an immutable environment.
1. It should be easy to maintain, and it would be great to have a community.
1. I was ready to pay not more than $20 for my ENV per month. I meant for the hardware on a cloud instance. In my opinion, the cost of security is worth it. 

With the above requirements, I ran into NixOS and nixpkgs. 
NixOS has a high learning curve like anything else, but I think it is worth the time spent on learning it. 

Now my environment is still a Macbook with a terminal. I only install packages from the App Store or signed packages, which has reduced my attack vector. 

I have different shell ENVs with nixpkgs for various projects:
* https://github.com/naveensrinivasan/dotvim/blob/master/lnd.nix
* https://github.com/naveensrinivasan/dotvim/blob/master/blog.nix
* https://github.com/naveensrinivasan/dotvim/blob/master/shell.nix

I have tmux sessions for each ENV, which are still running on a single cloud instance VM. I can update packages and don't have to worry about messing with the ENV's or compromising my machine security. 

It would be best if you were comfortable using CLI-based ENV. I have been using vim for a while now, and I don't miss my UI for writing code. 
