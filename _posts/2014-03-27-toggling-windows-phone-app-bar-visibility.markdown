---
layout: post
title:  "Toggling Windows Phone App Bar Visibility"
date:   2014-03-27 00:00:00 -0500
tags: windows windowsphone
---

Quick note on changing the visibility of an app bar in a Windows Phone app.

Toggling the visibility of a page’s app bar from code-behind is slightly different than most other controls.  Unlike other UI elements, the visibility of the app bar on a page should note be set using the app bar’s name – this could result in null reference exceptions when the app bar’s visibility is set to false.  Instead (and actually a bit more straightforward) the app bar’s visibility should be set using the page’s `ApplicationBar` property:

{% highlight csharp %}
ApplicationBar.IsVisible = false;
{% endhighlight %}