---
layout: post
title: Web apps with NancyFx and Sqlite
date: '2016-07-15'
tags: .net
---

Simple problems require simple solutions. I got the opportunity to build a small .Net web app using the less known web framework NancyFx, Sqlite for storage  and Ratchet library for the ui.

## The setup

A collaboration between the company and a local organization lead to me trying to teach a group of high school the nuts and bolts of developing web apps. The program spans over a period of about two months with weekly one hour sessions.    

## The frameworks

Developing web apps on .NET means most commonly using a flavor of ASP.NET and MSSQL. That's all fine but, given the starting level of knowledge and time constraints I wanted something more lightweight. Enter NancyFx ...


### NancyFx

NancyFx is a web framework inspired by the Sinatra, the web framework for Ruby.


NancyFx the spec sheet

- Lightweight & low ceremony
- Convention over configuration
- Built in dependency management
- Runs on .NET and Mono
- Runs (or will run) on .NET Core
- Supports Razor views


 NancyFx vs ASP.NET MVC at first glance.


 NancyFx as everything these days is distributed through Nuget. One of the first striking differences is what you end up with after setting up a new empty project. In case of Nancy that's three code files, that's all you need for a hello world app. Now granted that Nancy won't solve the same number of problems as ASP.NET without significant plumbing, extension libraries etc but if you are after low entry barrier that's an A+.  

### Sqlite

Every app needs persistence eventually. I wanted an embedded engine that did not require separate installation and worked with entity framework. LocalDb and SqlCompact are the offerings from Microsoft but Sqlite was better suited. It's very popular in the mobile devices but not that uncommon in low traffic websites. Best part: install a nuget package, job done you have a sql engine. The less good is that the database file and schema has to be created manually as the entity framework provider does not do data migrations yet, perhaps a future version.


Here's a good [podcast](https://changelog.com/201/) on Sqlite where it's author talks about the history and interesting facts.


Sqlite the spec sheet

 - No server process
 - Installed though Nuget
 - Has entity framework provider
 - Hugely popular
 - Official support in entity framework 7

### Ratchet

Bootstrap is what you'd normally use for web app these days. Ratchet is advertised as a prototyping tool for quickly building up mobile apps. What made me choose it, good documentation and visual examples for every component in the framework. Just copy paste the html fragments and arrange them to suit your needs.


Ratchet the spec sheet

- Ajax links via push.js
- Mobile(ish) look and feel
- Image gallery


## The app

Since this combination of frameworks was new to me too, I decided to do my homework before teach this to someone else, so I ended up building a prototype.

The theme: flash-cards ... it's a learning tool to help you memorize bits of information that can be written onto cards.

I won't go into details of the implementation, check out the [code](https://github.com/adam-gligor/archeology/tree/master/nancy-flash-master/nancy-flash-master) on github.

## Tips & things to watch out for

### Razor views

This doesn't work without a bit of sorcery, luckily the [Nancy docs](https://github.com/NancyFx/Nancy/wiki/Razor-View-Engine) to the rescue.

I had to do two things. First is specifying the assemblies and namespaces of the views in a special section in the web.config, second install two packages: Microsoft.AspNet.WebPages, Microsoft.Web.Infrastructure. Without these changes you'll get false compile errors that are annoying. Both issues are mentioned in the docs.  

### Sqlite connection string

Setting up the sqlite connection proved to be trickier than I anticipated. What happens is that Nancy has its own root path provider. To be able to find the database file and connect to it you'll have to hack the connection string from the program and use that path provider to create an absolute path, I haven't been able to make it work with relative paths.

### Azure deployment

A web app is a web app only when it's deployed and running on a server, this case azure. Small web apps can be hosted freely in azure. The Sqlite database is deployed alongside the web app, so nothing special has to be set up for that.    

## Links

- nancyfx [link](http://nancyfx.org/)
- ratchet [link](http://goratchet.com/)
- sqlite [link](https://www.sqlite.org/)
