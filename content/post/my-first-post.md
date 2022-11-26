---
title: "Building a DIY Power Meter Camera Sensor using IoT AI on the Edge with Tensorflow"
date: 2022-11-23T23:55:58-05:00
categories:
- Home Assistant
- IoT
- AI
- Power
- ESP32
tags:
- Power
- Dominion
- Smartmeter
- MQTT
- ESP32
- ESP32-CAM
- Sensor
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---
# Introduction
One thing I still had on my HA bucket list was to get a continuous reading on how much power our house consumes and at which rate. The power bills are through the roof and this might provide some insight to start the process of finding consuming devices to reduce the monthly bill.

![Smart power meter](https://cdn-dominionenergy-prd-001.azureedge.net/-/media/images/1920x1080-image-video/employees/1920x776_employees_smartmeter5_mini.jpg?la=en&rev=cf789ef02e52417581222cb60fca8ca0&h=775&w=1920&la=en&hash=A7D21094DC0BC73F18A1EC537B80135A)

For the folks in Europe and more populated areas in the US, there are Home Assistant integrations available, most of which let you get the reading from the power provider's web site. That would be super easy and would not require any local hardware.

Smart meters are beeing installed by Dominion in Virginia, but not for where I am at: Smithfield, VA.
According to the Dominion Virginia Power map for upgrades of the meters, this could take another couple of years for where I am at. Old meter, only has rotating dials - not even readable numbers:

![The meter I still have](/media/blog/122022-DominionOldMeter.jpg)

So how does Dominion read these old meters today? I've never seen a Dominion vehicle or person on my property to read my meter.
My theory is that my old meter has a little radio card in it and I tried to find and decode the frequency it transmits on, without much success, using a software defined radio setup (SDR). Most that tried this before said that these transmissions are also encrypted, but hackable - I didn't feel like spending that much time on getting a simple kWh reading.

Back to square one and then it gets really interesting.

# Research: Let's put a camera on it, then what?
My first thought was to put an IP camera on it, integrate that with Frigate and then look into perhaps some sort of tensorflow thing that frigate already has to recognize what these little dials are doing.
I mean, all you have to do is to find at what angle these red dials are pointing and it should be easily discernable what the consumption reading actually is.

# Solution
Well, that wasn't going to work I thought, so I took a research break from this for a few months.
Low and behold, eventually something turned up that looks very promising.

Turns out that there's a German landsman that figured out how to make this work for measuring water consumption at his house, using a camera, AI edge computing and MQTT.
[AI-on-the-edge-device on a Water Meter](https://github.com/jomjol/AI-on-the-edge-device)

## Wait, What?
I think this is amazing and ground breaking work by [Jomjol](https://github.com/jomjol), taking the leap to edge computing for something as simple as this. I'd call this either ignorance, stupid or pure genious.
I vote for pure genious.

This is where we enter the world of the ESP32, basically a modern computer that has everything included but is smaller than a match box. There are other choices to solve this problem, better explained here: [Ardunio vs Rasberry PI vs ESP32](https://www.cnx-software.com/2020/03/24/know-the-differences-between-raspberry-pi-arduino-and-esp8266-esp32/)

The ESP32 has 32-bit LX6 Xtensa® Dual-Core microprocessors, which run 600 DMIPS. The two cores are named Protocol CPU (PRO_CPU) and Application CPU (APP_CPU).

## What about the camera?
Check this out: The ESP32 comes sandwiched with a camera already at low and behold a few bucks, meet the ESP32-CAM dual board, for cheap: $17.99 for two of these.
[ESP32-CAM on Amazon](https://www.amazon.com/gp/product/B0948ZFTQZ)

![ESP32-CAM](/media/blog/122022-ESP32-CAM.png)
![ESP32-CAM](/media/blog/122022-ESP32-CAM.png)

Yes, two! So you can keep on for another project or have a backup if you brick one ;-)

## Implementation
### Attaching the Camera
The first thing I did was to secure the camera.
It was flopping around, so I slid a small square of double sided tape under it to secure it to SD-Card slot holder metal piece.
### Micro SD-Card
If you're in a hurry, this is not a project for you.
This will not work with any SD-Cards larger than 8GB.
Ha-ha, who has one of those at home?? You'll have to order one.
Also order an adapter if you do not have one.
#### Formatting the card
What worked best is using the following command line on either a 4GB or 8GB card (anything bigger will fail):
![ESP32-CAM](/media/blog/122022-FormatSD.png)
