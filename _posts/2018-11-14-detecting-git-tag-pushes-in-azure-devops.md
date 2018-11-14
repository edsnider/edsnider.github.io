---
layout: post
title: "Detecting git tag pushes in Azure DevOps"
date: 2018-11-14 00:00:00 -0500
tags: devops git
excerpt_separator: <!--more-->
---

I've been using Azure DevOps (formerly VSTS) for several years to do CI/CD for mobile app projects. However, for library development I've been using AppVeyor - mainly because it's super simple to configure builds with YAML and it always seemed like the de facto CI/CD for NuGet libraries. Over the past year Microsoft has <a href="https://docs.microsoft.com/en-us/azure/devops/release-notes/2017/nov-28-vsts#configuration-as-code-yaml-builds-in-public-preview-build-tagimgrelease-notes-tagbuildpng" target="_blank">added</a> and <a href="https://docs.microsoft.com/en-us/azure/devops/release-notes/2018/apr-16-vsts#trigger-ci-builds-from-yaml" target="_blank">improved</a> support for YAML builds in Azure DevOps. So, that coupled with Azure DevOps' very nice release pipeline setup (which will also allow for YAML configuration <a href="https://docs.microsoft.com/en-us/azure/devops/release-notes/#2018-q4" target="_blank">soon</a>), I've decided to start moving CI/CD for my libraries (I only have a few) from AppVeyor over to Azure DevOps.

One of the things I depend on in my AppVeyor build configurations is the `APPVEYOR_REPO_TAG` boolean variable to know if I'm building from new tag that was pushed or a regular commit. I use this for setting versions and determining deployment environments. Unfortunately, Azure DevOps does not have a similar variable (as of Nov 2018). But that doesn't mean it isn't possible to do something similar...

<!--more-->

You can simply use the `Build.SourceBranch` predefined build variable. When a build is triggered by a regular commit the value of `Build.SourceBranch` will begin with `refs/heads/`. So, for example, when building from a commit pushed to the `master` branch the value of `Build.SourceBranch` will equal `refs/heads/master`. When a build is triggered by a new tag being pushed the value of `Build.SourceBranch` will begin with `refs/tags/`. So, for example, when building from a new tag named `v1.0.0` the value of `Build.SourceBranch` will equal `refs/tags/v1.0.0`.

Here is a PowerShell script you can include in your build definition that uses simple regular expression matching to determine if `Build.SourceBranch` is a tag or not:

```powershell
$isTag = [regex]::IsMatch($Env:BUILD_SOURCEBRANCH, "^(refs\/tags\/)")
```

## CI build triggers and git tag pushes
If you're using the build definition designer (instead of YAML) you'll need to add a branch filter to the CI trigger to include `refs/tags/*` so builds will be kicked off when tags are pushed.

If you're using YAML there is a CI trigger setup for all branches by default and therefore any tag pushes will kick off builds by default. However, if you've added specific branches to the trigger in your YAML file, you'll need to specifically add `refs/tags/*` to the list.

For example:

```yaml
trigger:
- master
- refs/tags/*
```

For more details on CI build triggers, checkout the <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/build/triggers?view=vsts&tabs=yaml" target="_blank">docs</a>.