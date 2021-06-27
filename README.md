---
page: https://idle.run/bike-brightness
title: Controlling Screen Brightness with a Standing Bike
tags: arduino, bike, screen
date: 2021-06-27
---

![Bike](https://github.com/idlerun/bike-brightness/raw/master/bike.gif)

![Done](https://github.com/idlerun/bike-brightness/raw/master/done.jpg)

## Bike

I found a free standing bike on the side of the road. The handles are removable with a couple bolts, so I took those off.

I cracked open the side of the bike to get a feel for the internals. The braking is done with a series of magnets pulled closer to a iron (guessing) wheel. I removed the dial and spring to leave it always at the hardest setting.

The screen is connected to the pedals with a 2 wire connection. This connection is normally open and closed during one small part of the cycle.

![Internals](https://github.com/idlerun/bike-brightness/raw/master/internals.jpg)

## Wiring

I used an Arduino Leonardo leftover from another project. This Arduino comes with trivial functionality to act as a USB keyboard and send keystrokes to the connected computer.

The wiring was as simple as it gets. The two pedal wires into ground and a digital input (confirmed in a port that has INPUT_PULLUP support).

I taped that in place, drilled a hole for the usb connection. Removed the resistance control as described above, and closed things up.

## Control

I installed some free software ClickMonitorDDC that allows mapping keys to brightness controls. Simple and gets the job done! I used F8 to F12 mapped to some reasonable brightness levels that were easy to test with my normal keyboard.

![Control](https://github.com/idlerun/bike-brightness/raw/master/control.png)

## Code

The code is embarassingly simple. I have a set of hard coded Hz based on the time between pedals. These map to keyboard controls sending the control keys described above.

```
#include <Keyboard.h>
#define PIN 7

int prevMillis;
float DECAY = 0.5;
int level = -1;

void setLevel(int l) {
  if (level != l) {
    level = l;
    Keyboard.write(201+level);
    Serial.print("LEVEL ");
    Serial.println(l);
  }
}

void setup() {
  Serial.begin(9600);
  pinMode(PIN, INPUT_PULLUP);
  Keyboard.begin();
  prevMillis = millis();
  setLevel(4);
}


void checkStalled() {
  int now = millis();
  int elapsed = now - prevMillis;
  if (elapsed > 3000) {
    // STALLED
    setLevel(0);
  }
  delay(10);
}

void loop() {
  while(digitalRead(PIN)==LOW) {
     checkStalled();
  }
  while(digitalRead(PIN)==HIGH) {
     checkStalled();
  }
  int now = millis();
  int elapsed = now - prevMillis;
  prevMillis = now;
  float hz = 1000.0 / elapsed;
  if (hz < 0.4) {
    setLevel(0);
  } else if (hz < 0.5) {
    setLevel(1);
  } else if (hz < 0.6) {
    setLevel(2);
  } else if (hz < 0.8) {
    setLevel(3);
  } else {
    setLevel(4);
  } 
  //Serial.println(hz);
}
```

