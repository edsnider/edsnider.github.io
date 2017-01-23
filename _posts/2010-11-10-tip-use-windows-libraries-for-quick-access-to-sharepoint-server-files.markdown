---
layout: post
title:  "Tip: Use Windows Libraries for Quick Access to SharePoint Server Files"
date:   2010-11-10 00:00:00 -0500
tags: sharepoint sharepoint2010
---

As a SharePoint developer or server admin you’ll find yourself in and out of the 14 hive all day long.  You also make frequent trips to the various web app virtual directory folders.  Both of these key locations are fairly deep and it can get rather annoying traversing down into them.  So I started taking advantage of the Library feature that comes with Windows Server 2008 (and Windows 7, of course) to make it easier to access the files in these locations.

As you can see in the screenshot below I’ve setup a couple Libraries.  I setup a 14 Hive Library that basically just gives me a quick link right to the 14 folder.  I also setup a SP Virtual Directories Library in which I’ve included each of the web application root folders.  So now with only one click I can access all of my web.config files and bin directories in a single location – pretty handy.  “Windows 7 was my idea!”

{% include image.html file="/assets/postimages/windowslibraries.png" caption="" alt="Windows Libraries" %}