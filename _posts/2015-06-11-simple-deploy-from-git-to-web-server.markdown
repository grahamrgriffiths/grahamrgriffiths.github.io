---
layout: post
title:  "Simple deploy from git on a web server that has ssh access"
date:   2015-06-11 13:58:09
categories: "continuous integration"
tags: ci, git
comments: true
---
## tl;dr
A friend of mine asked me for help with deploying a git repo to their web server, 
this post explains my solution.
<!--more-->

## Rationalle
I didn’t want to spend too much time playing with services like [https://codeship.com/](https://codeship.com/){:target="_blank"} 
As they had ssh access to the server, and git installed - I figured a script would be the easiest way to go.

## Approach
I’ve used python fabric for ci deploys before, but that seemed a tad excessive so I decided to use a bash script.

First, authenticate git by following the instructions here:
[https://help.github.com/articles/generating-ssh-keys/](https://help.github.com/articles/generating-ssh-keys/){:target="_blank"} 

Then make a note of directories

Git repo structure:
{% highlight asm %} 
Stuff
other stuff
public_html
random file
readme.md
{% endhighlight %} 

Location of existing web root:
~/public_html

## Aim of the script
* Clone git repo to folder
* create a symbolic link to the public_html directory in the git repo from the web root

I figured version control would be useful too – every time we clone, we want to it to a timestamp directory. We could use git releases here and checkout the corresponding tags, but that was out of scope for this task.

## Problems encountered
Bash didn’t know what the command ‘git’ was. This is fixed by adding to the path variable. Which git or type –p git should give you the location.
{% highlight bash %} 
PATH=$PATH:/usr/bin/
{% endhighlight %} 

The authentication to git was reset, so I added
{% highlight bash %} 
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/git_rsa
{% endhighlight %} 

## A note on permissions
Web servers should have 755 on folders and 644 on directories, this is easily fixed with chmod and find.

{% highlight bash %} 
#Find and change recursively on directories
find $deployFolder -type d -exec chmod 755 {} +
{% endhighlight %} 

{% highlight bash %} 
#Find and change recursively on files
find $deployFolder -type f -exec chmod 644 {} +
{% endhighlight %} 

## A note on symbolic links
If you're hosting provider doesn't like symbolic links, then use the following instead

{% highlight bash %} 
rm -r $webRoot/*
cp -R $deployFolder/public_html $webRoot/.
{% endhighlight %} 

## The Resulting Script
{% gist grahamrgriffiths/61552cffa2f88a749236 %}

We’re all set to run
{% highlight bash %} 
sh deploy.sh
{% endhighlight %} 

To rollback:
{% highlight bash %} 
ln -sfn webDeploys/WORKINGTIMESTAMP/public_html/ /home/user/public_html 
{% endhighlight %} 

Done - in about 10 minutes too.
