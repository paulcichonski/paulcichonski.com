---
layout: post
title: "Setting Up Octopress on a new Machine"
date: 2013-11-10 20:46
comments: true
categories: octopress blogging
published: true
description: You need to run octopress on multiple machines? This post explains how.
---
As soon as I got this blog up and running, the first thing I did was to completely screw up my [Octopress](http://octopress.org/) install (apparently I did need all those files I deleted), thus rendering the entire site useless. As with anything "tinkering" related, this is fairly normal so it is also fairly normal to rebuild from scratch. Luckily, smart people have [already figured out](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/) how to do this.

The following steps outline how to setup a fresh Octopress install that connects to an existing [Github Pages](http://pages.github.com/) repository (this builds on the zerosharp post with some updates based on recent changes in Octopress). **These steps assume that the Github repository is fully up to date with all latest changes**

First you need to make sure that your `source` directory actually contains the `source/_posts` and the `stylesheets` folder. You also need to make sure .gitignore is not ignoring any of these (if there is a reason these should not be committed please let me know, I could not think of any).

```
## This needs to happen on your first machine, for some reason they are ignored by default.
cd source/
git add _posts/ 
git add stylesheets
git commit -m "adding posts and stylesheets dir"
git push origin source
```

The remainder of the steps happen on your second machine. First clone the `source` branch

```
git clone -b source git@github.com:username/username.github.io.git octopress
cd octopress
```

Next, install Octopress

```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
rake setup_github_pages
```

The last command deleted the `_deploy` directory and re-added it. We don't want this because we want the latest changes from `_deploy` so we don't run into any nasty ` [rejected]        master -> master (non-fast-forward)` git errors because of an out-of-date branch.

```
rm -rf _deploy
git clone git@github.com:username/username.github.io.git _deploy
```


Octopress should now be setup, and the `source` dir should contain your up-to-date markdown. To test things out make a change to a post (or make a new post) then regenerate and deploy.

```
rake generate
rake deploy
```

Your new changes should appear on your site. The only downside of this approach is that when you go back to your original machine and try to deploy, git will yell at you (i.e., non-fast-forward error) because the `_deploy` dir from that machine is now out of date. The best way to fix this is to remove it (see above) and re-clone it (see above). The other option is to edit the `octopress/RakeFile` and change the line: `system "git push origin #{deploy_branch}"` to `system "git push origin +#{deploy_branch}"` to force the deployment despite version mismatches (**be sure to undo this change immediately after**).
