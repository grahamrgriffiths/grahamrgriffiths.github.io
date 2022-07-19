--- 
layout: post 
title: "Handling viewstate with jMeter" 
date: 2015-07-15 13:15:00 
categories: jmeter 
tags: [performance, jMeter]
comments: true 
---
## tl;dr
Writing a jMeter test capable of performance testing a C# / ASP Web Forms application with unpredictable viewstate fields.
<!--more-->

{% capture imgURL %}{{ site.baseurl }}{{ site.url }}/img/{{ page.url | replace:'/','-' | remove_first: '-'}}/{% endcapture %}
{% capture thumbURL %}{{imgURL}}thumbs/{% endcapture %}

## Rationale
A question delivery system that uses C# / ASP web forms, a post back to mark the answer, and a form request to progress to the next question.

If you’ve dealt with web forms, you’ll know that post backs result in view state.

Handling view state on it’s own is easy - just use a regular expression extractor with the following expression.

{% highlight asm %} 
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="([^'"]+)"
{% endhighlight %}

The problem is, we don't know how many hidden viewstates and corresponding values there will be

## The essence of the test
- Log in as user
- Start a new assessment
- Answer the questions
- Confirm end of assessment

The number of questions in a session can change, I use a reusable test fragment, the while controller, and a variable to determine if there are still questions to answer.

Step three is the section that needs to handle view state.

Using fiddler, this is a cut down example of the post request made:

| Field Name  |  Field Value  |
|-------------|---------------|
|__VIEWSTATEFIELDCOUNT | 4 |
|__VIEWSTATE   | /wEPDwULLTE3MDQxODY1MTUPZBYCZg |
|__VIEWSTATE1  | cmVmPSIvSW5zdGl0dXRpb25zL0FjY291bn9 |
|__VIEWSTATE2 | bGFzcz0ibmF2SXRlbUluYWN0aXZlIiBocm |
|__VIEWSTATE3 | bnN3ZXJJRD5rX19CYWNraW5nRmllbGQbP |

Notice the viewstate fields, there are many and each have a unique value.

The number of fields also changes per request, depending on each question.

*How can we dynamically submit these fields through jMeter?*

I tried various solutions, but eventually settled on opting for using a regular expression extractor, and a Beanshell script. 

The regular expression extractor:

Reference name: viewstates
{% highlight asm %} 
<input type="hidden" name="__VIEWSTATE([\d]*)" id="__VIEWSTATE([\d]*)" value="([^'"]+)"
Template: $1$2$3
Match No. : -1
{% endhighlight %}

The expression is similar to the one we had above for the single viewstate field, notice how it matches on the viewstateNumber and the value. The template lines up the matched value, and the match number value specifies that we’re after all that are available

In essence, this pulls all of the values into one variable, viewstates.

In our test, we have a http request that submits the question with an answer selected. With this information, we also need to send the viewstates = this is where the beanshell script comes in.

The following script is placed in a beanshell pre-processor on the http request:

{% gist grahamrgriffiths/2a50f65f07249b2d4ea9 %}
It loops through all of the matches from our regular expression, and adds a header for each one to the request. The header name is straightforward, the value however needs to be encoded.

The encoding prevents invalid view state exceptions being thrown. Most form parameters should be encoded anyway.

Hopefully the comments explain what’s going on, and I’ve referenced the sources for how I learned to do this with the regular expression extractor, and how to encode URL values.

That's surpisingly all you need, enjoy!
