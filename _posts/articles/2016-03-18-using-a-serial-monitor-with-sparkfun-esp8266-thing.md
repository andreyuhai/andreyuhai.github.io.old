---
layout: post
title: "Using a serial monitor with SparkFun ESP8266 Thing on Linux"
excerpt: Using Arduino IDE's serial monitor with SparkFun ESP8266 Thing does not work. In this post you'll see how to use a serial terminal with the SparkFun ESP8266 Thing on Linux.
modified: 2016-06-30 12:17:11 +0200
categories: articles
tags: [sparkfun, esp8266, IoT, linux, arduino, SoC, gtkterm, dtr]
image:
  feature: 2016-03-15-using-a-serial-monitor-with-sparkfun-esp8266-thing/cover.jpg
  credit: www.openhomeautomation.net
  creditlink: https://www.openhomeautomation.net/wp-content/uploads/2015/07/thing.jpg
comments: true
share: true
published: true
aging: true
---

[ESP8266](http://www.esp8266.com/ "esp8266 community") is a low-cost system on a chip with integrated TCP/IP stack and WIFI. The SparkFun ESP8266 Thing is a breakout and a development board for for the ESP8266 chip that makes it easier to use and provides some additional features.

If you have experience programming Arduino boards, then most likely you have used the serial monitor that's available in the Arduino IDE to debug your code. It is possible to use the Arduino IDE to program the ESP8266 chip provided that you install the required add-on. Unfortunately using the serial monitor in the IDE is not going to work with the SparkFun Thing.

## Why doesn't the serial monitor work?

[SparkFun has a better explanation on it on their website](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/using-the-arduino-addon#serial-dtr "SparkFun ESP8266 Thing serial monitor issues") but in short to make it easy to program the ESP8266, they have tied GPIO0 to DTR (along with RST). When your programmer begins to upload a sketch, it’ll pull DTR low, in turn setting GPIO0 low and making the ESP8266 enter bootloader mode. Unfortunately, when you open a serial terminal, DTR usually goes low again. So every time you open the Arduino serial monitor, it’ll cause the ESP8266 to enter bootloader mode, instead of run-program mode and you're going to see a line of text which does not make any sense at all.

![Line of gibberish]({{ site.url }}/images/2016-03-15-using-a-serial-monitor-with-sparkfun-esp8266-thing/gibberish.png "Line of gibberish on the serial monitor")

## How to solve this issue?

There's two solutions you could use. The first one involves modifying the hardware and the second one is done only using software. We'll be looking the latter one because it's easier.

To use a serial monitor with SparkFun ESP8266 Thing on Linux you need to find an alternative serial terminal which allows to control the DTR pin directly. On [SparkFun's website](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/using-the-arduino-addon#serial-dtr "SparkFun ESP8266 Thing serial monitor issues") they have pointed out that you can use [RealTerm](http://realterm.sourceforge.net/) but unfortnunately this is for Windows only. After having done some research I installed [gtkterm](https://fedorahosted.org/gtkterm/ "gtkterm website") and it can do the job pretty well.

Open Gtkterm and configure it.

![Configuration menu]({{ site.url }}/images/2016-03-15-using-a-serial-monitor-with-sparkfun-esp8266-thing/configuration.png "Gtkterm configuration menu")

Toggle DTR with F7 or from the control signals menu. After that you can see the output printed by ESP8266.

![Configuration menu]({{ site.url }}/images/2016-03-15-using-a-serial-monitor-with-sparkfun-esp8266-thing/serial_terminal.png "Serial output")

Now you should be able to use a serial monitor with SparkFun ESP8266 Thing on Linux and you don't have to debug your code with LEDs.

## OS X users

Reader [**Kevin Hoyt**](http://www.kevinhoyt.com/) pointed out in the comments that on OS X he used [CoolTerm](http://freeware.the-meiers.org/ "CoolTerm homepage") to solve the same issue.

<hr/>

**Edit 30.06.2016:** added solution for OS X users pointed out by Kevin Hoyt in the comments
