---
layout: post
title:  "Who moved my cheese? The case of the missing launcher icon after updating an Android app"
date:   2018-04-13 00:00:00 -0500
tags: android xamarin xamarin.forms
excerpt_separator: <!--more-->
---

I'm working on a project where we're building an Android app using Xamarin.Forms and this app will actually be an update to an existing Java-built Android app (same package name). Some beta testers started reporting that after getting the update their "shortcut" launcher icon was missing.  

<!--more-->

Thanks to [this blog post](https://android.jlelse.eu/android-and-the-mystery-of-the-disappearing-launcher-icon-154f9267f98e) I learned that when you add a shortcut icon to your launcher, you're actually creating a link to an Activity, not to the app itself. The post talks about using an `<activity-alias>` tag in `AndroidManifest.xml` to map the name of the old version's Activity to the new version's Activity. However, it took me a few tries to get things working - based on this blog post along and several StackOverflow and Xamarin forums hits it still wasn't 100% clear to me how exactly this `<activity-alias>` tag worked and needed to be configured, especially since in this Xamarin.Forms app the Activity isn't defined in the `AndroidManifest.xml` file (no `<activity>` tags).

The name of the main launch Activity in the old version of the app was `.activities.ListActivity` and in the new Xamarin.Forms app `MainActivity.cs` class the `[Activity]` attribute actually didn't have a Name at all. After some trial and error and several `[INSTALL_PARSE_FAILED_MANIFEST_MALFORMED]` failures I learned that the `android:name` and `android:targetActivity` attributes of the `<activity-alias>` tag both need to be prefixed with the package name and that same package name-prefixed name needs to be added to the `[Activity]` attribute of the `MainActivity` class. Here is the final outcome that worked and preserved the original app version's shortcut icon:

In `AndroidManifest.xml`:
```xml
<application>
  <activity-alias
    android:name="com.company.app.activities.ListActivity"
    android:targetActivity="com.company.app.MainActivity"
    android:exported="true">
    <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
  </activity-alias>
</application>
```

In `MainActivity.cs`:
```csharp
[Activity(Name = "com.company.app.MainActivity",
          ...
         )] 
public class MainActivity : FormsAppCompatActivity 
{ 
    ... 
}
```

> Note: Make sure you remove `MainLauncher = true` from the `[Activity]` attribute in `MainActivity` or you will end up with two app icons.
