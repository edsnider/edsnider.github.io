---
layout: post
title:  "Statically Defining Tabs on a Xamarin.Forms TabbedPage"
date:   2014-07-20 00:00:00 -0500
tags: xamarin xamarin.forms
---

In Xamarin.Forms screens are represented by Pages – a Page is a View Controller in iOS, an Activity in Android, and well.. a Page in Windows Phone.

One of the Page types that come with the Xamarin.Forms toolkit is `TabbedPage` which allows you to place tabs on the screen in a platform specific manner – so on the bottom bar in iOS, along the top in Android and as a pivot control in Windows Phone. The `TabbedPage` is a `MultiPage<Page>` so it is made up of a collection of Page objects, or subpages, which represent the content of the tabs.

There are a couple of ways of defining the child `Page`s (i.e., tabs) of a `TabbedPage` – you can set its `ItemsSource` which will give you a dynamic set of tabs based on a collection of items which you then define an `ItemTemplate` for, or you can simply set the tabs statically.

There are a couple good examples of how to accomplish the `ItemsSource`/binding approach both in the [API documentation](https://developer.xamarin.com/api/type/Xamarin.Forms.TabbedPage/){:target="_blank"} and in the [TabbedPageDemo](https://github.com/xamarin/xamarin-forms-samples/tree/master/Navigation/TabbedPage){:target="_blank"} in the [Xamarin.Forms Sample GitHub repo](https://github.com/xamarin/xamarin-forms-samples){:target="_blank"}. The “static” approach isn’t specifically called out in the samples, but you can actually find an example of how to accomplish it in the [MobileCRM sample](https://github.com/xamarin/xamarin-forms-samples/tree/master/MobileCRM){:target="_blank"}.

I think it’s typically more common to set tabs on a mobile app up based on a couple of known items (which more than likely will have different content layouts) rather than a databound collection of items. So I thought I would specifically call out how this can be accomplished with a couple quick samples.

In XAML:

<script src="https://gist.github.com/edsnider/e1dd04b58c4eaae1d2eb.js?file=StaticTabs.xaml"></script>

In C#:

<script src="https://gist.github.com/edsnider/e1dd04b58c4eaae1d2eb.js?file=StaticTabs2.cs"></script>

Or, just add other Pages directly:

<script src="https://gist.github.com/edsnider/e1dd04b58c4eaae1d2eb.js?file=StaticTabs3.cs"></script>