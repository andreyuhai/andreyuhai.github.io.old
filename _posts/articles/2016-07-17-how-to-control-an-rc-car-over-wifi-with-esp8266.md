---
layout: post
title: "How to control an RC car over WiFi with ESP8266"
excerpt: ESP8266 is a low-cost WiFi chip with a full TCP/IP stack and a microcontroller capability. Here is how I used it to control an RC car.
modified: 2016-07-17 14:48:29 +0300
categories: articles
tags: [wifi, arduino, esp8266, IoT, hardware, electronics, python]
image:
  feature: 2016-07-11-how-to-control-an-rc-car-over-wifi/cover.jpg
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Some time ago I bought my first Arduino. I have several years of experience as a software developer but my knowledge of electronics is pretty minimal. For my first project I decided to [control a cheap RC car with an Arduino board]({{ site.url }}/articles/controlling-an-rc-car-with-arduino/ "Control an RC car with an Arduino"). This was done by connecting the board to the car's controller and sending commands from my laptop to the Arduino over a serial connection. Basically I made the original controller programmable and the car would still receive the signal over 27MHz radio waves. It was a relatively easy project to start with because I did not have to open up the car itself. The downside is that every time I want to take the car for a spin, I have to connect my laptop to my Arduino and then connect that to the controller. This post is about my next project where I started to control the RC car over WiFi.

## Ditching the controller

To make the setup easier, I decided to get rid of the original transmitter. My plan was not to have any extra wires dangling from my laptop and use the laptop itself as a transmitter. That left me with 2 possibilities - Bluetooth or WiFi. I decided to choose the latter because I wanted to try and work with the ESP8266 chip. This is a low-cost WiFi chip with a full TCP/IP stack and a microcontroller capability.

## Why use the laptop as a transmitter?

To be honest, it does not have to be a laptop. After all, holding a laptop and controlling the car at the same time is a bit uncomfortable. But if I can make the RC car controllable programmatically, I could use other WiFi connected devices as well (e.g. smartphone). For the initial prototype I feel it is reasonable to test the connection with a laptop.

## ESP8266 chip

They come in different shapes and sizes. For my project I went with the [SparkFun ESP8266 Thing
](https://www.sparkfun.com/products/13231 "SparkFun ESP8266 Thing product page") because the pins are broken out to two parallel, breadboard-compatible rows and it comes with a lithium polymer battery which is useful since the RC car is not plugged in to a power source. The Thing can be even programmed using the Arduino IDE by installing the [ESP8266 Arduino addon](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon "ESP8266 arduino addon").

## Initial plan

Overall, the setup should be straightforward. The ESP8266 will be placed inside the car. Both the board and my laptop will be connected to the same WiFi network so they can communicate freely with each other. Pressing arrow keys (or WASD keys) will trigger the car to either steer or move.


## Getting started

As mentioned previously, in my [first project]({{ site.url }}/articles/controlling-an-rc-car-with-arduino/ "Control an RC car with an Arduino") I did not open up the car. Obviously, if I want to start using the ESP8266 chip, I'm going to have to place it inside the car. So off I went to find a screwdriver.

Having opened the car I found that it uses the standard RX-2/TX-2 chipset used in many cheap RC toys. Additionally the car came with two DC motors for moving and steering and LED head/taillights. After removing the original printed circuit board inside the car, there was enough room to place the ESP8266 board in it.

<figure class="half">
  <a href="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" class="image-popup">
	  <img src="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/inside.jpg" alt="Inside of the RC car">
  </a>
  <a href="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" class="image-popup">
	  <img src="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/rx2tx2.jpg" alt="The chipset removed from the car">
  </a>
	<figcaption>A view of the inside of the car, RX-2/TX-2 chip removed</figcaption>
</figure>

## Controlling the motors

I can't just plug the power and ground wires to my ESP8266 and hope the DC motor starts working. Firstly, the chip can output only 3.3V but the motors require higher voltage. Secondly, I need to control the rotation direction of the motors which means reversing the polarity. For power, I'm going to use the original batteries that are mounted under the car but the ESP8266 chip will be powered by the LiPo battery included with the SparkFun ESP8266 Thing. To control the motors I decided to use a [Pololu DRV8833 Dual Motor Driver Carrier](https://www.pololu.com/product/2130 "DRV8833 Dual Motor Driver Carrier product page"). It allows me to change the rotation direction and as the name implies, it can handle two motors.

<figure>
	<a href="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/batteries.jpg" class="image-popup"><img src="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/batteries_thumb.jpg" alt="Batteries mounted under the car"></a>
	<figcaption>3 x 1.5V batteries mounted under the car power the DC motors</figcaption>
</figure>

## Setting up the motor driver

Reading the manual from Pololu helped me understand how to use the motor driver. They have great schematics and descriptions for each pin. The following is a wiring diagram provided by Pololu. Annotations in red are provided by me to describe some parts of the setup.

<figure>
	<a href="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/wiring_diagram.png" class="image-popup"><img src="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/wiring_diagram_thumb.png" alt="Pololu DRV8833 wiring diagram"></a>
	<figcaption><a href="https://www.pololu.com/product/2130" title="Pololu DRV8833 wiring diagram">Pololu DRV8833 wiring diagram</a></figcaption>
</figure>

Before taking out my soldering iron, I tested the motor driver on a breadboard to make sure I understood everything correctly. At that point I had not yet implemented the code which would eventually control the car. To test the driver I turned the control pins on the ESP8266 board high one by one to see if the motor started rotating. When I got that working I was ready to connect the motor driver with the motors in the car.

<figure class="half">
  <a href="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/breadboard_test.jpg" class="image-popup">
	  <img src="{{ site.url }}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/breadboard_test_thumb.jpg" alt="Testing the motor driver on a breadboard">
  </a>
  <a href="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/soldered_driver.jpg" class="image-popup">
	  <img src="/images/2016-07-11-how-to-control-an-rc-car-over-wifi/soldered_driver_thumb.jpg" alt="Motor driver connected with the car's DC motors">
  </a>
	<figcaption>Testing out the motor driver on a breadboard and connecting it with the car</figcaption>
</figure>

## Including other components

Finally, I added the ESP8266 controller to the car, connected ground and control wires with the motor driver and attached a LiPo battery. All the required components are in place. All that is left is to write the code that is going to transmit and receive commands over WiFi.

<figure>
	<a href="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/almost_ready.jpg" class="image-popup"><img src="{{ site.url}}/images/2016-07-11-how-to-control-an-rc-car-over-wifi/almost_ready_thumb.jpg" alt="Amost ready"></a>
	<figcaption>Attached a LiPo battery and a SparkFun ESP8266 Thing</figcaption>
</figure>

## Writing control logic

Debugging the SparkFun ESP8266 Thing is not as easy as a regular Arduino board. You cannot use the serial monitor provided by the Arduino IDE. [Luckily there is a simple solution for that]({{site.url}}/articles/using-a-serial-monitor-with-sparkfun-esp8266-thing/ "Using a serial monitor with SparkFun ESP8266 Thing").

I tested my initial ESP8266 code with **netcat**. For example, the following command sends a byte which corresponds to a *turn-right* command to the car. As you can see, the car has an IP address and a port it is listening on.

{% highlight bash %}
$ echo -e '\x03' | netcat 192.168.1.101 1111
{% endhighlight %}

Controlling the car with *netcat* is not fun. Therefore the next step was to implement a small Python program which would detect keyboard presses and send commands over the network to the car. I used the old Python program I wrote for [my previous project](https://github.com/indrekots/rc-car-controller) and [repurposed it](https://github.com/indrekots/esp8266-rc-car-controller/blob/master/wifi_controller_gui.py) so it could send commands over the network instead of over a serial connection.

You can have a look at [the source code of the project at Github](https://github.com/indrekots/esp8266-rc-car-controller "RC car controller source code").

## TCP vs UDP

During the first tests I discovered that the car was not very responsive. The car would continue driving even when I had released all keys for example. I found that this was due to the fact that the communication was done over [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol "TCP Wikipedia page"). I'm guessing the TCP handshake process and acknowledgements introduce some overhead. After I had reengineered the car to use UDP, everything worked flawlessly.

## End result

The cheap RC car (~15EUR/~17USD) which initially had only 2 channels (although the RX-2/TX-2 chipset has a [third unused channel](http://www.instructables.com/id/Hack-an-RC-cars-unused-5th-channel/ "RX-2/TX-2 unused channel")) can now be controlled programmatically. Not to mention, the ESP8266 chip supports [pulse-width modulation (PWM)](https://en.wikipedia.org/wiki/Pulse-width_modulation "PWM Wikipedia page"). This means that I can control the driving speed of the car. Previously, because it had only 2 channels, it was either driving at full speed or not moving at all. Above all, since the car can be controlled over the network, I can come up with new ideas for future projects. Theoretically I could implement an app for a smartphone which would act as a custom controller.

I'm going to leave you with this video of the car in action.

<iframe width="560" height="315" src="https://www.youtube.com/embed/p0O8tkGuvoY" frameborder="0" allowfullscreen></iframe>
