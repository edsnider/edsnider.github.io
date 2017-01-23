---
layout: post
title:  "Boot to VHD - Copying VHDs"
date:   2010-07-27 00:00:00 -0500
tags: vhd bcdedit
---

If you have a VHD file that you boot to (Boot to VHD) and you want to copy it and boot to that copy as well you will need to use `bcdedit` as follows:

### Step 1:

First make a copy of your actual VHD file and name the copied file to whatever you want.

### Step 2: 

Open a command prompt (as Administrator)

### Step 3:

Get the GUID of your original VHD by using running `bcdedit /v` – this will display your current boot loaders.  As you can see in the screen shot below I have 2 (Windows 7 and SharePoint 2010 Dev (2008 R2)).  The later is the boot to vhd loader which you can see by inspecting the **device** and **osdevice** settings.  The GUID you need capture is the value of the **identifier** setting.

{% include image.html file="/assets/postimages/bcdedit_view.png" caption="" alt="bcdedit view" %}

### Step 4:  

Execute the `bcdedit /copy` command using your original VHD’s GUID obtained in the previous step.  As you can see below after issuing the `/copy` command I re-ran the `/v` command and there is now a third loader entry with the new description (EPM 2010 Dev (2008 R2) in my case)… But, notice the **device** and **osdevice** settings are pointing to the original VHD file.

{% include image.html file="/assets/postimages/bcdedit_view_2.png" caption="" alt="bcdedit view" %}

### Step 5:

In order to associate the new loader option with the new VHD file you created by copying your original file you’ll need to execute `bcdedit /set {GUID} device` and `bcdedit /set {GUID} osdevice` commands using the GUID of the new loader’s **identifier** setting.  The `/set` commands set the **device** and **osdevice** settings to the correct VHD file.

{% include image.html file="/assets/postimages/bcdedit_set.png" caption="" alt="bcdedit set" %}