--- 
layout: post 
title: "Performance testing with fabric, jenkins, and jMeter" 
date: 2015-07-02 13:15:00 
categories: jmeter 
tags: [performance, jMeter, ci]
comments: true 
---
## tl;dr
I had a suite of jMeter tests that needed to be automated as part of a continuous integration setup, I used fabric 
and jenkins to achieve this. 
<!--more-->

{% capture imgURL %}{{ site.baseurl }}{{ site.url }}/img/{{ page.url | | replace:'/','-' | remove_first: '-'}}/{% endcapture %}
{% capture thumbURL %}{{imgURL}}thumbs/{% endcapture %}

## Rationalle
Different websites to performance test automatically, running from the 
same third party application. Along with similar tests to run for each website, 
and site specific tests.

## Directory structure

Before explaining how it's all setup, this is the directory structure 
used for the tests 
{% highlight asm %} 
-common
 --view_page.jmx
 --login
   --user_login.jmx 
-website_one
 --checkout.jmx
 --view_page.csv 
-website_two
 --view_page.csv 
-reports 
{% endhighlight %}

## Common tests

As you may have geussed, view_page.jmx and view_page.csv are releated. 

The .jmx is a jmeter test that relies on the csv to run the tests. The 
csv contains a URL to test, and a value to check on the page. 

[![View CSV Settings in jMeter]({{"csvSetting1.jpg" | prepend: thumbURL}} "View CSV Settings in jMeter")]({{"csvSetting1.png" | prepend: imgURL}}){: data-lightbox="csv"}{: data-title="Setting up CSV in jMeter"}

[![View CSV Settings in jMeter]({{"csvSetting2.jpg" | prepend: thumbURL}} "View CSV Settings in jMeter")]({{"csvSetting2.png" | prepend: imgURL}}){: data-lightbox="csv"}{: data-title="Setting up CSV in jMeter"}

Notice the variable names, url_key,text_to_test

These are the values that jMeter expects to find in the csv for example
{% highlight asm %} 
/about, about our site
{% endhighlight %} 

[View the jMeter test for this](https://gist.githubusercontent.com/grahamrgriffiths/91db8bb3cc9c244346a0/raw/d8cb9438dce79cc8beb5c95b186fddae25cfa358/view_page.jmx){:target="_blank"}

[user_login.jmx](https://gist.githubusercontent.com/grahamrgriffiths/b4197d1aa8c5548d4ab4/raw/f2de0bf9d57b10558171c17de082403b72c40287/user_login.jmx){:target="_blank"} is used for testing the account sections of our sites, it's a test that carries out a post request to the login form of a web application. 

In the fabric script, I set it up so that any test can use a csv data set. 
The csv file just needs to exist in the correct website test folder, and be named after the test file.

## Jenkins build
The build takes three main parameters: users, loops, and the ramp up 
period. It uses the git module to pull the latest code from a hosted 
repository, then executes a fabric script using the shell command. 

[![View Parameterised Build Example]({{"jenkinsParametersjMeter.jpg" | prepend: thumbURL}} "View Parameterised Build Example")]({{"jenkinsParametersjMeter.png" | prepend: imgURL}}){: data-lightbox="jenkinsParametersjMeter"}{: data-title="Example of setting parameters for a jenkins build"}

[![View Git Configuration Example]({{"jenkinsGit.jpg" | prepend: thumbURL}} "View Parameterised Build Example")]({{"jenkinsGit.png" | prepend: imgURL}}){: data-lightbox="jenkinsGit"}{: data-title="Example of git configuration for a jenkins build"}

[![View Shell Execution Example]({{"jenkinsShell.jpg" | prepend: thumbURL}} "View Parameterised Build Example")]({{"jenkinsShell.png" | prepend: imgURL}}){: data-lightbox="jenkinsShell"}{: data-title="Example of executing a shell script for a jenkins build"}

The  jMeter plugin we use provides some basic reporting graphs which are useful.

[![View Response Graph Example]({{"responseTimeGraph.jpg" | prepend: thumbURL}} "View Parameterised Build Example")]({{"responseTimeGraph.png" | prepend: imgURL}}){: data-lightbox="responseTimeGraph"}{: data-title="Example of response time graph from jenkins jMeter plugin"}

This show's me how response time of a widely used page has improved per build, the jenkins builds detail the changes too.

## The fabric script
The essence of this script is to run jmx files found in the directories 
listed above. I use the command line arguments to setup parameters for 
the tests. 

{% gist grahamrgriffiths/2e873ac76687acabcd1c %}

## Inactive tests
In the scenario where a test needs to be excluded from the automated running process, append .inactive to the file name and it will be ignored.
