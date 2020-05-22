---
layout: post
title:  "NuGet local repository"
date:   2020-05-19 15:09:28 +0300
tagline: Faster development of projects using NuGet packages
categories: nuget
image: assets/img/2020-05-19-nuget-local/nuget_logo.png
---

[NuGet](https://www.nuget.org) is amazing thing and written a huge amount of articles that describe all benefits of used it. Many companies produce a lot of packages and publish it to public or [private](https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server) NuGet repository. There is nothing easier to add NuGet package into your project and start using it. 

But what if you are a consumer and a producer of a package at the same time? If improvements to the package's API are required due to a change in the consumer’s project? In this case, you would like to modify the package code, check its new functionality in the dependent project, and only after that deploy new package version to the repository and share it with another developers.

For this scenario, working with an intermediate local NuGet repository, which will be just a folder on your hard drive, can be very convenient. So let's configure your local NuGet repository.

Create a folder for your local repository on a hard drive and save path to it in the environment variable `LOCAL_NUGET_REPOSITORY`. After that, you need to add new NuGet source, with the command:

{% highlight bash %}
nuget sources add -Name Local -Source "%LOCAL_NUGET_REPOSITORY%"
{% endhighlight %}

That's it, your local repository is configured and registered as a NuGet source. It remains only to fill it with your packages.

For `SDK-style` projects, you can very easy add `Task` that copy the `.nupkg` files to the directory of your local repository. That's this `Task`:

{% highlight xml %}
<Target Name="CopyToLocalNugetRepository" AfterTargets="Pack" 
        Condition="$(LOCAL_NUGET_REPOSITORY)!=''">

    <Copy SourceFiles="bin\$(Configuration)\$(PackageId).$(PackageVersion).nupkg" 
          DestinationFolder="$(LOCAL_NUGET_REPOSITORY)"
          Condition="Exists('bin\$(Configuration)\$(PackageId).$(PackageVersion).nupkg')"/>
</Target>
{% endhighlight %}

Add this `Task` to your `.csproj` file and run the [pack](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-pack) command, or enable generate NuGet package on build. That's it, now after packaging, your package will be automatically published to the local repository. You can easily add a new package from this local NuGet source or upgrade to a newer version.

If you change the code in the NuGet project, it will be automatically published to the local repository after packaging, but your project that consume this package will not see the changes. It's happen because NuGet have a cache. And we need to clean it with every publication to local repository. So, we extend `Task` and add cache clear command:

{% highlight xml %}
<RemoveDir  Directories="$(NugetPackageRoot)\$(PackageId)\$(PackageVersion)"
            Condition="Exists('$(NugetPackageRoot)\$(PackageId)\$(PackageVersion)')" />
{% endhighlight %}

Now you can change NuGet project very easily and immediately test it with a depended project. After the new package API is fully developed and tested, you can publish the new version of your package to the public or private NuGet repository and make it publicly available.

Completed code:

<script src="https://gist.github.com/ggaller/c4c5792c07df1f442e6108bd1b34ec39.js"></script>
