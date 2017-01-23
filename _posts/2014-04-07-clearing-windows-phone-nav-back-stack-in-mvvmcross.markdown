---
layout: post
title:  "Clearing Windows Phone Nav Back Stack in MvvmCross"
date:   2014-04-07 00:00:00 -0500
tags: mvvmcross windows windowsphone
---

There are often times when you might need to clear the navigation stack (aka back stack) in an app.  One common example of this is when you allow a user to sign out – you don’t want them to be able to tap back and still have the ability to see the previous views.  As you can image, this is done differently on each platform and therefore it can’t be done directly from a ViewModel – but there is a way to control it from the ViewModel.

In [MvvmCross](https://mvvmcross.com/){:target="_blank"} you can create “Presentation Hints” that can be passed to the UI/view directly from a ViewModel.  Presentation Hints are handled by a ViewPresenter and since ViewPresenters live in the view layer and are platform specific you can detect the hints passed from the ViewModels and handle them in platform specific ways.

So in the case of clearing the back stack [you can create a “ClearBackStackPresentationHint”](https://github.com/MvvmCross/MvvmCross/wiki/ViewModel--to-ViewModel-navigation#how-do-i-remove-views-and-viewmodels-from-the-back-stack){:target="_blank"} in your PCL which really has no implementation and then pass a new instance of it into the `ChangePresentation()` method which you get when you inherit from `MvxViewModel`.

<script src="https://gist.github.com/edsnider/10035854.js?file=ClearNavBackStackHint.cs"></script>

Then, to handle the hint you need to make a custom ViewPresenter inheriting from the platform specific version of `IMvxViewPresenter` (e.g., inherit from `MvxPhoneViewPresenter` for Windows Phone, `MvxAndroidViewPresenter` for Android, or `MvxTouchViewPresenter` for iOS).  Finally, you need to register your custom ViewPresenter so it is used over the default one.  You do this in the `CreateViewPresenter()` method of your `Setup`class.

Here is how to do it for Windows Phone:

<script src="https://gist.github.com/edsnider/10035854.js?file=CustomWP8ViewPresenter.cs"></script>

<script src="https://gist.github.com/edsnider/10035854.js?file=Setup.cs"></script>

And then from your ViewModel you can do this and the back stack will be cleared:

{% highlight csharp %}
ChangePresenation(new ClearNavBackStackHint());
{% endhighlight %}

Since the `Show()` method gets called before the `ChangePresentation()` method in the ViewPresenter you have to be careful the order in which you call the `ChangePresnetation()` method in your ViewModel. If you call it before you call the `ShowViewModel<>()` method (in a situation where you want to leave the current view and clear the stack) you will end up with that last view still on the stack.  To fix this simply call the `ChangePresentation()` method after calling the `ShowViewModel<>()` method.  Taking this a step further I typically add a `ShowViewModelAndClearBackStack<>()` method to my `BaseViewModel` class so it can be handled in one place and you know the order is always consistent.

<script src="https://gist.github.com/edsnider/10035854.js?file=BaseViewModel.cs"></script>

Now, from your ViewModel just call `ShowViewModelAndClearBackStack<>()` and the user will be taken to the view with no way of going back (other than to exit the app).

{% highlight csharp %}
ShowViewModelAndClearBackStack<WhateverViewModel>();
{% endhighlight %}