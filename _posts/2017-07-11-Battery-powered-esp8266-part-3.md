---
layout: post
title:  Battery powered esp8266 - part 3
date: '2017-07-11'
tags: iot
---


This is part 3 of the series dedicated to learning about esp8266 building a wifi thermometer. Included: prototype source code and conclusion.

**UPDATE** 2017-08-25 Inserted lua code in the article

## The outcome

Wifi thermometer:

    - Measures temperature 
    - Runs on battery and has decent autonomy 
    - Collects data and stores it in the cloud 
    - Web application to plot the temperature readings 
    - Can run in partially connected environment

TODO: insert a pic of the device

## The device code

Features:

 - Switch enabled autorun (see init.lua). 
 - Use of modules 
 - Experimental rtcmem local storage for storing reading when wifi is not available 


The modules

 - init.lua - hook to autoexecute code after a restart
 - firmware.lua - the main module, executed from init
 - comm.lua - http communication for sending sensor data to a server
 - sensor.lua - reading the sensor (ds18b20 temperature sensor)
 - storage.lua - experimental rtcfifo based storage for when wifi connection is not available 
 - timer.lua - timer functions

**init.lua** 
```lua 
autorunPin = 1
gpio.mode(autorunPin, gpio.INPUT,gpio.PULLUP)

if (gpio.read(autorunPin) == gpio.LOW) then 
    dofile("firmware.lua") 
end
```
**firmware.lua**
```lua
timer = require "timer"
comm = require "comm"
storage = require "storage"
sensor = require "sensor"

comm.init()  
storage.init()

local secondsWaited = 0
local secondsToWait = 10
local timedOut = false 

timer.registerAutoAlarm(function() 

    print('wait')
    timedOut = secondsWaited >= secondsToWait

    if(not timedOut and not comm.ready()) then
        secondsWaited = secondsWaited + 1
        return
    end
         
   timer.unregisterAlarm()

    if(not comm.ready()) then
       print('no comm')
       storage.push(sensor.read())
       timer.sleep()
    end

    local syncTime = true 
        
    timer.registerSemiAutoAlarm( function() 

        if(syncTime) then
            comm.readTime(function(code, timestamp)               
               if(comm.sendOk(code)) then                
                    timer.sync(timestamp) 
                end
                syncTime = false
                timer.triggerAlarm()
            end)
            return  
        end  
        
       storage.push(sensor.read())
   
       local storedValues = {}
       local count = 0
       storedValues,count = storage.readAll()
           
       comm.send(storedValues,count, function(code, data) 
            if(comm.sendOk(code)) then                
                storage.clear()            
            end
            timer.sleep()            
        end)       
    end)
     
end)
```

**comm.lua**

```lua
local comm = {}

local wifiName = '<add your own>'
local wifiPass = '<add your own>'
local deviceId = '927afe47-9b07-46af-8b5a-7fa92ab6f7ef'     

function comm.ready()
    return wifi.sta.getip() ~= nil;
end
 
function comm.init()
  
    wifi.setmode(wifi.STATION)
    wifi.setphymode(wifi.PHYMODE_N)
    wifi.sta.config(wifiName,wifiPass)
    wifi.sta.connect()
 
end

function comm.readTime(callback)

    http.get('http://hub9.azurewebsites.net/api/time',    
        'Authorization: Basic <add your own>\r\n',       
        function(code, data) 
            callback(code, tonumber(data))            
        end)    
end

function comm.send(values, count, callback)
    local i
    local payload = {}
    
    for i = 1, count do
       payload[i] = '{"sensor":"temp", "value":"' ..values[i]..'"}'
    end

    local payloadAll = table.concat(payload,',')
   -- print(payloadAll)

    http.post('http://hub9.azurewebsites.net/api/devices/'..deviceId..'/logs',    
        'Content-Type: application/json\r\n'..
        'Authorization: Basic <add your own>\r\n',
        '['..payloadAll..']',
        function(code, data) callback(code) end)
    
end

function comm.sendOk(code)
    return code >= 200 and code < 300
end

return comm
```

**Sensor.lua**

Prety much the same thing you'll find in the nodemcu documentation

```lua
local sensor = {}

local pin = 2

function sensor.read()
    
    ow.setup(pin)   
    local count = 0
    local addr = nil
    repeat
        count = count + 1
        addr = ow.reset_search(pin)
        addr = ow.search(pin)
        tmr.wdclr()
    until (addr ~= nil) or (count > 100)
    
    if addr == nil then      
        return nil
    end  
    
    local crc = ow.crc8(string.sub(addr,1,7))
    if crc ~= addr:byte(8) then        
        return nil 
    end
    
    if (addr:byte(1) ~= 0x10) and (addr:byte(1) ~= 0x28) then       
       return nil
    end 
    
    ow.reset(pin)
    ow.select(pin, addr)
    ow.write(pin, 0x44, 1)
    tmr.delay(1000000)
    local  present = ow.reset(pin)
    ow.select(pin, addr)
    ow.write(pin,0xBE,1)
    local data = string.char(ow.read(pin))
    
    for i = 1, 8 do
        data = data .. string.char(ow.read(pin))
    end
      
    crc = ow.crc8(string.sub(data,1,8))
    if crc ~= data:byte(9) then
       return nil
    end

    local t = (data:byte(1) + data:byte(2) * 256) * 625
    t = t / 10000     
    if t > 4094 then
        t = 4094 - t
    end
   
    ow.depower(pin)
    return t
end
    
return sensor
```

**storage.lua**

```lua
local storage = {}

function storage.init()
   if rtcfifo.ready() == 0 then
        print('init storage')
        rtcfifo.prepare()
   end 
end

function storage.ready()  
    return rtcfifo.ready() ~= 0
end

function storage.empty()     
    return rtcfifo.count() == 0
end

function storage.count()     
    return rtcfifo.count()
end

function storage.push(val,timestamp)
  rtcfifo.put(timestamp, val, 0, '')
end

function storage.peek()    
    local timestamp, value, neg_e, name = rtcfifo.peek()
    return adjust(value), timestamp
end

function storage.pop()    
    local timestamp, value, neg_e, name = rtcfifo.pop()
    return adjust(value), timestamp
end

function adjust(value)

    if value > 32767 then
        return value-65536
    end

    return value
end

return storage
```

**timer.lua**

```lua
local timer = {} 
 
function timer.sleep() 
   -- rtctime.dsleep(10000000) --10sec
   -- rtctime.dsleep(900000000) --15 min
    rtctime.dsleep(3600000000) --1h
end 

function timer.sync(timeStamp)   
   rtctime.set(timeStamp, 0)
end

function timer.now()
    return rtctime.get()
end 

function timer.registerAutoAlarm(callback)
    tmr.alarm(1, 1000, tmr.ALARM_AUTO, function() callback() end)
end

function timer.unregisterAlarm()
    tmr.stop(1)
end

function timer.registerSemiAutoAlarm(callback)
    tmr.alarm(1, 1, tmr.ALARM_SEMI, function() callback() end)
end

function timer.triggerAlarm()
    tmr.start(1)
end


return timer
```


## The server code

This bit I stitched together in one afternoon so it's not the best quality code but it works. It's a website that exposes a http rest api, built with asp.net core, google charts and runs in Microsoft Azure. 

Features: 

 - Receive sensor data over http
 - Store sensor data in sqlite database
 - Provide current time for the device
 - Plot a a graph of the reading over the past 7 days 

Code is [here](https://github.com/adam-gligor/Hub9)


## Conclusion

Initially I struggled to pick up the basics, that's because of the multitude of resource and different ways to program the device (lua, arduino) 

Once past the basic I discovered a very capable device backed by civilised and well behaved high level programming language.

I also appreciated the good documentation put together by the nodemcu firmware team [here](https://nodemcu.readthedocs.io/en/master/). 

Definitely will be using this device for more projects.

