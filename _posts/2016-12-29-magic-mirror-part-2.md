---
layout: post
title:  "Magic Mirror Part 2"
date:   2016-12-29 12:00:00
categories: nodejs, electron, quantified-self, diy, maker
permalink: /2016/12/29/magic-mirror-part-2/
published: true
---

![Mirror Display](/images/posts/2016-12-29-mirrordisplay.jpg "mirror")

*The completed project along with a screenshot of the software display behind my Magic Mirror*

In [part one of my D.I.Y. Magic Mirror blog post](/2016/06/12/magic-mirror-part-1/) I showed the software setup of the modules for the excellent Magic mirror project.  This post I'll show the final build out.

![Software architecture](/images/posts/2016-12-29-arch.jpg "software architecture")

I moved to a new home recently, so the I swapped out the capital bikeshare feed with an indicdents feed for the DC metro system.  I live near a bus line, which I find I'm using more often, so I'll probably try to figure out how to get bus ETAs fromt he wmata apis as a later project.

![Wiring Photo](/images/posts/2016-12-29-behindmirror.jpg "wiring photo")

*My Ugly wiring behind the mirror*

The hardware build out only took a few minutes once I had a plastic one-way mirror that fit my frame.  I recommend picking a picture frame you like first and then ordering a plastic mirror that fits it rather than the other way around.  (especially if you are like me and don't ahve access to the space or woodworking tools necessary to build your own frame).  My mirror is held together by electrical tape, which isn't ideal.  I'm a software engineer, not an electrical engineer, so I'm excused.

![Wiring Diagram](/images/posts/2016-12-29-mirrorwires.jpg "wiring diagram")

Here's a cleaned up diagram of what's going on behind all that electrical tape.  I'm using a Raspberry Pi 3 and a "HDMI 4 Pi - 10.1 Display 1280x800 IPS - HDMI/VGA/NTSC/PAL" from adafruit ([link](https://www.adafruit.com/products/1287)).  

![Kibana](/images/posts/2016-12-29-kibana.jpg "kibana metrics")

As the mirror is pulling some interesting data that I want in my quantified-self dashboard project, I wrote some quick nodejs request code to POST commands to my generic REST metrics endpoint for visualizing data from my Magic Mirror in kibana (hosted in [Elastic Cloud](cloud.elastic.co) of course.)

