---
layout: post
title:  "Filtering in SPGridView JavaScript Error"
date:   2010-10-21 00:00:00 -0500
tags: sharepoint
---

I created a web part that uses the SPGridView control to display data from a List<T>.  I set AllowFiltering to true and when the web part rendered the gridview the column header appeared as a hyperlink with the dropdown menu arrow for applying a filter (just like the OoB grids do).  But, when I clicked the arrow the filter menu failed to display – instead I received the following javascript error.

>‘null’ is null or not an object – line 5 of spgridview.js.

{% include image.html file="/assets/postimages/spgridview_javascript_error.png" caption="" alt="SPGridView Null or not an object error message dialog" %}

So, I opened up spgridview.js (it’s in LAYOUTS) to see what was happening on line 5.

{% highlight csharp linenos %}
var SPGridView_CallbackContext=null;
function SPGridView_FilterPreMenuOpen(gridViewClientId, templateClientId, menuClientId, dataFieldName, e)
{
    var gridView=document.getElementById(gridViewClientId);
    var callbackEventReference=gridView.getAttribute("callbackEventReference");
{% endhighlight %}

Basically, line 5 attempts to get a reference to the `callbackEventReference` attribute of the SPGridView rendered control (a table).  In order to do this it’s calling document.getElementById on line 4 to first find the SPGridView rendered control by its Client Id.

It’s getting the null reference because the statement in line 4 is not finding the control.  This would be because I forgot to **SET THE SPGRIDVIEW ID PROPERTY** in my web part code.

Without providing the SPGridView with an Id the gridview is rendered to the client without an id which makes it kind of hard for the javascript to get references to it. 🙂