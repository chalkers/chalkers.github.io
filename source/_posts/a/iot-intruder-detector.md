---
title: Internet of Things Intruder Detector
date: 28 Aug 2016
tags: 
  - Arduino
  - PIR Sensor
  - Internet of Things
  - Losant
  - ESP8266
---

## Overview

Want to receive a text if there's a bump in the night? Want an SMS if your cat is drinking from the toilet again? Want to get a text if a snooping parent or sibling comes in your room? This is a project for you!

We're going to use an ESP8266 based WiFi prototyping development board - the Adafruit Feather HUZZAH and IoT platform Losant to send an SMS when an intruder is detected.

## Bill of Materials

You'll need the following components for this project.

* [Breadboard](https://www.adafruit.com/products/64)
* [Adafruit Feather HUZZAH](https://www.adafruit.com/products/2821)
* [Jumper wires](https://www.adafruit.com/products/153)
* [Passive Infrared (PIR) Sensor](http://www.frys.com/product/6726705) 
* Micro-USB cable
* Any USB power supply

## Services Used

* [Losant](https://www.losant.com/)

## Wiring 

A Passive Infrared (PIR) sensor works by detecting the infrared radiation from an object - like a human or animal. 

Each PIR sensor can have different pin outs. I'm using the [Parallax 555-28027 PIR Sensor](https://www.parallax.com/sites/default/files/downloads/555-28027-PIR-Sensor-Prodcut-Doc-v2.2.pdf). If you face the sensor toward you the pins from left to right are, out ot signal, power and ground. `OUT` will send a `HIGH` signal once an infrared body is detected by the sensor. Here's a illustration of the sensor and pin out.

<img src='/attachments/iot-motion/pir.min.svg' alt='A graphic of the PIR sensor with the sensor facing toward the screen. At the bottom of the sensor there are 3 pins. Reading OUT, VCC and GND from left to right.' width="50%">

Here's what the Adafruit Feather HUZZAH looks like - big style!

<img src='/attachments/iot-motion/feather-huzzah.min.svg' alt='A graphic of the Adafruit Feather HUZZAH' width="80%">

Wire up the `VCC` on the PIR sensor to `3V` on the HUZZAH, the `GND` pin on the PIR sensor to the `GND` on the HUZZAH. The sensor should be working now. There's a red LED that comes on when the sensor detects a body. Finally connect the `OUT` from the PIR sensor to any digital pin on the HUZZAH. I picked `12`. 

<img src='/attachments/iot-motion/wiring-motion-sensor.min.svg' alt='A graphic of the wiring for the PIR sensor to the Feather HUZZAH showing a yellow wire going from OUT on the PIR to pin 12 on the HUZZAH, a red wire going from VCC on the PIR to 3V on the Huzzah and a black wire connecting the GND pins on the PIR and HUZZAH.' width="80%">

Here's how it looks <abbr title='In Real Life'>IRL</abbr>.

<img src='/attachments/iot-motion/irl.png' alt="A photo of a breadboard with a Adafruit Feather HUZZAH and PIR sensor connected. There's a white wire connecting power, black ground and yellow for OUT and 12.">


For further reading on the PIR sensor visit the [Parallex Website](https://www.parallax.com/sites/default/files/downloads/555-28027-PIR-Sensor-Prodcut-Doc-v2.2.pdf) or Adafruit's Learning System for the [Feather HUZZAH](https://learn.adafruit.com/adafruit-feather-huzzah-esp8266).

## Arduino Code

The Adafruit Feather HUZZAH is Arduino compatible, so we'll be using the [Arduino IDE](https://www.arduino.cc/en/Main/Software). If you're new to Arduino you should check out [The Absolute Beginner's Guide to Arduino](http://forefront.io/a/beginners-guide-to-arduino/).

The Arduino IDE doesn't come working "out-of-the-box" with the Feather HUZZAH. To get it up and running take a quick detour to [Adafruit's Learning Guide](https://learn.adafruit.com/adafruit-io-basics-esp8266-arduino/overview?view=all#using-arduino-ide). When the IDE has been set up, let's get ready to code.

Our Arduino code needs to do the following things:

1. Connect and reconnect to WiFi (`connect()` and `reconnect()`)
2. Connect and reconnect to the Losant IoT platform  (`connect()` and `reconnect()`)
3. Read the PIR sensor (`int currentRead = digitalRead(MOTION_PIN);`)
4. Send a message to Losant when motion of an infrared body has been detected (`motionDetected()`)

The Arduino code is below and requires the [Losant mqtt library for Arduino](https://github.com/Losant/losant-mqtt-arduino). See the [README.md](https://github.com/Losant/losant-mqtt-arduino/blob/master/README.md) for other dependencies.

```cpp
#include <ESP8266WiFi.h>
#include <Losant.h>

// WiFi credentials
const char* WIFI_SSID = "<YOUR SSID>";
const char* WIFI_PASS = "<YOUR PASSWORD>";

// Losant credentials
const char* LOSANT_DEVICE_ID = "<LOSANT_DEVICE_ID>";
const char* LOSANT_ACCESS_KEY = "<LOSANT ACCESS KEY>";
const char* LOSANT_ACCESS_SECRET = "<LOSANT ACCESS SECRET>";

const int MOTION_PIN = 12;

int motionState = 0;

WiFiClientSecure wifiClient;

LosantDevice device(LOSANT_DEVICE_ID);

// Connect to WiFi
void connect() {
  WiFi.begin(WIFI_SSID, WIFI_PASS);

  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  device.connectSecure(wifiClient, LOSANT_ACCESS_KEY, LOSANT_ACCESS_SECRET);

  unsigned long connectionStart = millis();
  while(!device.connected()) {
    delay(500);
    // If we can't connect after 5 seconds, restart the board.
    if(millis() - connectionStart > 5000) {
      // Failed to connect to Losant, restarting board.
      ESP.restart();
    }
  }

  //Device should be connected now and is now ready for use!
}

// Reconnects if required 
void reconnect() {
 bool toReconnect = false;

  // If the WiFi or HUZZAH is not connected to Losant - we should reconnect
  if(WiFi.status() != WL_CONNECTED || !device.connected()) {
    toReconnect = true;
  }

  if(toReconnect) {
    connect();
  }
}


void motionDetected() {
  // Losant uses a JSON protocol. Construct the simple state object.
  // { "motion" : true }
  StaticJsonBuffer<200> jsonBuffer;
  JsonObject& root = jsonBuffer.createObject();
  root["motion"] = true;

  // Send the state to Losant.
  device.sendState(root);
}

void setup() {
  pinMode(MOTION_PIN, INPUT);
  connect();
}

void loop() {
  reconnect();
  device.loop();

  // Read the sensor
  int currentRead = digitalRead(MOTION_PIN);

  // If motion is detected we don't want to 'spam' the service
  if(currentRead != motionState) {
    motionState = currentRead;
    if(motionState) {
      motionDetected();
    }
  }

  delay(100);
}
```
Fill in your SSID for `WIFI_SSID` and WiFi password for `WIFI_PASS`.

This only leaves the Losant credentials.

## Setting up Losant 

To connect to Losant you require 3 things:

1. A _Device ID_ to identify the device from another device (`LOSANT_DEVICE_ID`)
2. An access key (`LOSANT_ACCESS_KEY`)
3. An access secret (`LOSANT_ACCESS_SECRET`)

If you haven't signed up to Losant yet, do that now at [Losant.com](http://losant.com) - it's free for our use case.

First we need to create an application by going to **Applications** > **+ Create Application** from the drop down menus.

I'm calling mine **Security System**.

![Screenshot of a web form containing an application name and description field. Application name has the words 'Security System' entered in.](/attachments/iot-motion/create_application.png)

Now we can create an Access Key. Visit the **Access Keys** link, then **+ Add Access Key**.

![Screenshot showing the Add Access Key button screen](/attachments/iot-motion/access-key.png)

Then, simply press **Create Access Key** for _All Devices_. You'll be presented with an Access Key and Access Secret. These will be `LOSANT_ACCESS_KEY` and `LOSANT_ACCESS_SECRET` respectively in your Arduino code. Keep your secret safe. If you loose it you have to regenerate it and update all your devices with the new secret.

![Screenshot showing the access key and secret obscured by blue bars](/attachments/iot-motion/access_key_and_secret.png)

Now we need to create a device to get a _Device ID_ to put in to our Aruino code to assign to the `LOSANT_DEVICE_ID` constant.

![Screenshot showing the menu for creating a device](/attachments/iot-motion/device-menu.png)

Then click on **Create Blank Device**. 

![Screenshot showing the Create Blank Device](/attachments/iot-motion/blank.png)

Then we can create a new device. Let's give the device a _Name_ of `Bedroom` since this is going to be where the device will be. Then keep the _Device Type_ set to _Standalone_, since it connects directly to Losant.

![Screenshot showing the a web form with Device Name filled in with 'Bedroom' and Device Type set to 'Standalone'](/attachments/iot-motion/new-device.png)

Then you can specify the _Device Attributes_ that the device will send to Losant. In our case it's a boolean value of `motion`. You can have as many device attributes as you'd like, like a temperature reading or humidity reading. Device attributes are contained in the JSON that we send from the device in the `motionDetected()` function to Losant.

![Screenshot showing the a web form with a Device Attribute named 'motion' set to the Data Type 'Boolean'](/attachments/iot-motion/device-attributes.png)

Then click **Create Device**. You'll now get a _Device ID_. You can copy and paste that for your `LOSANT_DEVICE_ID`.

![Screenshot showing the new Device ID](/attachments/iot-motion/bedroom-device-id.png)

Awesome, we have all the required strings for the Arduino Skecth. Update the values for `LOSANT_DEVICE_ID`, `LOSANT_ACCESS_KEY` and `LOSANT_ACCESS_SECRET` and _Upload_ your Arduino Sketch to the Adafruit Feather HUZZAH.

The final step is to get Losant to send us a text message when the motion state is `true`. To do this we need to create a _Workflow_. A _Workflow_ is work triggered by your device's states. In our case we want Losant to send an SMS message (the work) when the `motion` event occurs and is the value of `true`.

Create a Workflow from the drop down menus.

![Screenshot showing the menu for creating a workflow](/attachments/iot-motion/workflow-menu.png)

Enter in a _Workflow Name_ like _Send Notification_ or _Send SMS_ and press **Save Workflow**.

![Screenshot showing a web form with the text Send SMS in a 'Workflow Name' field](/attachments/iot-motion/create-workflow.png)

You are now presented with a simple drag and drop interface to create your workflow. First drag _Device_ from _Triggers_ in to the grid in the center of the screen.

<video controls>
  <source src="/attachments/iot-motion/drag-device.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

Then drag _Conditional_ from the _Logic_ panel.

<video controls>
  <source src="/attachments/iot-motion/drag-conditional.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

We then want to add the expression of...

```
{{ data.motion }} == true
``` 

...in the _Expression_ section of the _Conditional_ panel.

<video controls>
  <source src="/attachments/iot-motion/expression.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

Now let's drag _SMS_ from the _Output_ section and type in your cell phone number in the _Phone Number Template_.

<video controls>
  <source src="/attachments/iot-motion/drag_enter_phone_number.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

Then connect the nodes of the workflow together. Be sure to connect the green connection point on the _Conditional_ to the _SMS_ node. This means the conition has evaluated to true.

<video controls>
  <source src="/attachments/iot-motion/connect-the-flow.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

Oh, let's not forget to include a _Message Template_ of `Motion Detected!`.

<video controls>
  <source src="/attachments/iot-motion/message-contents.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

Finally, we **Deploy Workflow**, meaning it's ready to work when events come in.

Unplug your device and set it up where you want to have it.

Now it's time to see if it works!

<video controls>
  <source src="/attachments/iot-motion/its-alive.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 

And it does! Awesome!

## Conclusion

Losant limits the SMS service to 1 per minute, which is fine for our purposes, but you can integrate with third party services such as telecom API provider Twilio which doesn't have that restriction. 

Taking the project further, you may want to add a button to "Arm" the device, with an LED indicator, so it doesn't send the motion detection JSON when you're expecting movement. 