--- 
layout: post 
title: "Handling Multiple Environments With Fabric" 
date: 2015-08-22 10:30:00 
categories: ci
tags: [ci]
comments: true 
---

## tl;dr
Fabric is a python library for streamlining SSH based application deployment, this post details configuring multi-environment deployments.
<!--more-->

{% capture imgURL %}{{ site.baseurl }}{{ site.url }}/img/{{ page.url | | replace:'/','-' | remove_first: '-'}}/{% endcapture %}
{% capture thumbURL %}{{imgURL}}thumbs/{% endcapture %}

## Rationale
Deploy scripts are extremely useful once they are in place, but handling differences between development, staging, and production environments can be tricky.

For example, the staging database should be different from the live database. If you’re deployment involves database changes, the correct connection settings need to be in place.

Another example is variables that determine actions during the build, such as a database reload. 
* This is a useful step in staging and development, for keeping test data up to date. It shouldn’t be used on live.

## Approach
Three environment based config files, development.py, staging.py and production.py

A common.py config file that contains a default environment varaible values. These don’t have to be defined (overwritten) in the separate environment config files, but can be.

fabfile.py, database.py, maintenance.py (and others) that handle the deployment

## Loading the appropriate configs

For clarity and maintainability, we define the method in a separate file as below

{% gist 7ae226c03db5a87945e7 %}

Fabfile.py loads that library. The file name and function name can be different here, and don’t need to match - this is just an example.

{% highlight py %} 
from loadEnfConfig import loadEnvConfig
{% endhighlight %}

Fabfile.py also contains a build method, which can be invoked by Jenkins Shell or similar.

[![Shell Execution in jenkins]({{"ENV-jenkinsShell.png" | prepend: thumbURL}} "Execute Shell in Jenkins")]({{"ENV-jenkinsShell.png" | prepend: imgURL}}){: data-lightbox="shell"}{: data-title="Executing a fabric command via shell in jenkins"}

In this example, three methods are called in turn pre_build, build, and deploy

pre_build calls the config load method for the environment

{% highlight py %}
@task
def pre_build(envname='staging'):
 
    # Load the env config before we try to do anything.
    loadEnvConfig(envname)
{% endhighlight %}
    
## Where to go from here

You should be able to reference the variables defined in the config files throughout the build.

For example, 

{% highlight py %}
if (env.skip_database_reload == 0):
	importDatabase(envname) # this does backupDatabase() and loadDatabase()
{% endhighlight %}

You will also be able to write one deploy script for multiple environments.
