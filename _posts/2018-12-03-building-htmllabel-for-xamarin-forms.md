---
layout: post
title: "Building an HTML Label for Xamarin.Forms"
date: 2018-12-03 00:00:00 -0500
tags: xamarin xamarin.forms
excerpt_separator: <!--more-->
---

I recently built an HTML Label control for Xamarin.Forms. It's available on [nuget](https://www.nuget.org/packages/XamForms.HtmlLabel/) if you'd like to give it a try. I decided to write this post on how and why I built this particular library. As you may or may not know, there are already several other HTML label controls out there and of course there is the `WebView` control. So why did I find it necessary to build my own?

<!--more-->

First of all, I had some very basic requirements (for a side-project) - I needed to do basic text markup like bold/strong, italics, and hyperlinks. Hyperlinks quickly turned into the deal-breaker requirement for most, if not all, of the other libraries out there. Some didn't support hyperlinks at all while others had very complicated code to support hyperlinks (e.g., coordinates and gestures). As for the Xamarin.Forms `WebView` control, it just seemed like overkill for what I needed to do. I feel like the `WebView` is good for rendering screens in an app that are entirely driven by HTML, rendering very complex CSS-heavy HTML, or, of course simply displaying a web page. I just needed to display some text that is coming from a web API that may or may not have some minimal HTML for formatting and linking.

## The issue with hyperlinks

I'm pretty sure the reason hyperlinks are not supported or overly complex in other libraries is because by default including hyperlinks in a `UILabel` is not supported on iOS. Since the Xamarin.Forms `Label` element is rendered down to a `UILabel`, this limits what can be done with hyperlinks in a custom `LabelRenderer`. However, hyperlinks can be included very easily in a `UITextView`. But how do we make a custom `LabelRenderer` use `UITextView` instead of `UILabel`?

## My solution

My solution is to use a custom `Label` class (named `HtmlLabel`) mapped to a custom `LabelRenderer` for Android, but **_not_** for iOS. For iOS I mapped `HtmlLabel` to a `UITextView` using a custom `ViewRenderer<>`. So, when using the `HtmlLabel` control from a `Page` or `ContentView` everything will act and work like a `Label` since it's literally just a subclass of `Label`. But, on iOS it will be rendered differently than how `Xamarin.Forms` would normally render a `Label`, using a `UITextView` instead of a `UILabel`.

First I made a new `Label` subclass named `HtmlLabel` that will be the "control" used to display HTML:

```csharp
public class HtmlLabel : Label 
{
}
```

I didn't need to add any new properties since the `Text` property from the base `Label` class is all that is needed to store the text value that contains the HTML.

### Android

Since HTML and hyperlinks are easily rendered in an `Android.Widget.TextView`, there is no issue simply subclassing `LabelRenderer` to add HTML rendering support:

```csharp
Control.TextFormatted = Build.VERSION.SdkInt >= BuildVersionCodes.N
    ? Html.FromHtml(Element.Text, FromHtmlOptions.ModeCompact)
    : Html.FromHtml(Element.Text);
```

And to include support for hyperlinks:

```csharp
Control.MovementMethod = Android.Text.Method.LinkMovementMethod.Instance;
```

That's it for Android... just a few lines of code in a custom `LabelRenderer`. Nothing out of the ordinary. 

### iOS

For iOS it's a slightly different story but just as simple - I want to render `HtmlLabel` using a `UITextView`, not a `UILabel`. I did this by subclassing `ViewRenderer<HtmlLabel, UITextView>` instead of `LabelRenderer`.

Now that the custom renderer is working with a `UITextView` as its native control, HTML rendering support can easily be added:

```csharp
NSError error = null;
Control.AttributedText = new NSAttributedString(
    NSData.FromString(Element.Text), 
    new NSAttributedStringDocumentAttributes { 
            DocumentType = NSDocumentType.HTML 
        }, 
    ref error);
```

Even though `Control` (the native control) is a `UITextView`, `Element` (the Xamarin.Forms element being rendered) is still a `Label` (via `HtmlLabel`), so `Element.Text` can still be used as well as any other `Label` properties just like we would in a basic custom `LabelRenderer`.

And to include support for hyperlinks:

```csharp
Control.ShouldInteractWithUrl += delegate { return true; };
```

Also, while not really required, it's nice to disable editing and scrolling so the `UITextView` behaves more like a label:

```csharp
Control.Editable = false;
Control.ScrollEnabled = false;
```

And that's all it takes to enable HTML (with hyperlinks) rendering support using a `UITextView`. Much simplier than trying to manipulate the `UILabel` in a custom `LabelRenderer` with a ton of complex code.

---

You can view all of the code for **HtmlLabel** on my [github](https://github.com/edsnider/htmllabel/) and you can get the package on [nuget](https://www.nuget.org/packages/XamForms.HtmlLabel/).