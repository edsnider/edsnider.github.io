---
layout: post
title:  "SharePoint 2010 - Adding Managed Accounts for Local Accounts"
date:   2010-08-03 00:00:00 -0500
tags: sharepoint sharepoint2010
---

If you build a 2010 dev machine as a single server complete setup (instead of stand alone mode) and go the local account route for service and worker process accounts you will run into some issues when setting up Managed Accounts.  Just like during configuration, SharePoint wants you to use local accounts only with stand alone mode.

When you attempt to register a Managed Account that points to a local user account you will get the following message:

>“The specified user &lt;machine_name&gt;\&lt;account_name&gt; is a local account.  Local accounts should only be used in stand alone mode.”

{% include image.html file="/assets/postimages/localaccountasmanagedaccounterror.jpg" caption="" alt="Local account as managed account error" %}

You can’t add Managed Accounts pointing to local accounts via the Central Admin web interface but you can do it using PowerShell for SharePoint 2010.

Simply run the `New-SPManagedAccount` cmdlet.  No need for parameters as it will prompt you for credentials of the account you are associating the new Managed Account with.  When prompted just type in the username and password for the local account you’re using.

{% include image.html file="/assets/postimages/new-spmanagedaccount_poshcmd.png" caption="" alt="New-SPManagedAccount PowerShell Command" %}

Click OK and the Managed Account is created.  Notice you still get the warning about only using local accounts in stand alone mode only this time the account was actually created, regardless.

{% include image.html file="/assets/postimages/new-spmanagedaccount_poshcmd_2.png" caption="" alt="New-SPManagedAccount PowerShell Command" %}

Go back into Central Admin and take a look at the Managed Accounts list and you’ll see the local account is there and ready to use when firing up service apps or creating web applications.

>Note:  I only recommend this bypass for development purposes when setting up a DC would be overkill.  It is not advised to run production SharePoint services or processes using local accounts.