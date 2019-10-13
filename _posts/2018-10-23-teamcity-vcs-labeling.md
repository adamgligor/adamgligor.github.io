---
layout: post
title: Confused about teamcity vcs labeling
date: '2018-10-23'
tags: programming
---

I'm confused about teamcity Vcs labeling feature ...

## Intro 

[TeamCity](https://www.jetbrains.com/teamcity/) has a [VCS Labeling](https://confluence.jetbrains.com/display/TCD18/VCS+Labeling) build feature. This is commonly used to apply version number to the changeset used for the build. 


I strougled a bit using this feature and found the original documentation fuzzy in some regards, here's what I learned.


These observations apply to TeamCity version 2017, and git source control.


## The screen 

First a couple of oddities, find these listed in a comment of the official docs page

- VcsLabeling action does not show up in the build log 
- It does show up up in the server log 
- Also shows show up in the build's *changes* tab 


Here's the default screen

![placeholder](/public/teamcity/vcs-labeling.png "teamcity vcslabeling")


## The Settings

VCS root to label - obvious, which vcs to label, usually there's only one.

Labeling pattern - obvious, the label to apply. It has parameter support.

Label builds in branches - not so obvious ... First thought this is the branch to label. Actually it defines the branches that are allowed to be labeled. The actual label is applied to the branch from where the build is executed. Values:

 - `+:<default>` - Only the default (usually master) branch is allowed to be lebeled. A label will be applied to the default branch when running a build from the default branch. Running a build from any other branch will not result in any labels being applied.

 - `+:*` - Any branch can be labeled. A label will be applied to the branch from which the build was executed.

 - `empty` (no value) same behavior as `+:*`


Let's see it in action 

Fig1. With the default settings no label is applied when running a build from any other branch

![placeholder](/public/teamcity/vcs-labeling-b.png "teamcity vcslabeling not ok")

Fig2. Use the `*` wildcard to allow any branch to be labeled.

![placeholder](/public/teamcity/vcs-labeling-a.png "teamcity vcslabeling ok")

