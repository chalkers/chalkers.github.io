---
title: Tonight's Arduino Project - An Apple Remote Controlled Buzzer
date: 2013-01-09
tags: 
  - Arduino
  - Apple
  - IR
  - Piezo
  - Reverse Engineering
---
I received a package today from [Adafruit](http://adafruit.com) and I decided to tinker with some of the things that I ordered. Two items were a [Piezo Buzzer - PS1240](https://www.adafruit.com/products/160) and an [IR sensor - TSOP38238](https://www.adafruit.com/products/157).

I decided to see if I could read the button presses from an Apple Remote.

![Apple Remote](/attachments/Apple-Remote.jpeg)

I was able to using [Ken Shirriff's](http://www.arcfn.com/) [IRremote](https://github.com/shirriff/Arduino-IRremote) library and the example [IRrecord](https://github.com/shirriff/Arduino-IRremote/blob/master/examples/IRrecord/IRrecord.ino).

	Button         HEX        
	-------------- -----------
	Pause/Play     0x77E12047 
	Menu           0x77E14047 
	Volume Up      0x77E1D047 
	Volume Down    0x77E1B047 
	Rewind         0x77E11047
	Fast Forward   0x77E1E047
	
I wanted to use each key press to do something. I did Pause/Play to stop and start the buzzer, volume up and down to change the tone, rewind to slow down the frequency of tones and fast forward to increase the frequency.

## The Code

I mashed up [Adafruit's Playing a Scale](http://learn.adafruit.com/adafruit-arduino-lesson-10-making-sounds/playing-a-scale) code and the receive code example [IRrecvDemo](https://github.com/shirriff/Arduino-IRremote/blob/master/examples/IRrecvDemo/IRrecvDemo.ino).

I added in some extra logic and variables to customise various aspects of the application. This is by no means a polished piece.

```cpp
#include <IRremote.h>;

#define MENU 0x77E14047
#define PLAY_PAUSE 0x77E12047
#define VOLUME_UP 0x77E1D047
#define VOLUME_DOWN 0x77E1B047
#define REWIND 0x77E11047
#define FAST_FORWARD 0x77E1E047

int RECV_PIN = 11;
int speakerPin = 2;

int numTones = 10;
int tones[] = {261, 277, 294, 311, 330, 349, 370, 392, 415, 440};
int i = 0;
long previousMillis = 0;
long maxTime = 2000;
long between = maxTime;
long beeplength = 50;

boolean isOn = false;

IRrecv irrecv(RECV_PIN);

decode_results results;

void setup()
{
  irrecv.enableIRIn();
}

void loop() {
  
  unsigned long currentMillis = millis();
  
  if(currentMillis - previousMillis > between + beeplength ) {
    previousMillis = currentMillis;  
    if(isOn)
    {
      tone(speakerPin, tones[i]);
    }
  } 
 
  if(currentMillis - previousMillis > beeplength) {
    noTone(speakerPin); 
  }
  if (irrecv.decode(&results)) {
    switch(results.value) 
    {
      case MENU:
          break;
      case PLAY_PAUSE:
          isOn = !isOn;
          break;
      case VOLUME_UP:
        i++;
         break;
      case VOLUME_DOWN:
        i--;
        break;
      case REWIND:
        between= between + 100;
        break;
      case FAST_FORWARD:
        between= between - 100;
        break;
    }
    if(i &gt; numTones) {
      i = numTones;
    }
    
    if(between &gt; maxTime) {
      between = maxTime;
    }
    
    if(i &lt; 0) {
       i = 0;
    }
    if(between &lt; 0) {
      between = 0;
    }
    irrecv.resume();
  }
}
```

### Gotcha

I got this compile error:
	
	core.a(Tone.cpp.o): In function `__vector_7':
	/Applications/Arduino.app/Contents/Resources/Java/hardware/arduino/cores/arduino/Tone.cpp:535: multiple definition of `__vector_7'
	IRremote/IRremote.cpp.o:/Users/andrew/Documents/Arduino/libraries/IRremote/IRremote.cpp:311: first defined here


In order to use `tone()` with the IRremote library I had to modify [IRremoteInt.h](https://github.com/shirriff/Arduino-IRremote/blob/master/IRremoteInt.h) in the `libraries/IRremote` folder on to use `IR_USE_TIMER1` on line 66 and commented out `IR_USE_TIMER2`. I don't fully understand this yet, other than there was some sort of conflict going on, but that's how I got it to work, and that's the important thing!

So now I have the following:

```cpp
    #define IR_USE_TIMER1 
    //#define IR_USE_TIMER2
```

#### Code Download

The code is available on [GitHub](https://github.com/chalkers/IR_Buzzer_Control).

## The Circuit

I wired up the buzzer to pin 2 and ground. It doesn't matter which leg of the buzzer goes to which. With the curved surface of the IR sensor facing toward me, I wired the first leg to pin 11, the second leg to ground and the third to power (5V). 

<img src="/attachments/AppleBuzzer.jpg" alt="Bread Board Diagram" width="484">

## Conclusion

It's pretty amazing what you can do with a few bucks of electronics with an Arduino and I am starting to feel that less and less things are beyond my grasp, which is a great feeling. I feel I've learned a lot already and have made mistakes along the way, but without those mistakes I wouldn't have learned so much!

I plan on documenting my little projects as I go along with a few more epic, thorough guides like [The Absolute Beginner's Guide to Arduino](http://forefront.io/a/beginners-guide-to-arduino) along my way.

What would you do with an IR sensor and a remote you've got kicking about?