---
layout: post
title:  "Quantified Self: Week three progress and real time beer events using Untappd"
date:   2016-01-17 00:00:00
categories: projects, untappd
permalink: /2016/01/17/quantified-self-three-untappd-beer/
published: true
---

![Beer in my dashboard](/images/posts/2016-01-17-beer.jpg "Beer Dashboard")

Untappd beer data turned out to be very easy to add to my quantified self dashboard.  Registerring an app on their api page gives you a client ID and key useable for 100 REST calls an hour without any kind of OAuth trickery (all untapped data is pretty wide open).  I programmed two rest services.  The first a recurisve decent through untappd's /v4/user/checkins/ api and the second using the min_id parameter of the same api to check for recent checkins after the last checkin_id stored in elasticsearch (which is retrievable with sorted, size=1 call using an elasticsearch _search api from nodejs). 

I learned while on a trip to the west coast that fitbit's APIs give data back in a timezone-less fashion, probably to keep information around daily achievement badges intelligible.  This makes life problematic when my tracker and server are in different time zones and caused problems until I returned to the east coast.  I think I'm going to have to add support for data source directed _ids so that i can update old time sections when I know the data source is updated irregularly by apps that work so asynchronously like Fitbit's phone app.  My daily steps are still a few off even when I'm on the east coast, so there's a bug in there somewhere.  It would may be interesting to use the subtraction feature in Timelion to find the data discrepancy.