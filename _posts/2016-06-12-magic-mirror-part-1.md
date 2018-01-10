---
layout: post
title:  "Magic Mirror Part 1"
date:   2016-06-12 12:00:00
tags: nodejs electron diy maker
permalink: /2016/06/12/magic-mirror-part-1/
published: true
---

![Magic Mirror](/images/posts/2016-06-12-mirror.jpg "mirror")

*The software display of my Magic Mirror, with custom widgets.*

With a number of other things behind me there is a home DIY project I want to take a crack at.  A [Magic Mirror](https://www.raspberrypi.org/blog/magic-mirror/) is a home information display showing a clock and useful data readouts from various web services.  Michael Teeuw came up with the concept and put out an [open source project](https://github.com/MichMich/MagicMirror) showing how he did it.  Teeuw's first version was a web app built to run on a raspberry pi with a local Chromium browser put in full screen mode.  The pi is attached to a stripped down flat screen monitor placed behind a piece of one way mirror.  

Several others followed suit making their own alterate versions including [Hannah Mitt on Adafruit](https://learn.adafruit.com/android-smart-home-mirror/overview) using a native Adroid app and an old Nexus 7 tablet.    The Android looked like an attractive place to get started, expecially since I happen to have an spare Nexus 7 tablet from an old mobile web test rig.  The native app gives full control over disabling power-off and full screen.  Comparatively trying to get a browser on an android to be both full screen

Fast forward a few months and Teeuw has updated the code to be more modular and easy to extend as well as making it into an [Electron](http://electron.atom.io/) application.  This is basically a way of packaging a linux, windows, or mac standalone app using HTML, JS, and CSS rather than having to learn a platform specific web framework.  I decided to go this way so that I could build skills that would stay in the ballpark of other technology projects that I want to look into this year.

I'll go through the hardware set up with the shiny, new Raspberry Pi 3 in a future post.  But my first version has a few new modules for the info that I want to have hanging on my wall by my front door:

* The current date and time (using the default)
* The current weather from weather underground
* A weather forecast from weather underground
* The predicted time for an UberX and the current UberX surcharge
* The current status of the bikeshare station near my apartment
* The current platform train times of my local DC Metro stop.

Full code is [here](https://github.com/derickson/MMderickson)

Teeuw's tutorial the update MagicMirror project does a fantastic job stepping a person through modifying the Rasperri Pi to disable screen savers and power saving functions as well as launching the electron app with pm2, a task persistence manager.  The raspberri pi portion of the project, which I was expecting to take most of the project time, actually only took a single morning given that Adafruit [packages their Raspberri Pi 3 kits](https://www.adafruit.com/products/3058) with a preinstalled OS on the included microSD card. 


