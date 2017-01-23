---
layout: post
title:  "SharePoint Custom Role Provider – Important RoleManager Attribute Settings"
date:   2009-06-09 00:00:00 -0500
tags: sharepoint
---

Be sure to include the `enabled` attribute in your `roleManager` tag and set it to `true`. It is `false` by default which means if you don’t include the attribute your role provider isn’t going to work. Note: There is no enabled attribute for the membership tag.

Part of setting up a custom forms based authentication is adding the membership and role providers to the Central Administration’s web.config as well. When doing this be sure to set the default role provider to the default Windows authentication provider by setting the value of the `defaultProvider` attribute of the `roleManager` tag to `AspNetWindowsTokenRoleProvider`.