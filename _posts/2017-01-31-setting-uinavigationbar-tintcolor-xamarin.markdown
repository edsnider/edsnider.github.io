---
layout: post
title:  Setting UINavigationBar TintColor (Xamarin.iOS)
date:   2017-01-30 00:00:00 -0500
tags: xamarin ios
excerpt_separator: <!--more-->
---

#### TL;DR

``` csharp
UIBarButtonItem.AppearanceWhenContainedIn(typeof(UINavigationBar))
        .SetTitleTextAttributes(attributes, UIControlState.Normal);
```

<!--more-->

#### Background

If you need to set the color of buttons and items, specifically `UIButtonBarItem` objects, in the Navigation Bar across your app you can set it using the static properties on the `UIButtonBarItem.Appearance` object.  However, it is important to note that this will impact the color of **ALL** `UIButtonBarItem`s throughout the app - including those used in `UIToolbar`s.  

A common use of `UIToolbar` is to add custom functionality to the `InputView` of a `UITextField` (for adding buttons above the keyboard).

I recently ran into a situation where the buttons I had added to the keyboard toolbar suddenly (perhaps over the course of some commits) turned white (or, completely missing according to my end users).  I was racking my brain trying to figure out what could have changed to make these buttons go from the default blue color to white.  Eventually I came across a line of code doing the following:

``` csharp
UIBarButtonItem.Appearance
        .SetTitleTextAttributes(attributes, UIControlState.Normal);
```

This code was added so all buttons in the navigation bars would be white, but the side effect was it made all toolbar buttons throughout the app white as well.

Luckily, iOS provides a way of specifying an appearance for a specific object type - allowing you to restrict the style of the `UIButtonBarItem`s to a specific area of the app.   So in the case where we only intended to impact the appearance of the navigation bar I updated the above code to the following:

``` csharp
UIBarButtonItem.AppearanceWhenContainedIn(typeof(UINavigationBar))
        .SetTitleTextAttributes(attributes, UIControlState.Normal);
```