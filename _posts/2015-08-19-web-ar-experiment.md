---
layout: post
title:  "Web Augmented Reality Experiment"
date:   2015-08-19 18:00:00
categories: work, tech, WebAR, visualization
permalink: /2015/08/19/web-ar-experiment/ 
published: true
---

![WebAR GIF](/images/posts/2015-08-19-Augmented.gif "WebAR GIF")

Above is a weekend project visualizing data from Logstash and Elasticsearch using Web Augmented Reality.  

According to many on the web working with Virtual Reality (VR) and Augmented Reality (AR) technologies, something big is about to happen.  With the upcoming release of a number of gaming head mounted display (HMD) units [1][2] and a cool AR demo from Microsoft at this year's E3. [3]  This is going to cause a lot of attention and investment in VR/AR.  This tech has been around in research spaces for a while, see the 1985 NASA HMD below, but I agree that a perfect storm is coming.

![Smithsonian exhibit, NASA VR Helmet 1985](/images/posts/2015-08-19-NASA.jpg "Smithsonian exhibit, NASA VR Helmet 1985")

The thing that is different is that the devices necessary to do the processing necessary for 3D and video capture are getting smaller and more affordable.  It's the same movement that's causing wearables to get so much attention.  There may not be a practical consumer application for VR/AR beyond video games right now, but it may be coming.  All it will take is one killer app, and I suspect every kid in the world is going to want AR googles that fit that fancy Android / iOS computer they carry around with them everywhere.

One of the potential uses for AR I see is collaborative data visualization.  Imagine standing around a table with tablets virtually "projecting" a map or a graph onto a conference room table.  Becuase augmented reality can have a 1:1 spatial mapping with the real world, actions like pointing with your real hand at something work just fine as long as everyone in the room has a tablet, phone, HMD, or a conference room camera system for the benefit of those telecon'ed in.

The challenge here, as is often the case when personal "bring your own device" is being leveraged to speed our transition to a shared immersive world, is making use of the tech already in consumers hands and standards.  Both Android and iOS devices are capbable of making native applications that do AR; however, the build-once run everywhere framework promise always breaks down somewhere along the line and the people I'm talking to tell me that that line is VR/AR.   This is where web technolgies can come to the rescue.  Google cardboard has shown us that web browsers (now both on Android and iOS) are perfectly capable of rending a partial VR experience in mobile browswer (no head tracking).  The only thing left for collaboration is true integration of the forward (outward) facing camera.  Two points I'd like to make here:

1) I really like the approach put forward by Boris Smus of Google on Responsive VR.  His WebVR boilerplate code is a great way of hosting one VR enabled site that gracefully transitions from desktop, to mobile touch screent, to stereoscopic VR the same way responsive web layout tools do. [4][5]

2) It is so rediculously frustrating that iOS doesn't support the HTML5 standards for pulling webcamera data from within their embedded browser.  If they did that, universal WebAR would probably already be here. [6]

I'll keep working on this to make it more demo ready, but it's going to stay a toy for a while (My code: [7]).  Special thanks to jeromeetienne for the excellent examples, sample code, and Threex library. [8]

![Still image of laptop camera rig](/images/posts/2015-08-19-Augmented.jpg "WebAR")


* [1] [Oculus Rift](https://www.oculus.com)
* [2] [HTC Vive](http://www.htcvr.com/)
* [3] [Microsoft Hololens Demo E3 2015](https://www.youtube.com/watch?v=6yg6ljnASxw)
* [4] [Boris Smus's WebVR Boilerplate](https://github.com/borismus/webvr-boilerplate)
* [5] [Boris Smus on responsive VR](http://smus.com/responsive-vr/)
* [6] [HTML5 incompatibility on iOS](http://mobilehtml5.org/)
* [7] [My code for the AR experiment](https://github.com/derickson/es-ar-experiment)
* [8] [Inspiration from Threex project](https://github.com/jeromeetienne/threex.webar)