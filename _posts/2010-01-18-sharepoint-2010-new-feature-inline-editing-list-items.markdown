---
layout: post
title:  "SharePoint 2010 - New Feature: Inline Editing List Items"
date:   2010-01-18 00:00:00 -0500
tags: sharepoint sharepoint2010
---

In SharePoint 2010 you can now edit list items directly from within the list view itself using the new Inline Editing feature.

### Enabling it… 
The Inline Editing feature is not enabled by default (beta version). The feature is a per-view setting that can be enabled on the list view edit page. To edit a view and turn this feature on click Modify View on the List ribbon. On the View edit page expand the Inline Editing section and check the Allow inline editing checkbox.

{% include image.html file="/assets/postimages/inlineediting_settings.png" caption="" alt="Inline Editing Settings" %}

### Using it… 
Once inline editing has been enabled for a list view you will notice an edit icon (datagrid with pencil) next to the list items as you hover over their rows. There is also a new icon (green plus sign) below the list for adding new items inline.

{% include image.html file="/assets/postimages/inlineediting_editicon.png" caption="" alt="Inline Editing Edit Icon" %}

When you click the edit icon the values in each column in that row are replaced with form fields corresponding to the column’s type and pre-populated with the row’s value. Notice even the managed metadata column type works with the inline editing functionality just as it would on the standard entry dialog. Simply click the save icon (blue diskette) at the beginning of the row to save your changes.

{% include image.html file="/assets/postimages/inlineediting_editing.png" caption="" alt="Inline Editing" %}

Clicking the new icon (green plus sign) sets up a new blank row and allows users to create new items in the list view as well.

{% include image.html file="/assets/postimages/inlineediting_newitem.png" caption="" alt="Inline Editing New Item" %}