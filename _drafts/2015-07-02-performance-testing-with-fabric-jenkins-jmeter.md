--- 
layout: post 
title: "Performance testing with fabric, jenkins, and jMeter" 
date: 2015-07-02 13:15:00 
categories: jmeter 
tags: [ci, performance, jmeter] 
comments: true 
---

{% capture imgURL %}{{ site.baseurl }}{{ site.url }}/img/{{ site.url }}/{{page.path}}/{% endcapture %}
{% capture thumbURL %}{{imgURL}}/thumbs/{% endcapture %}

## tl;dr
I had a suite of jMeter tests that needed to be automated, I used fabric 
and jenkins to achieve this. 
<!--more-->

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

[Screen Capture] [Links to test and sample CSV] 

user_login.jmx is used for testing the account sections of our sites, it's a test that carries out 
a post request to the login form of a web application. 

[Link to generic login test] 

In the fabric script, I set it up so that any test can use a csv data set. 
The csv file just needs to exist in the correct website test folder, and be named after the test file.

## Jenkins build
The build takes three main parameters: users, loops, and the ramp up 
period. It uses the git module to pull the latest code from a hosted 
repository, then executes a fabric script using the shell command. 

[![View Parameterised Build Example]({{"jenkinsParametersjMeter.png" | prepend: site.thumbURL}} "View Parameterised Build Example")]({{"jenkinsParametersjMeter.png" | prepend: site.imgURL}}){: data-lightbox="jenkinsParametersjMeter"}{: data-title="Example of setting parameters for a jenkisn build"}

[View Parameterised Build Example]({{"/img/2015-07-02-performance-testing-with-fabric-jenkins-jmeter/jenkinsParametersjMeter.png" | prepend: site.baseurl | prepend: site.url }}){: data-lightbox="jenkinsParametersjMeter"}{: data-title="Example of setting parameters for a jenkisn build"}

[View Git Configuration Example]({{"/img/2015-07-02-performance-testing-with-fabric-jenkins-jmeter/jenkinsGit.png" | prepend: site.baseurl | prepend: site.url }}){: data-lightbox="jenkinsParametersjMeter"}{: data-title="Example of git configuration for a jenkisn build"}

[View Shell Execution Example]({{"/img/2015-07-02-performance-testing-with-fabric-jenkins-jmeter/jenkinsShell.png" | prepend: site.baseurl | prepend: site.url }}){: data-lightbox="jenkinsParametersjMeter"}{: data-title="Example of executing a shell script for a jenkisn build"}

The built in jMeter plugin provides some basic reporting graphs which are suprisingly useful.

![Response Graph]({{"/img/2015-07-02-performance-testing-with-fabric-jenkins-jmeter/responseTimeGraph.png" | prepend: site.baseurl | prepend: site.url }})

This show's me how response time of a widely used page has improved per build, the jenkins builds detail the changes too.

## The fabric script
The essence of this script is to run jmx files found in the directories 
listed above. I use the command line arguments to setup parameters for 
the tests. 

{% gist grahamrgriffiths/2e873ac76687acabcd1c %}

## Inactive tests
In the scenario where a test needs to be excluded from the automated running process, append .inactive to the file name and it will be ignored.
