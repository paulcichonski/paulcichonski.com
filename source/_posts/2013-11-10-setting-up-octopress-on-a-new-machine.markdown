---
layout: post
title: "Setting Up Octopress on a new Machine"
date: 2013-11-10 20:46
comments: true
categories: octopress
---
As soon as I got this blog up and running, the first thing I did was to completely screw up my [Octopress](http://octopress.org/) install (apparently I did need all those files I deleted), thus rendering the entire site useless. As with anything "tinkering" related, this is fairly normal so it is also fairly normal to rebuild from scratch.

The following steps outline how to setup a fresh Octopress install that connects to an existing [Github Pages](http://pages.github.com/)  repository. 

**The following steps assume that the Github repository is fully up to date with all latest changes **

1. Clone Octopress repo: `git clone git://github.com/imathis/octopress.git octopress` and navigate to the directory: `cd octopress`.
2. Setup the required directory structure with: `rake install`
3. Connect the install to your Github repo with: `rake setup_github_pages`