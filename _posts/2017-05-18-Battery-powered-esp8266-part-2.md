---
layout: post
title:  Battery powered esp8266 - part 2
date: '2017-05-18'
tags: iot
---

Running esp8266 on battery is definitely possible but not without challanges. Battery powered opereation is best suited for applications where long deep sleep cycles can be used.  
This is part two of the series on the esp8266.

## Powering esp8266

Here's some data on esp power consumption gathered from data sheets and articles: 

Input voltage has to be in the range of 1.7-3.6V

Typlical power consumption with wifi on is in the range of a few hundred milliamps.

Deep sleep power consumption is in the range of a few hundred microamps.

## Prototype #1. Esp running on AA or NIMH

First attempt was to power the esp with AA or NIMH batteries. Since there are a lot of references on using a pair of AA alcaline batteries I gave this a try.

Unfortunately I did not manage to get this to work since these batteries cannot reliably deliver 300-400ma current. 

Next option was to use more batteries and and use a voltage converter buck or boost. This might work but the voltage convertes characteristics also must be taken into account. Out of the two voltage convertes I bought one was faulty and another one was rated for a lower current.

Not enough power manifest itseft as random resets, communication failure or failure to properly boot.


## Prototype #2. Esp runnig on Lipo 

After more research I found lithium batteries as the next option. Lithium batheries come in different flavors and size and have the ability to deliver high currents, but pose new challanges for safe long term usage.

Types of lithium batteries 

**Lipo - Lithium polimer**

These are used in hobby rc, here's a typical single cell battery, rated at 3.7v, it does not have a protection circuit.

![lipo cell](/public/esp8266/lipo-cell.jpg)

**LiIon - Lithium ion** 

These are used in laptops and other consmer electronics, one of the more common forms is called 18650 and looks like this, rated at 3.6v it comes with or without protection circuit.

![18650](/public/esp8266/18650.jpg)

**LiFe - lithium iron phosphate**

The main characteristics of this type is the 3.2 v cell voltage, better stability but it's also less common.


To use lithium batteries safely one needs a specialized charging circuit or device, then for operation a protection circuit agains overcurrent and over discharge, acidental short circuit and such. While generally safe, unpropper use and handling can lead to explosion or fire.


Here's a picture of an early prototype. It's the wemos d1 mini hooked up to a lipo cell.

![proto1](/public/esp8266/proto1.jpg)

This was the first partial success. It worked for a few days, then failed because of a bug in code that prevented the device from going into deepsleep and thus draining the battery empty. I was planning to monitor the voltage and recharge it as needed, I did not forse this as an issues. It takes only a couple of over discharges to destroy a lipo battery.


## Proptype #3 The winner. Wemos d1 mini and 18650 LiIon battery with included protection circuit

Later on I found that some LiIon 1865 batteries come with integrated protection circuit, so I updated my prototype. This is what I have at the moment:

**The Charger**

This is the simplest possible charger. It's hooked up to a 5v supply at the one end (ex. phone charger) and to the battery at the other end. A status led idicates when the charge is complete.

![charger](/public/esp8266/charger.jpg)

**The prototype**

Again the wemos d1 mini, with diy arduino style shield hooked up to a LiIon battery with built in protection circuit this time.

![proto2](/public/esp8266/proto2.jpg)


## Estimating battery life ##

So for my setup I plan to read the sensors once every hour, and between the samples to put the esp in deep sleep. Next by measurement I know that:

 - 200 ua is the sleep current 
 - 300 ma is the normal operation current
 - I'll wait maximum 10 seconds to connect to the wifi read the sensors and send the data (typically takes 5 seconds)

Then under ideal conditions the power consumption for one cycle (sleep + wake) is: (10sec x 300mA + 3590sec x 0.2mA) / 3600sec ~= 0.83mA 

Thus a 2500mAh battery should last: 2500 / (0.83 x 24) ~= 120 days

Double checked with this calculator [link](http://oregonembedded.com/batterycalc.htm)

... source code, and practical evaluation to follow.

## References

- Esp comunity forum [link](http://www.esp8266.com/)
- Nodemcu firmware home [link](http://www.nodemcu.com/index_en.html)