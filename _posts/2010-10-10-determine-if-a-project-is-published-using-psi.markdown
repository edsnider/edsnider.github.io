---
layout: post
title:  "Determine if a Project is Published Using PSI"
date:   2010-10-10 00:00:00 -0500
tags: projectserver psi
---

Believe it or not there is no property of the Project class in the PSI Project Web Service that tells you whether or not a project is published.  One might expect something like IsPublished or PublishStatus or even IsDraft but unfortunately none of these exist.

So, I’ve found a way of checking if a project is published by using the Project Web Service `ReadProject` method:

{% highlight csharp %}
bool isPublished;
ProjectWebService.ProjectDataSet projectDataSet = projectService.ReadProject(projectUID, ProjectWebService.DataStoreEnum.PublishedStore);
isPublished = projectDataSet != null;
{% endhighlight %}

The `ReadProject` method takes two parameters, the first being the unique identifier of the project you’re checking on and the second being the data store to read the project from.  The data store parameter is of the `DataStoreEnum` type.  As you can see in my example above, if you pass in the `DataStoreEnum.PublishedStore` value for the data store parameter the method will attempt to retrieve the project from the published database.  If this returns a null dataset then you can assume the project has not been published.

This should work for Project Server 2007 and 2010.