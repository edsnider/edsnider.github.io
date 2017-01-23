---
layout: post
title:  "Darken Shading on PWA My Tasks with jQuery"
date:   2010-05-23 00:00:00 -0500
tags: pwa jquery
---

The My Tasks page in PWA (Project Web Access) has a
Timephased View that shows users their task assignments by week with planned and
actual hours per day.  To make it easier on the user to quickly find the tasks
by day where they need to enter hours worked the control “grays out” the tasks
on the days where 0 hours were planned leaving the days that need hours entered
white.  The problem is the grayed out tasks are not very dark, in fact you have
to look at your screen from the side to notice the difference between the grayed
out cells and the white cells.  To make the grayed out cells more obvious for
the user you can darken them using the following line of jQuery
code:

{% highlight javascript %}
$("td[title='Planned 0h']").css("background", "#D9D9D9");
{% endhighlight %}

The code uses a selector to grab all of the table
cells with the title attribute set to ‘Planned 0h’ (which are the “grayed out”
cells) and sets their background to a specific color using the css method.