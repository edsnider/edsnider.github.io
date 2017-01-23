---
layout: post
title:  "Building Apps for Xbox One with UWP"
date:   2016-03-30 00:00:00 -0500
tags: xbox uwp windows10 
---

As you probably heard in the keynote at **//build** today – it is now possible to leverage the Universal Windows Platform (UWP) to build apps for Xbox One!  Since the last //build in 2015 we have been using UWP to build single package apps that can be shared by phones and desktops that run Windows 10 – a huge step up from the Windows 8.1 “Universal” app development story.

This is huge news for Windows developers, especially those that aren’t existing Xbox developers.  With this UWP model, the developer tools are available to anyone with Visual Studio – making the barrier to entry very low.  Lowering that barrier even further is the fact that your existing UWP Windows 10 apps will run as-is on Xbox One.  So, if you have an existing Windows 10 app for desktop or mobile you’re probably 75-90% done with your first Xbox One app before even writing a line of new code!

In this blog post I’m going to discuss the key things you need to know to successfully take your UWP app to the Xbox One.

## What’s different?

Before we start with what it takes to leverage UWP for Xbox One, let’s first understand what is different when it comes to the Xbox One.

Honestly, the Xbox One console doesn’t present anything different from an API or platform perspective – which makes the UWP concept a reality.  The main difference we need to actually think about is how to adapt to TVs and leverage gamepads and/or remotes.

### TV over scan

Some TVs have different visible areas than others.  There is an area that is guaranteed to be visible on all TVs and then there is the rest of the area that may or not be seen depending on the TV’s make, model and/or settings… this area is known as the over scan area. 

### TV colors

TV displays handle colors different than the desktop, laptop, and mobile displays we are used to.  In fact, while most TVs support RGB 0-255, according to specifications they support RGB 15-235.

### Gamepads

The primary input for an app running on the Xbox One is a gamepad, unlike the desktop where its keyboard and mouse or tablets and phones where it’s primarily touch.

So now that we know the main differences when working with UWP on TVs – let’s dig into them a bit more and see how to work with these differences.

## Working with the TV “safe area”

With each TV model having a different visible area we have to work with what is known as the “TV Safe Area” when building apps for the Xbox One.  The area outside of this “safe area” is known as the overscan and it may or may not be visible depending on the TV.  This means we need to make sure our app is fully visible within the “safe area” so it isn’t clipped when running on a TV that doesn’t have the overscan area, this is the default behavior.  However, by doing this you will end up with a margin around your app on TVs that do have the overscan area. 

In code the full window size can be found by calling:

`Windows.UI.Core.CoreWindow.GetForCurrentThread().Bounds;`

When running on an Xbox One console, this will always translate to 960 x 540.

The size of the TV safe area can be found by calling:

`Windows.UI.ViewManagement.ApplicationView.GetForCurrentWindow().VisibleBounds;`

The difference between the values of `Bounds` and `VisibleBounds` in these two statements represents the overscan area. 

Because each TV will handle displaying the app differently, the size of the overscan area will be different.  The minimum size of the `VisibleBounds` (TV safe area) is 864x486 which means the maximum size of the overscan area is 48px on the left and right and 27px on the top and bottom.

As mentioned above, your app will fill within the `VisibleBounds` by default - the only thing you need to do is ensure the overscan area is handled appropriately to avoid a black margin around your app on some TVs.  To do this use negative margins on the elements of your UI.  As the developer you are responsible for making sure all of your app's content is properly visible within the TV safe area, however, the `Page.Background` will automatically cover the over scan area by default.

## Working with TV safe colors

The colors displayed on a TV vary from model to model.  The actual specification for TV colors in RGB is 15-235, as opposed to the full range of 0-255.  Therefore, when building an app that will run on a TV it is important to adjust your colors to fit within this range, including images and icons. 

In addition to supporting TV safe colors in your XAML – it is also a good idea to think about the overall contrast of your app.  While a white background might look great on a mobile device it could be overwhelming and too bright when displayed on a TV screen.  In most cases a darker background provides a better user experience for an app running on a TV.

## Working with a gamepad

The best way to ensure your app is able to work with an Xbox gamepad is to make sure it works with a keyboard first.  If you can control your app entirely with a keyboard using the directional arrows, tab, space, and escape then you can be sure the Xbox gamepad will control your app just as well.  The D-pad and left joystick on the gamepad control focus just as tab on the keyboard does.  The *A* button on the gamepad maps to the space bar on a keyboard.  The *B* button on the gamepad maps to the escape button on a keyboard as well as the navigation back event.  You can handle gamepad button events using the traditional `KeyDown` and `KeyUp` events. The event arguments `Key` property will give you the actual gamepad key that was pressed.  The `OriginalKey` property will give you the traditional keyboard equivalent, if applicable.

Although you can test functionality using a keyboard, it is a good idea to also test using a gamepad even if you do not have a console.  You can connect an Xbox One gamepad to your Windows machine using a USB cable and use it to control your app running in Desktop mode to completely test it's functionality with the gamepad.

## How to adapt your UI specifically for Xbox One

While your existing UWP app might run perfectly fine on a TV via the Xbox One console, you might decide to adapt its UI to support the different experience your users have.  When your users are using your app on a phone, computer, or tablet we assume they are within inches of our app and therefore we design a user experience specific to that use case.  However, when your users are using your app on their TV we assume they are potentially upwards of 10 feet away from our app, sitting comfortably on a couch.  The user experience we design for a 10-inch use case is probably not going to be the same as a 10-foot use case.  Some UI elements simply need to be larger, whereas others may not even make sense in a TV use case.  

>NOTE: By default, your app will be displayed at 200% larger when running on the Xbox One console, therefore things will automatically be larger.  This means an app with a display area of 540x960 will actually make up the full display area of a 1080x1920 TV screen.

We can adapt our UI in several ways, just as we do when we support the differences between the desktop and mobile device families.  There are three main ways of adapting your UI and depending on your scenario you may need only one, or a combination of them.

### Triggers

If you only need to make simple changes to the app’s layout, *Adaptive Triggers* are often your best bet.  For example, if you want to make a `StackPanel` orient to vertically on a phone screen, but on Xbox you want it to span horizontally, you could use a simple trigger.  Triggers for the Xbox device family are no different than what might be used for *Mobile* or *Desktop*.  In fact, if you already have a device family trigger written for *Mobile* and *Desktop* adaptations, you are already done – you just need to pass “Windows.Xbox” into the trigger in XAML, similar to “Windows.Mobile”or “Windows.Desktop”.  Here is a simple device family trigger and example of how to use it to adapt your UI when running on the Xbox One console:

<script src="https://gist.github.com/edsnider/31a7b3b4a3c26085c14d.js?file=DeviceFamilyTrigger.cs"></script>

<script src="https://gist.github.com/edsnider/31a7b3b4a3c26085c14d.js?file=MainPage.xaml"></script>

### Separate XAML views

In some situations, you may decide to deliver a completely different UI for your users that access your app on the Xbox One console.  In this case, it may get a bit messy trying to manage visual states with triggers and it’s probably better to create a new XAML view.  When UWP first came out the concept of “XAML Views” was introduced for this exact purpose.  A XAML View enables you to create new XAML definitions for the different device families, but they can all share the same code-behind. 

## Summary

As you can see, if you have already invested in UWP there really isn’t much more than some minor polish you need to do to get your app up and running on a TV through the Xbox One console.  If you haven’t already used UWP but want to get into Xbox One development, it offers a very straightforward approach to doing so.

*This blog post was originally posted on the InfernoRed Technology blog at [blog.infernored.com](http://blog.infernored.com/building-apps-for-xbox-one-with-uwp){:target="_blank"}.*