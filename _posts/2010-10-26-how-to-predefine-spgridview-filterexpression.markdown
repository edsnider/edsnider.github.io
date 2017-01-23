---
layout: post
title:  "How to Predefine SPGridView FilterExpression"
date:   2010-10-26 00:00:00 -0500
tags: sharepoint
---

There are many situations where you might need to predefine or set a default FilterExpression for a SPGridView control.  This would be a situation where you want to filter the rows within a SPGridView prior to the user selecting any filter parameters from the column headers.  I find this particularly useful when initial grid filtering should be based on a selection made from a connected web part such as in a drilldown scenario.

The filter settings including data fields to be filtered and the format of the FilterExpression that are applied to the SPGridView in the CreateChildControls method aren’t actually applied to the data source until the user actually selects a filter from the column header.  This means you can’t temporarily modify the FilteredDataSourcePropertyFormat property on the grid view prior to the initial DataBind call because the FilterExpression won’t be used by the data source at this point.  So, in order to have the data source perform a filter you need to set the FilterExpression on the data source object itself prior to DataBind call in the OnPreRender method.   You’ll see what I mean in the code below.

So by doing this you now have the SPGridView filtered by default without the user selecting the filter from the column header.  But now that you’ve defined the FilterExpression on the data source it will override any user selected filters since they applied prior to the OnPreRender call.  The way around this is to simply check if the read only fields FilterFieldName and FilterFieldValue are defined.  If they are then the user has specified a filter and you want the data source to use it as opposed to the predefined expression.

Here is some sample code to illustrate my point (assume the two string variables represent values defined by another web part or source external to the grid view):

{% highlight csharp %}
private SPGridView _spgv;
private ObjectDataSource _ods;
 
private string defaultFilterColumn;
private string defaultFilterValue;
 
protected override void OnPreRender(EventArgs e)
{
    base.OnPreRender(e);
 
    // ...
    // code to create grid view columns would be here...
    // ...
 
    if (string.IsNullOrEmpty(_spgv.FilterFieldName) && 
        string.IsNullOrEmpty(_spgv.FilterFieldValue))
    {
        _ods.FilterExpression = string.Format(_spgv.FilteredDataSourcePropertyFormat, defaultFilterValue, defaultFilterColumn);
    }
 
    _spgv.DataBind();
}
 
protected override void CreateChildControls()
{
    base.CreateChildControls();
 
    // ... 
    // code to define and setup ObjectDataSource
    // ...
 
    _spgv = new SPGridView();
    _spgv.AllowFiltering = true;
    _spgv.FilterDataFields = "ColumnOne,,ColumnThree,";
    _spgv.FilteredDataSourcePropertyName = "FilterExpression";
    _spgv.FilteredDataSourcePropertyFormat = "{1} = '{0}'";
 
    this.Controls.Add(_spgv);
}
{% endhighlight %}

This configuration will let you to apply a filter to the SPGridView data on load and then allow users to change it by specifying their filter criteria from the column header filter menus within the SPGridView control.

#### Some notes:

* If the user selects “Clear Filter from …” on the filter menu the SPGridView will return to the state at which it was filtered using the predefined FilterExpression.  This may not be an issue – just depends on the use case really.  If you want the Clear Filter command to completely clear the filter, including the predefined filter, then I would recommend setting a boolean variable with some logic to ignore the default filter information and set the data source FilterExpression to empty.  You can catch the Clear Filter click in the LoadViewState method some like below: 

{% highlight csharp %}
protected sealed override void LoadViewState(object savedState)
{
    base.LoadViewState(savedState);
 
        if (Context.Request.Form["__EVENTARGUMENT"] != null &&
            Context.Request.Form["__EVENTARGUMENT"].EndsWith("__ClearFilter__"))
    {
        _ignoreDefaultFilter = true;
    }
}
{% endhighlight %}

* Also, because you are setting the initial filter directly on the data source and not via the SPGridView filter properties the column header filter menu will not reflect the predefined filter being applied (only the user selected filter).  There is probably a simple way around this, I just haven’t messed with it yet.
