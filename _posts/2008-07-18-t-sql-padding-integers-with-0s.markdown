---
layout: post
title:  "T-SQL - Padding Integers with 0's"
date:   2008-07-18 00:00:00 -0500
tags: t-sql
---

Here is a quick and easy tip for displaying integers padded with 0’s.  This can be very useful when you need to display numbers that have inconsistant string lengths all as the same length.  For example, you may have a list of employee numbers that can very in string length (number of digits) from 1 to 5 but need to be printed as 5 character strings padded with 0’s in the front (see the table below).

| EmpNo (int) |
| --- |
| 17 |
| 972 | 
| 89 |
| 18834 |
| |

The employee numbers in the above table need to be displayed as follows:

| 00017 |
| 00972 | 
| 00089 |
| 18834 |  
| | 

I’ve seen some pretty nasty approaches to doing this including conditional statements testing the length of the integer’s string length and then appending a string of 0’s to the front of it.

Assume `@empNo` is a string (nvarchar) that was cast from an integer value.

{% highlight sql %}
if len(@empNo) = 2
    set @empNo = '000' + @empNo
{% endhighlight %}

Now, this might seem simple enough but imagine if you were working with numbers that could range from 1 to 30 characters in string length… Lots of conditional statements to write out.  Yes, there are some ways of simplfying it a little using loops but there’s still an easier way.

Assume again that @empNo is a string that has already been cast from an integer value.

{% highlight sql %}
set @empNo = right('000000000000' + @empNo, 5)
{% endhighlight %}

It’s that easy.  Just append a bunch of 0’s to the string version of your integer and use the `right()` function to trim all but the 5 (or how every many characters you need to display) right most characters of the resulting string.