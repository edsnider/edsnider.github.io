---
layout: post
title:  Integrating Visual Studio Mobile Center and Trello using Azure Functions
date:   2017-11-03 00:00:00 -0500
tags: xamarin mobilecenter ci azure azurefunctions github trello devops appcenter vsappcenter
excerpt_separator: <!--more-->
---

**UPDATE (11/15/17):** Today Microsoft [announced](https://blogs.msdn.microsoft.com/vsappcenter/introducing-visual-studio-app-center/) the general availability of Visual Studio App Center. Mobile Center is now App Center.
- - -

My current project is an awesome project. We're using Xamarin.Forms to build beautiful native iOS and Android apps. And, if that isn't awesome enough, we've totally automated the build and distribution process of these apps using [Visual Studio Mobile Center](https://www.visualstudio.com/app-center/). I absolutely love Mobile Center and how simple it makes setting up CI/CD. You can literally setup a build job in less than 5 minutes with no build host machines or any of those types of complexities that normally come with setting up CI. As you can tell I'm pretty excited we're using Mobile Center to automate the app builds and releases for this project. But there was one thing I was still having to do manually - updating the Trello board after each build. Clearly this must be automated as well!

<!--more-->

On this project we use Trello for managing our backlog and current work. It is a pretty standard workflow. There are Trello cards for features, tasks, and bugs. The cards start off in the "Backlog" list and then they move to the "Doing" list when they are in progress by a developer and then they finally move to a "Done/Ready for Testing" list when they are in a build release and ready to be tested by the QA team. So while the entire build process is automated (kicked off when a PR is merged into the main develop branch) I was still having to go into Trello after each build and move the card to the "Done/Ready for Testing" list and update it with a comment containing the version and build number of the app release - a manual and tedious process that is easy to forget, so I decided to automate it as well.

{% include image.html url="/assets/postimages/mobilecenter-trello-dev-process.png"  file="/assets/postimages/mobilecenter-trello-dev-process-sm.png" caption="The development workflow (click to enlarge)" alt="The development workflow" %}

## Enter webhooks and Azure Functions

The Mobile Center team is constantly adding new capabilities making it more robust and easier to use.  

They recently added the ability to use scripts in build definitions which was one of my number one feature requests since Mobile Center debuted. I've already started taking advantage of this feature and it's great! In fact, I initially set out to automate the process of updating Trello cards using a post-build script. Well, until I discovered a couple weeks ago that Mobile Center snuck in webhook triggers!

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">loving the ability to use build scripts in <a href="https://twitter.com/hashtag/vsmobilecenter?src=hash&amp;ref_src=twsrc%5Etfw">#vsmobilecenter</a> ! also just stumbled upon webhooks in app settings!! 😍 <a href="https://twitter.com/MobileCenter?ref_src=twsrc%5Etfw">@MobileCenter</a></p>&mdash; Ed Snider (@edsnider) <a href="https://twitter.com/edsnider/status/916336334607970305?ref_src=twsrc%5Etfw">October 6, 2017</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

As of this post, Mobile Center has the ability to trigger a webhook when a new crash group is created and/or when a new build release is available. This is perfect for solving my problem - When a new build is released I need to do something.

### Creating a webhook triggered Azure Function

It turns out Azure Functions make [setting up a webhook triggered function](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-generic-webhook-triggered-function#create-function) super easy. I'm new to Azure Functions, so this was a great excuse for me to dive in and play with them. For what I needed to do I simply used the "Webhook + API" quickstart function template and selected C# for my language preference. This set me up with a blank function that I could start adding logic to.

Below is a quick logic flow that I defined for the Azure Function:
- Get the release's notes, version number, build number and platform (this comes through with the webhook's json payload)
- Extract the pull request number from the release notes
- Get the pull request details from GitHub using the [GitHub API](https://developer.github.com/v3/pulls/#get-a-single-pull-request)
- Extract the Trello card identifier from the pull request body
- Add a comment to the Trello card containing the release details using the [Trello API](https://developers.trello.com/v1.0/reference#cardsidactionscomments)
- Move the Trello card to the "Done/Ready for Testing" list using the [Trello API](https://developers.trello.com/v1.0/reference#cardsid-1)

As you can see, this logic relies on some specific procedures, for example, including the Trello card URL in pull requests. In my experience, it is not uncommon to include some sort of card or issue identifier in a pull request description - the easiest way to do this with a Trello card is to just copy and paste in the card's share URL.

The actual code for this Azure Function is pretty simple - just some basic pattern matching and HTTP calls. I've made the function I used available in a [Gist](https://gist.github.com/edsnider/26e95a57acb9a913589cf048278d9830).

Once you've added your Azure Function the only thing you need to do is copy the function's URL and add it to a new webhook in your app's settings in Mobile Center.

{% include image.html file="/assets/postimages/mobilecenter-new-webhook.png" caption="Adding a new webhook in Mobile Center" alt="Adding a new webhook" %}