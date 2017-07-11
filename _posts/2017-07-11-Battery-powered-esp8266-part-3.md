---
layout: post
title:  Battery powered esp8266 - part 3
date: '2017-07-11'
tags: iot
---


This is part 3 of the series dedicated to learning about esp8266 building a wifi thermometer. Included: prototype source code and conclusion.

## The outcome

Wifi thermometer:

    - Measures temperature 
    - Runs on battery and has decent autonomy 
    - Collects data and stores it in the cloud 
    - Web application to plot the temperature readings 
    - Can run in partially connected environment

TODO: insert a pic of the device

## The device code

The most advanced version of the nodemcu script I built is [here](https://github.com/adam-gligor/Hub9.Device.Thermometer/tree/master/v4)

Features:

 - Switch enabled autorun (see init.lua). 
 - Use of modules 
 - Experimental rtcmem local storage for storing reading when wifi is not available 

## The server code

This bit I stitched together in one afternoon so it's not the best quality code but it works. It's a website that uses asp.net core, google charts and runs in Microsoft Azure. Code is [here](https://github.com/adam-gligor/Hub9)

Features: 

 - Receive sensor data over http
 - Store sensor data in sqlite database
 - Provide current time for the device
 - Plot a a graph of the reading over the past 7 days 


## Conclusion

Initially I struggled to pick up the basics, that's because of the multitude of resource and different ways to program the device (lua, arduino) 

Once past the basic I discovered a very capable device backed by civilised and well behaved high level programming language.

I also appreciated the good documentation put together by the nodemcu firmware team [here](https://nodemcu.readthedocs.io/en/master/). 

Definitely will be using this device for more projects.

