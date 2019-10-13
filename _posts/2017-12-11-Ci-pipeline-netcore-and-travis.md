---
layout: post
title:  CI piepeline with Travis CI for .Net Core and Azure hosting
date: '2017-12-11'
tags: programming
---


A CI pipeline for Azure hosted .Net Core apps using Travis CI ... 

## Intro

I recently set up a CI piepline using Travis CI for a personal project written in .net core. The process was frictionless and I'm sure that anyone can figure this out as I did, but still felt like droping a few lines here. 

Tools evolve at a fast pace, the things listed here were valid in december 2017, if you are reading long after please better head to the official documentation provided with for the tools. Travis CI docs are [here](https://docs.travis-ci.com/)


## Requirements 

My CI pipeline has the following features: Build the code and run the unit tests when changes are pushed to the master branch, same goes for pull requests. Additionally changes to the master branch get deployed into Azure.

### Building code and running unit tests 

The build step leverages the cli tool for .net core. Drop a configuration file `.travis.yml` in the root of you git repo and you're ready to go. 

Here is my config: 

```
language: csharp
mono: none
dotnet: 2.0.0
dist: trusty
sudo: required
solution: Hub9.Web.sln

install:
 - dotnet restore

script:
 - dotnet build --configuration Release
 - dotnet test Hub9.Tests/Hub9.Tests.csproj

```

This is self describing. The first section contains information about the language and tools. Next sections list the actions to be taken during the build. Build lifecycle is better described [here](https://docs.travis-ci.com/user/customizing-the-build/)

As first step the nuget packages are restored then the probject is built and finally tests are executed. NUnit and XUnit test runners are supported MsTest is not. Pull requests are built by default but the deployment step if any is skiped.

**1st Note** that in case the solution file is not in the root folder you need to `cd` into that folder first during the build. That command can be placed in the `script` section

**2nd Note** the `sudo` and `solution` keys might not be required. This works so I did not look further. 

### Deploy to Azure

One option here is to leverage the local git deployment provided by azure. Local git deployment means I can do something like `git push azure` and Azure will run the necessary steps to deploy the app.

Azure deployment in Travis is described [here](https://docs.travis-ci.com/user/deployment/azure-web-apps/) and setting up local git deploy in Azure is described [here](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-local-git)

Here is the config bit related to deploy 

```
deploy:
  provider: azure_web_apps
  verbose: true
  skip_cleanup: true
```

(The credentials are infered from the environment variables called AZURE_WA_USERNAME, AZURE_WA_PASSWORD, AZURE_WA_SITE)

**Note** on using special characters in the password field. 

I got stuck here and kept getting unauthorized errors from Azure. My Azure deployment credentials are stored as environment variables in Travis and I overlooked the documentation paragraph that states that special characters have to be escaped [here](https://docs.travis-ci.com/user/encryption-keys/#Note-on-escaping-certain-symbols)


### Github integration 

Works by default. You'll get a notification in the github ui when builds are running and the outcome. Extra you get a link with the status of the latest build to embedd in your project readme page if you wish. 


My final config file with everything in is this: 

```
language: csharp
mono: none
dotnet: 2.0.0
dist: trusty
sudo: required
solution: Hub9.Web.sln

install:
 - dotnet restore

script:
 - dotnet build --configuration Release
 - dotnet test Hub9.Tests/Hub9.Tests.csproj

deploy:
  provider: azure_web_apps
  verbose: true
  skip_cleanup: true
```
