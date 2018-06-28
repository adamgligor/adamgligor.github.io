---
layout: post
title: Weight scale with esp8266
date: '2018-06-28'
tags: iot
---


Here's how to make a cheap weight scale for your iot project.


## Intro

Part of one of the projects I'm envolved in this year was a digital scale that can send data over the internet.

I'll describe the details and prototype i've come up with next.  

## Hardware

**Esp8266 (wemos d1 mini)** This is a wifi enabled programmable board. 

![placeholder](/public/esp8266/wemos-d1mini.jpg "esp8266")


**50 kg load cell x 4** This a a half bridge resitive weight sensor. 

![placeholder](/public/esp8266/load-cell.jpg "load cell")


**hx711 module** This module contains the necessary components to make the connection to the esp possible (amplifier and adc converter)

![placeholder](/public/esp8266/hx711.jpg "hx711")

Everything can be sourced from aliexpress for a couple of bucks.

## Theory 

Information about this method of weight measurement and how to connect the compontents is extensively described on the internet. 

Basically I had to  connect the load cells in a wheatstone bridge, the hx711 to the bridge and then to the esp.


## Assembly 

These load cells I got are half bridge, meaning they contain two resistors. When under load one resistor will measure a positive change the other a negative change. 

The wiring on the cells I have is red to black is one resitor red to white is the other resistor. 

Made the bridge such as each arm has two resistors of the same type, either positive or negative. Then one pair of red wires is for applyng the input the other for reading the result.

![placeholder](/public/esp8266/load-cell-bridge.jpg "bridge")


To connect the esp and hx711 I use this diagramm.  Two pins D1 and D2 from on the esp are required to drive the hx711. 

![placeholder](/public/esp8266/esp-hx711.jpg "esp-hx711")

Everything working at 3.3V.

## Programming 

The nodemcu firmware for the esp8266 includes a module for operating the [hx711](https://nodemcu.readthedocs.io/en/master/en/modules/hx711/) 

Source code is three lines of code. 

```
hx711.init(2, 1) -- 
raw_data = hx711.read(0)
print(raw_data)
```

This will return a a number proportianal to the weight. It's actually the raw value of the adc conversion.

## Curve fitting 

To get to a mathematical formula, assume a linear relationship that can be aproximated with a first order function. 

Measure a few items with known weights and then use a curve fitting programm like [this](https://mycurvefit.com/) to find that function.


## Prototype 

Here's the prototype board 

![placeholder](/public/esp8266/weight-proto.jpg "proto")

## Links 

Sone articles with explanations I found useful 

- [article 1](https://electronics.stackexchange.com/questions/102164/3-wire-load-cells-and-wheatstone-bridges-from-a-bathroom-scale/199470)
- [article 2](https://arduino.stackexchange.com/questions/11946/how-to-get-weight-data-from-glass-electronic-bathroom-scale-sensors/18698)

