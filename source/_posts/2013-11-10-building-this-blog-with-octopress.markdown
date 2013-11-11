---
layout: post
title: "Building this Blog with Octopress"
date: 2013-11-10 15:14
comments: true
categories: 
---

I just setup this blog using [Octopress](http://octopress.org/) as the page generation engine and [Github Pages](http://pages.github.com/) to **freely** host the content. In total, it took me about four hours to get from start to first blog post, and that was mainly because my ruby environment on my dev machine was fairly messed up from mucking around six months ago. 

This initial post is both a how-to for setting up a blog with Octopress/Github and quick cheat-sheet so I don't forget how I did things. 

## Setup
The [Octopress setup docs](http://octopress.org/docs/setup/) are incredibly helpful, so I'm not going to duplicate content explaining what everything means. Assuming ruby is correctly installed the setup commands are as follows:

```
## 1. Create personal github repo named "username.github.io", mine is "paulcichonski.github.io"
## 2. Clone Octopress repo locally and setup directory structure
git clone git://github.com/imathis/octopress.git octopress
cd octopress
rake install
## 3. connect your local install with your git repo (see Octopress docs for more details: http://octopress.org/docs/deploying/github/)
rake setup_github_pages
```
The above commands will leave your '/octopress' dir in a state where you are ready to begin blogging. You can think of the '/octopress' dir as a container for all your blog content as well as the library for all the commands you need to push content to Github.

## Important Octopress Files and Directories

The following files and directories comprise the essential building blocks for an Octopress site:

* **/octopress/\_config.yml**: Holds all the configuration for the site, Octopress uses this when generating your static HTML pages from markdown.
* **/octopress/source/** - This is the directory that includes all the content you actually edit, it syncs with the 'source' branch of your github repository. Remember to always commit any changes made in this directory to your 'source' branch. Running "rake generate" from the '/octopress' dir level will take the files in 'source', parse them in combination with your '_config.yml' file and generate static html content into the '\_deploy' dir.
* **/octopress/\_deploy** - This is the directory that Octopress will dynamically generate, it contains your actual website static HTML that people see. When you run "rake deploy" Octopress handles pushing all content from '\_deploy' to the 'master' branch of your github repo (i.e., the branch responsible for serving content).

## Important Octopress Commands

The following commands are the most useful for doing basic things with Octopress

```
## generates static html by parsing markdown in 'source' directory, using '\_config.yml' for configuration parameters
rake generate
## deploys all content in '\_deploy' to the 'master' branch on your github repo 
rake deploy 
## binds a webserver to localhost:4000 that serves pages from '\_deploy'; watches for changes in 'source' and auto-generates all changes into '\_deploy'. Use this for local development.
rake preview 
```

That is it for now, still need to document how to perform development from multiple machines and how to rebuild your local development workstation if something goes wrong.
