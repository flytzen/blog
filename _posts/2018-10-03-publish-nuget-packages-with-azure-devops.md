---
layout: post
title: Publsh Nuget packages with Azure Dev Ops
date: '2018-10-03'
author: Frans Lytzen
tags: DevOps
modified_time: '2018-10-03'
excerpt: Use GitFlow and Azure Devops to automatically publish Nuget packages with sensible version numbers
---
Whenever I decide to create a Nuget package, whether for OSS or to publish on our internal MyGet feed I end up spending an inordinate amount of time trying to figure out a flow that works for testing and publishing. I guess it's one of those things that, once you have figured it out, becomes easy but it has eluded me until recently.  

My requirements are quite specific and may not be to everyone's liking;
- I want to use [GitFlow](https://datasift.github.io/gitflow/IntroducingGitFlow.html) to control my branches, including using Pull Requests etc.
- Whenever a commit is made to `develop` (or a PR is merged in), I want to publish that package with a "`-pre.123`" suffix as per [SemVer].(https://semver.org/)
- Whenever the same happens to `master` I want to publish a "full" release.

I haven't explicitly covered it here, but it would also be nice to have packages sat in a `release/*` branch be published with a `-beta.1` suffix - but you can easily extend it to cover that scenario as well.

In this post I will show how to set up GitHub with Azure DevOps to do this for us.

I am basing this on "modern" (i.e. 2017) csproj files, the ones where the package references are in the .csproj files. This came in with .Net Core but works fine with Full Framework projects.


## Version numbers
When you create and publish Nuget packages you can specify the version number you want to use on the command line and there is ample of documentation about how to do that with Azure DevOps and other builder services, including MyGet build services, which I used previously.  
However, I really like more control so I like to control the version number in my .csproj file - but I want the build service to automatically append `-pre.nnn` when it published from the `develop` branch.

The first thing to understand is that there are two ways you can specify the version number in your `.csproj` file:
```xml
<version>1.2.3-pre.987</version>
```
or
```xml
<VersionPrefix>1.2.3</VersionPrefix>
<VersionSuffix>pre.987</VersionSuffix>
```

Both the above will create packages with version `1.2.3-pre.987`. The naming of the Prefix and Suffix threw me for the longest time - I thought they were meant to interact with the `<Version>` attribute somehow, but Prefix and Suffix is more like "main part" and "extra bit" and you should *either* those *or* `<Version>`.

The second thing to understand is that you can use conditionals and environment variables in the attributes. 
For my purposes, this is what I ended up with:

```xml
<VersionPrefix>1.2.3</VersionPrefix>
<VersionSuffix></VersionSuffix>
<VersionSuffix Condition=" '$(Configuration)' == 'Debug' ">debug</VersionSuffix> <!-- For local/debug builds -->
<VersionSuffix Condition=" '$(Build_SourceBranch)' == 'refs/heads/develop' ">pre.$(Build_BuildID)</VersionSuffix> <!-- This is using variables that are specific to Azure Dev Ops Pipelines -->
```

My `<VersionPrefix>` here is really the proper version I want my package to have.  
I have an empty `<VersionSuffix>` as default. I probably don't actually need that tag, but it helps make it clearer in my mind.  

The next `<VersionSuffix>` uses the `Configuration` variable that is provided by the dotnet build process; if I build in Debug mode, the package version will become `1.2.3-debug`. This is mainly useful for local scenarios as I will always build in Release mode for publishing.  

The `<VersionSuffix>` after that looks at an environment variable provided by Azure DevOps when you are running in the pipeline. This means that if I am building from the `develop` branch in an Azure Pipeline then it will set the Suffix. `Build_BuildID` is another environment variable provided by Azure Dev Ops to the pipeline, which will always increment. So, in the example here I may end up with a version number of `1.2.3-pre.6239`. As long as that last nunber reliably increments (which it does) you are fine for package control.  
There is another variable called `Build_BuildNumber` which you may be tempted to use instead. However, I found some scenarios where that variable would have the name of the pipeline instead of a number, which causes the build to fail.

For more advanced scenarios you can invent your own attributes, which become variables in their own right, which you can then re-combine in other ways.

## Publish Symbols
Traditionally, when you create a Nuget package, it *won't* include the `pdb` files (the debug symbols). In the past, the answer was to `--include-symbols` when building your Nuget pacakge. This will create *two* Nuget packages, one with the `pdb` files and one without. Up until a few years ago, you could publish both of these together to Nuget, but then that changed and now you have to publish the symbols package to a different server with a different API key and a different command. It becomes a real headache, especially because of the inconsistent and out of date documentation. Hence why I have included it in this guide; I either need to tell you how to publish symbols from within the pipeline or tell you how to avoid it.  

[SourceLink](https://github.com/dotnet/sourcelink) to the rescue. SourceLink provides a way to link your package to a specific commit on, say, GitHub or elsewhere. I do recommend using SourceLink as it does so much more than just give you the PDB file - but even if you can't or won't, there is a gem hidden in the documentation, namely this line to add to your `.csproj`:
```xml
<AllowedOutputExtensionsInPackageBuildOutputFolder>
  $(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb
</AllowedOutputExtensionsInPackageBuildOutputFolder>
``` 

What this will do is include the `pdb` file in your main Nuget package, meaning you don't need a separate symbols package at all. Of course, using the full SourceLink is much better. Incidentally, this also works for private repos without sharing the source publicly.

## Variables
When you are looking at the documentation for Azure DevOps there are lists of variables scattered in different places. You will probably also find that the same variable in some context is referred to as Build.BuildId and in another as BUILD_BUILDID etc. Sometimes you have to reference it as %BUILD_BUILDID%, other times as $(Build.BuildId) and yet other times as $(Build_BuildID). It does sort of make sense, but as a good starting point, when designing your YAML file, I recommend adding this task somewhere:
```yaml
- script: set
  displayName: show variables
```
It just dumps all the environment variables to the log, so you can have a look through to see what is actually available for you to reference.

## Setting up a pipeline.
The easiest way to set up a build pipeline on Azure DevOps from GitHub is to add the [Azure Pipelines](https://github.com/marketplace/azure-pipelines) GitHub App to your Github account. When you connect it to a repository, it will walk you through setting up a default pipeline; just choose the "empty" option. This pipeline will save a YAML file into your repository and will set up two triggers. One is a simple trigger to run the pipeline for any commit on any branch, the other is a specific integration into Pull Requests; essentially any pull request will be run through the pipeline and if it fails it will block the PR from being merged. 

This is the YAML file I ended up:

```yaml
# This only runs for master and develop. Plus a seperate trigger is run for PR validation. This means commits to branches not in a PR won't get tested. Choices, choices...
name: NewOrbit.NewOrbit.AddOne - build and test
trigger:
  - master
  - develop

variables:
  buildConfiguration: Release

pool:
  vmImage: 'vs2017-win2016'

steps:

- script: set
  displayName: show variables

- script: dotnet restore
  displayName: dotnet restore

- script: dotnet build --configuration $(buildConfiguration) --no-restore
  displayName: build

- task: DotNetCoreCLI@2
  displayName: test
  inputs:
    command: test
    projects: '**/*tests/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

- script: dotnet pack --configuration $(buildConfiguration) --no-build --output %Build_ArtifactStagingDirectory%
  condition: and(succeeded(), or(eq(variables['Build.SourceBranchName'], 'master'),eq(variables['Build.SourceBranchName'], 'develop')))
  displayName: pack

- task: NuGetCommand@2
  displayName: publish
  condition: and(succeeded(), or(eq(variables['Build.SourceBranchName'], 'master'),eq(variables['Build.SourceBranchName'], 'develop')))
  inputs:
    command: push
    nuGetFeedType: external
    publishFeedCredentials: 'NewOrbit MyGet Nuget'
    packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
```

The `trigger` part limits this to only run on checkins to `develop` and `master` (note, some of the documentation has a more verbose syntax that seems to not work). The Pull Request trigger still works so all Pull Request and all commits into an open Pull Request will be run through this pipeline. But, for me, I don't need CI to run on every commit on every feature branch. That's just me - if you want the pipeline to run for every commit, just delete the `trigger` section altogether.

The steps through restore and build should be obvious. The step after that uses a special Azure DevOps task to run the unit tests, which ensures that the results are reported in a nice way in the pipeline.

The `script: dotnet pack` packs the Nuget package and outputs the package to a particular holding area. To be honest, I could probably forgo the `output` parameter but it helps to understand what is going on.  
The key thing here is the `condition` line. This will ensure that a Nuget package is *only* created if the build is of either the develop or the master branch. If you wanted to publish "beta" versions from `release/*` branches, it should be straight forward to extend the condition accordingly.  
Incidentally, there is an Azure DevOps task for creating the Nuget package but I couldn't get it to work so used `dotnet pack` instead.

The final task publishes the created nuget package to Nuget. In this case I am publishing it to Myget; In order to do this, you first need to go to your *project* in Azure DevOps, go to Project Settings and then select Service Connections (it's well hidden). Then add a connection to Nuget or MyGet or whatever Nuget feed you want to publish to. You put the *name* of that service connection in the `publishFeedCredentials` property in the YAML file. 

if you wanted to publish packages from your develop branch to MyGet and the ones from Master to NuGet you can hopefully see how you can just duplicate the last task and change the `condition` statements to suit your needs.

**NOTE** There is a bug in Azure DevOps that may result in an error saying something like that your pipeline doesn't have the right permissiom to use the service connection. It's easy to fix by following the guidance [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=vsts).

## Approvals
The approach described above will publish packages immediately. If you wanted, you can easily set it up so you have to manually approve the publish. In short, you need to replace the final publish task in the YAML above with a Publish Artifacts task. 

```yaml
- task: PublishBuildArtifacts@1
  inputs:
    artifactName: 'package' 
```

This will copy whatever is in the `%Build_ArtifactStagingDirectory%` directory (where we put the Nuget package before) and make it available as an artefact of the build. Once you run the pipeline, look at the build and you will see an Artefact. If you click on that, Azure DevOps will take you through a wizard to set up a release pipeline, which you can then use to add manual approval before you publish the package to Nuget.