---
layout: post
title:  "Fail - Saab Distance-to-Empty Indicator"
date:   2010-05-14 00:00:00 -0500
tags: C#
---

Looks like when Saab was developing the Distance-to-Empty logic for the SID (Saab Information Display) they didn’t think they needed to provide a conditional branch for DTE specifically equal to 1…

{% include image.html file="/assets/postimages/saab_sid_1miles.jpg" caption="" width="300" alt="Saab SID 1 Miles" %}

I guess they didn’t count on anyone actually letting their tank get that low. Well I did manage to get mine that low and, being the nerd that I am, when I saw “1 miles” on my dash the first thing that came to my mind was:

{% highlight csharp %}
string dte = miles == 1 ? "1 mile" : miles.ToString() + " miles";
{% endhighlight %}