---
layout: post
title:  "Reusable EditorPart - Using an EditorPart on Multiple Web Parts"
date:   2010-09-10 00:00:00 -0500
tags: C# sharepoint sharepoint2010
---

Typically when writing a custom EditorPart you make it specific to a particular web part.  What makes an EditorPart specific to a single web part is the reference back to the web part in the ApplyChanges and SyncChanges methods. You need this reference in order to access the properties on the web part that the EditorPart is collecting values for.

Now, suppose you have multiple web parts that have the same properties – it would be nice to be able to write a single EditorPart to be used by all of the web parts wouldn’t it?  There are a few approaches that can be taken here to accomplish this:

1.  You could pass the web part type into the EditorPart when instantiating the EditorPart when adding it to the web part’s EditorPartCollection in the IWebEditable.CreateEditorParts method.  Then in the EditorPart’s ApplyChanges and SyncChanges methods you can use a switch statement or something to figure out what to cast the WebPartToEdit to in order to access the web part properties
– obviously this is not an ideal solution as it involves changing the EditorPart code every time you hook it to another web part.  You also have to rely on the custom web part to be developed with the correct properties in order for the EditorPart to work as there is nothing enforcing that to happen.

2.  Or, you could create a base web part class that inherits from WebPart and IWebEditable and then have that base web part class contain the properties needed by your EditorPart.  You can put the IWebEditable.CreateEditorParts method in this base web part class, too.  Now, instead of your EditorPart having to reference the actual web part to access the properties it can just reference
the base web part class and then of course any web parts that need to use that EditorPart will need to inherit from this base web part class instead of directly from WebPart.  This actually works nicely until you have a second base web part class for using a different EditorPart… Since you can’t inherit from more than one base class your web part won’t be able to use more than one  EditorPart.  You could incorporate multiple EditorParts into a single base web part class… but what if not all of those EditorParts are needed by each of the web parts inheriting from the base class?  If your solution is only going to have one EditorPart or no web parts will have multiple reusable EditorParts then this might be a good solution for you.  The thing this option has over the
previous one is the web part properties are guaranteed to be there when the EditorPart goes to access them since the web part is inheriting them from the base web part class – which is how the EditorPart is getting to the properties.

3.  (Best approach) Or, you could do something similar to the base web part idea but use interfaces instead so your web part can inherit multiple. In order to do this you simply create an interface for each group of web part properties that are handled by an EditorPart and have your web part implement those interfaces.  Also have your web part implement IWebEditable.  In the IWebEditable.CreateEditorParts method of the web part class add each of the EditorParts you need to the collection.  Have the EditorParts reference the interfaces as opposed to the web part class in order to access the properties.
This now mean any web part that implement one of those interfaces will have the required properties and the EditorPart will be able to be used on multiple web parts since it’s using the interfaces.  In my opinion this is the way to go.

Here is some sample code to show you what I mean:

{% highlight csharp linenos %}
// Interface - contains properties that can be used by
// any web part that displays stock symbols (Ticker, Table, List, etc.)
public interface IStockWebPart
{
    List SymbolsToShow { get; set; }
}
 
// Interface - contains common properties that can be used by
// any web part that displays a ticker
public interface ITickerWebPart
{
    int TickerWidth { get; set; }
    string TickerTitle { get; set; }
}
 
// EditorPart - This EditorPart allows users to set the properties of a ticker user control
//  in any web part that implements the ITickerWebPart interface.
public class TickerEditorPart : EditorPart
{
    private TextBox _txt_tickerWidth;
    private TextBox _txt_tickerTitle;
 
    public override bool ApplyChanges()
    {
        EnsureChildControls();
 
        ITickerWebPart webPart = (ITickerWebPart)this.WebPartToEdit;
 
        if (webPart == null)
            return false;
 
        webPart.TickerWidth = Int32.Parse(this._txt_tickerWidth.Text);
        webPart.TickerTitle = this._txt_tickerTitle.Text;
 
        return true;
    }
 
    public override void SyncChanges()
    {
        EnsureChildControls();
 
        ITickerWebPart webPart = (ITickerWebPart)this.WebPartToEdit;
 
        if (webPart == null)
            return;
 
        this._txt_tickerTitle.Text = webPart.TickerTitle;
        this._txt_tickerWidth.Text = webPart.TickerWidth.ToString();
    }
 
    public override void CreateChildControls()
    {
        // ... add the text boxes here ...
    }
}
 
// EditorPart - This EditorPart allows users to set the properties of any web part that implements
//  the IStockWebPart interface.
public class SymbolPickerEditorPart : EditorPart
{
    // ... implemented same as EditorPart above ...
}
 
// WebPart - This web part has a scrolling ticker user control and has common ticker properties from
//  the ITickerWebPart interface.  It also has a property from the IStockWebPart interface that
//  specifies a list of stock symbols.
public class ScrollingTickerWebPart : WebPart, IWebEditable, ITickerWebPart, IStockWebPart
{
    #region ITickerWebPart Members
    [WebBrowsable(false)]
    [Personalizable(PersonalizationScope.Shared)]
    public string TickerTitle { get; set; }
 
    [WebBrowsable(false)]
    [Personalizable(PersonalizationScope.Shared)]
    public int TickerWidth { get; set; }
    #endregion
 
    #region IStockWebPart Members
    [WebBrowsable(false)]
    [Personalizable(PersonalizationScope.Shared)]
    public List StockSymbols { get; set; }
    #endregion
 
    public ScrollingTickerWebPart()
    {
        this.ExportMode = WebPartExportMode.All;
    }
 
    public override void CreateChildControls()
    {
        // ... ...
    }
 
    #region IWebEditable Members
    EditorPartCollection IWebEditable.CreateEditorParts()
    {
        List editors = new List();
 
        TickerEditorPart tickerEditorPart = new TickerEditorPart();
        tickerEditorPart.ID = this.ID + "_tickerEditorPart1";
 
        SymbolPickerEditorPart symbolPickerEditorPart = new SymbolPickerEditorPart();
        symbolPickerEditorPart.ID = this.ID + "_symbolPickerEditorPart1";
 
        editors.Add(tickerEditorPart);
        editors.Add(symbolPickerEditorPart);
 
        return new EditorPartCollection(editors);
    }
 
    object IWebEditable.WebBrowsableObject
    {
        get { return this; }
    }
    #endregion
}
 
// WebPart - This web part displays a list of stock symbols.  It has a property from the IStockWebPart
//  interface that specifies a list of stock symbols.
public class StockSymbolsListWebPart : WebPart, IWebEditable, IStockWebPart
{
    #region IStockWebPart Members
    [WebBrowsable(false)]
    [Personalizable(PersonalizationScope.Shared)]
    public List StockSymbols { get; set; }
    #endregion
 
    public StockSymbolsListWebPart()
    {
        this.ExportMode = WebPartExportMode.All;
    }
 
    public override void CreateChildControls()
    {
        // ... ...
    }
 
    #region IWebEditable Members
    EditorPartCollection IWebEditable.CreateEditorParts()
    {
        List editors = new List();
 
        SymbolPickerEditorPart symbolPickerEditorPart = new SymbolPickerEditorPart();
        symbolPickerEditorPart.ID = this.ID + "_symbolPickerEditorPart1";
 
        editors.Add(symbolPickerEditorPart);
 
        return new EditorPartCollection(editors);
    }
 
    object IWebEditable.WebBrowsableObject
    {
        get { return this; }
    }
    #endregion
}
{% endhighlight %}