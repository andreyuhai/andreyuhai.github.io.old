---
layout: post
title: "Wi-Fi Controlled RC Car Using ESP8266-01"
excerpt: In this post I will be documenting my project about a Wi-Fi controlled RC car using WebSocket protocol with ESP8266-01.
modified: 2018-07-10 19:22:52 +0300
categories: articles
tags: [arduino, ESP8266, IoT, WebSockets]
image:
  feature: 2018-06-19-wifi-controlled-rc-car-using-esp8266/cover.jpg
  credit: Spencer_
  creditlink: https://unsplash.com/photos/lbqLxgvLt0U
comments: true
share: true
published: true
aging: true
---
#### Contents

1. [Components](#what-are-the-components-ive-used-for-the-project)
2. [Wiring Diagram](#wiring-diagram)
3. [WebSocket Protocol](#what-is-websocket-protocol)
4. [Creating the JavaScript File](#creating-the-javascript-file)
5. [Creating the HTML File](#creating-the-html-file)
6. [Programming the ESP8266-01](#programming-the-esp8266-01)
7. [Programming the Arduino UNO](#programming-the-arduino)
8. [References](#references)

---

Here I will be documenting my Arduino project about a Wi-Fi controlled (ESP8266-01) RC car using WebSocket protocol that I've been working on literally for months. Actually not have been working on but instead have had in mind for months would be more appropriate to say since I didn't really have time to completely focus on it so far and moreover with a tight budget you can't really buy everything you need at once. Also a misplaced capacitor between the 3.3V regulator and ESP8266 took me three days to figure out why my ESP8266 was not working properly but in the meantime I'd already ordered a new NodeMCU one. **facepalm*\*

#### What are the components I've used for the project?

* An Arduino UNO (obviously)
* An ESP8266-01
* An L298N motor driver
* An LF33CV 3.3V regulator
* An RC car
* Also batteries that meet your needs

#### Wiring Diagram

![Wifi controlled RC car wiring diagram][wiring diagram]

**Note:** *Since I don't have a proper battery -thanks to LiPo batteries and their even more expensive chargers-, I've used 3 9V batteries to source the circuit just to test it, hence I am still having problems driving the car but this doesn't mean any problems with the code and etc. I will edit here when I get a neat battery to test the car and also record a video of it.*

#### What is WebSocket Protocol?

>WebSockets are a bi-directional, full-duplex, persistent connection from a web browser to a server. Once a WebSocket connection is established the connection stays open until the client or server decides to close this connection. With this open connection, the client or server can send a message at any given time to the other.Introduction to WebSockets.<sup>[1][1]</sup>

#### How is all this going to be working?

After making all the wire connections we are going to create a small website where we will be checking for keystrokes to control the RC car using our arrow keys. We will be sending those keystrokes as commands using JavaScript. 

Actually I would like to control my RC car using my gamepad and adjust the car's speed using its analogue sticks with a code written in C or C++ using the same protocol but somehow my gamepad stopped working. Therefore, until I get a new one I will be using arrow keys and a constant speed for the motor. Of course once I get the gamepad and have time for it I will also try controlling the car with the gamepad and with other protocols like HTTP or UDP to see the difference, if any.

About how I send the commands exactly, I just send a two digit number representing the motion of the car and also the direction of its wheels every time an arrow key is pressed and released. When the key is released we will be sending commands to stop whatever motion was triggered by the released key. **The tens digit of the number represents the steering wheel**, e.g.:

1: *Left*,

2: *Straight*,

3: *Right*

**Ones digit of the number will be representing the movement of the car and its direction**, e.g.:

1: *Backwards*,

2: *Stop*,

3: *Forwards*

So for example, 21 would represent "*go straight backwards*" in this case and 13 would represent "*go left forwards*".

#### Creating the JavaScript File

Here I will be explaining the codes in my `app.js` file.
```javascript
var movement = 2; // 1: BACKWARDS, 2: STOP 3: FORWARDS
var wheelDirection = 1; // 0: LEFT, 1: STRAIGHT, 2: RIGHT
```
Above we are declaring our variables which we will be assigning different values according to keystrokes we've read and then send them to our Arduino using WebSocket Protocol.

---
```javascript
document.onkeydown = checkKeyDown; //the function which is invoked on keydown
document.onkeyup = checkKeyUp; //the function which is invoked on keyup
```
Defining our keydown and keyup functions.

---

```javascript
//Creating a new connection
var connection = new WebSocket('ws://192.168.4.1:81/', ['arduino']);

connection.onopen = function () {
  connection.send('Connect ' + new Date());
};
connection.onerror = function (error) {
  console.log('WebSocket Error ', error);
};
connection.onmessage = function (e) {
  console.log('Server: ', e.data);
};
connection.onclose = function () {
  console.log('WebSocket connection closed');
};
```
We are going to connect to the server at  `ws://192.168.4.1` on `the port 81` with a protocol named `arduino`.

On successful connection, we send a string to the server including the connection date.

On error, we log error to the console.

On lost connection or close, `WebSocket connection closed.` logged to the console.

On message from the server we again log it to the console.

---

```javascript
function checkKeyDown(e) { //When the key is pressed

    e = e || window.event;

    if (e.keyCode == '38') {
      // up arrow
      movement = 2;
      console.log("forward");
    }
    else if (e.keyCode == '40') {
        // down arrow
        movement = 0;
        console.log("down");
    }
    else if (e.keyCode == '37') {
       // left arrow
       wheelDirection = 1;
       console.log("left");
    }
    else if (e.keyCode == '39') {
       // right arrow
       wheelDirection = 3;
       console.log("right");
    }
    var sum = (wheelDirection*10)+movement;
    connection.send(sum.toString());
    console.log("sum: "+sum);
}
```
Here we are defining our `checkKeyDown` function. What we are doing is simply checking the keystrokes whether they are one of the arrow keys, if so, we are assigning the variables the above specified values accordingly.

At the end we are summing the two values `wheelDirection` and `movement` then we are sending the sum to the server and also logging it to the console. 

---
```javascript
function checkKeyUp(e) { //When the key is released

    e = e || window.event;

    if (e.keyCode == '38') {
      // up arrow
      movement = 1;
      console.log("stop forward");      
    }
    else if (e.keyCode == '40') {
        // down arrow
        movement = 1;
        console.log("stop backwards");
    }
    else if (e.keyCode == '37') {
       // left arrow
       wheelDirection = 2;
       console.log("stop left");
    }
    else if (e.keyCode == '39') {
       // right arrow
       wheelDirection = 2;
       console.log("stop right");
    }
    var sum = (wheelDirection*10)+movement; //Summing the commands as mentioned.
    connection.send(sum.toString()); //Sending the number or command so to speak.
    console.log("sum: "+sum);
}
```
This is the same process as `checkKeyDown` function. The only difference is that this function sends stop signals after a key is released.

#### Complete JavaScript File

```javascript
//app.js
document.onkeydown = checkKeyDown; //the function which is invoked on keydown
document.onkeyup = checkKeyUp; //the function which is invoked on keyup

//Creating a new connection
var connection = new WebSocket('ws://192.168.4.1:81/', ['arduino']);

connection.onopen = function () {
  connection.send('Connect ' + new Date());
};
connection.onerror = function (error) {
  console.log('WebSocket Error ', error);
};
connection.onmessage = function (e) {
  console.log('Server: ', e.data);
};
connection.onclose = function () {
  console.log('WebSocket connection closed');
};

var movement = 2; // 1: BACKWARDS, 2: STOP 3: FORWARDS
var wheelDirection = 1; // 0: LEFT, 1: STRAIGHT, 2: RIGHT

function checkKeyDown(e) { //When the key is pressed

    e = e || window.event;

    if (e.keyCode == '38') {
      // up arrow
      movement = 2;
      console.log("forward");
    }
    else if (e.keyCode == '40') {
        // down arrow
        movement = 0;
        console.log("down");
    }
    else if (e.keyCode == '37') {
       // left arrow
       wheelDirection = 1;
       console.log("left");
    }
    else if (e.keyCode == '39') {
       // right arrow
       wheelDirection = 3;
       console.log("right");
    }
    var sum = (wheelDirection*10)+movement;
    connection.send(sum.toString());
    console.log("sum: "+sum);
}

function checkKeyUp(e) { //When the key is released

    e = e || window.event;

    if (e.keyCode == '38') {
      // up arrow
      movement = 1;
      console.log("stop forward");      
    }
    else if (e.keyCode == '40') {
        // down arrow
        movement = 1;
        console.log("stop backwards");
    }
    else if (e.keyCode == '37') {
       // left arrow
       wheelDirection = 2;
       console.log("stop left");
    }
    else if (e.keyCode == '39') {
       // right arrow
       wheelDirection = 2;
       console.log("stop right");
    }
    var sum = (wheelDirection*10)+movement; //Summing the commands as mentioned.
    connection.send(sum.toString()); //Sending the number or command so to speak.
    console.log("sum: "+sum);
}
```

#### Creating the HTML File

Creating the index.html is quite easy since we won't need any content. Our `app.js` will do the work, hence we only need to include it with `<script>` tags.

```html
<!--- index.html --->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Motor Controller</title>
</head>
<body>

  <!-- Required for the application to work. -->
  <script type="text/javascript" src="app.js"></script>
</body>
</html>
```

#### Programming the ESP8266-01
We are going to program the ESP8266 using libraries<sup>[2][2]</sup> on GitHub.
```cpp
#include <SoftwareSerial.h>

#define BACKWARDS 0
#define STOP 1
#define FORWARDS 2
#define LEFT 1
#define STRAIGHT 2
#define RIGHT 3


SoftwareSerial esp8266(2, 3); //Rx, Tx

int lastSpeed = 21;
int info[2] = {0}; // info[0] = wheelDirection, info[1] = movement
int movement;
char c;
int i;

void setup()
{
  Serial.begin(115200);  //For Serial monitor
  esp8266.begin(115200); //ESP Baud rate
  /* F R O N T  W H E E L S'  D E C L A R A T I O N */
  pinMode(13, OUTPUT);
  pinMode(12, OUTPUT);
  pinMode(9, OUTPUT);

  /* R E A R  W H E E L S'  D E C L A R A T I O N */
  pinMode(7, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(5, OUTPUT);

  /* F R O N T  W H E E L S'  A S S I G N M E N T */
  analogWrite(9, 0);
  digitalWrite(12, LOW);
  digitalWrite(13, HIGH);

  /* R E A R  W H E E L S'  A S S I G N M E N T*/
  digitalWrite(7, HIGH);
  digitalWrite(6, LOW);
  analogWrite(5, 0);
}

void setSpeed(int newSpeed) {

    int temp = newSpeed;
    i = 0;
    while (temp) {
      info[i] = temp % 10;
      temp /= 10;
      i++;
    }
    switch (info[1]) {
      case LEFT:
        analogWrite(9, 150);
        digitalWrite(12, HIGH);
        digitalWrite(13, LOW);
        break;
      case RIGHT:
        analogWrite(9, 150);
        digitalWrite(12, LOW);
        digitalWrite(13, HIGH);
        break;
      case STRAIGHT:
        analogWrite(9, 0);
        break;
    }
    switch (info[0]) {
      case BACKWARDS:
        analogWrite(5, 60);
        digitalWrite(6, HIGH);
        digitalWrite(7, LOW);
        break;
      case FORWARDS:
        analogWrite(5, 60);
        digitalWrite(6, LOW);
        digitalWrite(7, HIGH);
        break;
      case STOP:
        analogWrite(5, 0);
        break;
    }
    lastSpeed = newSpeed;
}


void loop()
{

  if (esp8266.available() > 0) {
    char c = esp8266.read();
    if (c == ':') {
      if (!isdigit(esp8266.peek())) {
        Serial.println("Skipped");
      } else {
        movement = esp8266.parseInt();
        
        setSpeed(movement);
      }
      while (esp8266.read() != -1) {
        //DO NOTHING - TO CLEAR BUFFER
      }
    }
  }
}
```
---
#### Programming the Arduino
We are going to use the arduinoWebSockets library by Links2004. Download the library from [GitHub](https://github.com/Links2004/arduinoWebSockets) and install it. (Sketch > Include Library > Add .ZIP Library...)<sup>[3][3]</sup>
```cpp
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <WebSocketsServer.h>

WebSocketsServer webSocket(81);
const char *ssid = "softAPI";  // You will connect your phone to this Access Point
const char *pw = "pass123"; // and this is the password
IPAddress ip(192,168,4,1);
IPAddress netmask(255,255,255,0);

void startWebSocket() { // Start a WebSocket server
  webSocket.begin();                          // start the websocket server
  webSocket.onEvent(webSocketEvent);          // if there's an incomming websocket message, go to function 'webSocketEvent'
  Serial.println("WebSocket server started.");
}

void webSocketEvent(uint8_t num, WStype_t type, uint8_t * payload, size_t lenght) { // When a WebSocket message is received
  switch (type) {
    case WStype_DISCONNECTED:             // if the websocket is disconnected
      Serial.printf("[%u] Disconnected!\n", num);
      break;
    case WStype_CONNECTED: {              // if a new websocket connection is established
        IPAddress ip = webSocket.remoteIP(num);
        if( payload != NULL){
          Serial.printf("[%u] Connected from %d.%d.%d.%d url: %s\n", num, ip[0], ip[1], ip[2], ip[3], payload);  
        }        
      }
      break;
    case WStype_TEXT:                     // if new text data is received,
    if(payload == NULL){
      Serial.println("Nothing useful");
    }else{
      Serial.printf("[%u] speed:%s\n", num, payload);
    }
    break;
  }
}

void setup() {
  Serial.begin(115200);
  
  delay(1000);
  
  WiFi.softAPConfig(ip, ip, netmask);
  WiFi.mode(WIFI_AP);
  WiFi.softAP(ssid, pw); // configure ssid and password for softAP
  startWebSocket();  
  Serial.printf("Soft AP created: %s",WiFi.softAPIP().toString().c_str());
}


void loop() {
  webSocket.loop();  
}
```
**_I will be updating this post adding more detailed explanations and photos from the project when I have more time to focus on this post. Till then, you can contact me via email for any question about the project._**
 
## References
1. [Introduction to WebSockets] 
2. [ESP8266 core for Arduino]
3. [WebSocket communication by tttapa] (*I really recommend reading this guide to anyone who is interested in IoT or simply ESP8266*)

[1]:#references
[introduction to websockets]: http://socketo.me/docs/
[2]:#references
[esp8266 core for arduino]:https://github.com/esp8266/Arduino
[3]:#references
[WebSocket communication by tttapa]:https://tttapa.github.io/ESP8266/Chap14%20-%20WebSocket.html

[wiring diagram]: ../../images/wifi_controlled_rc_car.png 
