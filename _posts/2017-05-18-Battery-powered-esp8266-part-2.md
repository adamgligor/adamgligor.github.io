---
layout: post
title:  Battery powered esp8266 - part 2
date: '2017-05-18'
tags: iot
draft: true
---

Running esp8266 on battery is definitely possible but not without challanges. Battery powered opereation is best suited for applications where long deep sleep cycles can be used.  

## Powering esp8266


Typlical power consumption with wifi on is in the range of a few hundred milliamps.

Deep sleep power consumption is in the range of a few hundred microamps.

Input voltage range is 1.7-3.6V



## Prototype #1 Barebone esp running on AA, NIMH

- provide eneough power 
- voltage converters , buck boost 
- instability, reset

.. not enough power from 1 battery second from converter


##Prototype #2 barebone esp runnig on Lipo 

- Lipo voltages, lipo changing 

Fail because bug in code, fast battery frain. Requires protectio circuit 


## Proptype #3 The winner 

Lithim battery with protection circuit (cutoff voltage in the right range) 
Lithim batter charger circuit 
Wemos nodemcu with onboard voltage regulator

## Not tested

Life battery and barebone esp


##How long can it last on one charge

200ua sleep current 
300ma normal operation (10 sec)
2500mah battery 
data aquisition frequency 1h 





## References

- Esp comunity forum [link](http://www.esp8266.com/)
- Nodemcu firmware home [link](http://nodemcu.com/index_en.html)