---
title: Using KML to define Geofences
author: Dave
layout: post
permalink: /2013/02/14/using-kml-to-define-geofences/
categories:
  - Experiments
tags:
  - alerting
  - geofence
  - HTML5
  - KML
  - MarkLogic
  - Mobile
---
I&#8217;ve written about [geospatial alerting][1] in previous posts.  Here&#8217;s a fun example where we drive [Philips Hue lighting changes][2] based on geofence definitions driven by a mobile device&#8217;s GPS.

Demo Overview:

  * MarkLogic will be configured to accept KML with colored polygons as a description of geofences
  * Each supplied geofence will be used to create an alerting rule
  * <span style="line-height: 13px;">An HTML5 mobile app will poll the GPS device of my phone to send &#8220;ping&#8221; messages to MarkLogic</span>
  * MarkLogic will insert a &#8220;ping&#8221; document containing the lat/lon of the phone
  * MarkLogic will detect whether the ping document exists within a saved geofence using reverse queries and the alerting framework.
  * If we have a hit, the demo will change the lights in my living-room to the color of the KML polygon which defined the geofence.

Results:
<iframe width="420" height="315" src="https://www.youtube.com/embed/-A598bnDT6g" frameborder="0" allowfullscreen></iframe>


App Install:

  1. Create a marklogic database with an assigned triggers database
  2. Turn on fast reverse query for the database
  3. Create a new HTTP app server, pointing it&#8217;s modules to the xquery code in the following [Source code][3]
  4. from qconsole, with the dropdown set to the primary db (not the trigger db) run the [installAlert.xqy][4] script

Creating some geofences:

[<img class="alignnone size-medium wp-image-516" alt="kml" src="/wp-content/uploads/2013/02/kml-300x248.jpg" width="300" height="248" />][5]

  1. <span style="line-height: 13px;">Using the &#8220;My Places&#8221; feature of Google Maps, create a new map</span>
  2. draw some polygons on the map and give each a different color
  3. I placed a red polygon on the right side of my front lawn, and a blue polygon on the left side of my front lawn so that as I cross the walkway to my apartment, I move from one region to another.
  4. Copy and paste the &#8220;sharing&#8221; link to the map.  By adding &#8220;&output=kml&#8221; to the end of the URL, google will provide a KML view of the map.

Now we inert this record into MarkLogic as a geo fence from qconsole.  This code will download the KML, parse it into polygons, and create alerts:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";
import module namespace mkf = "http://derickson/kmlalert/model/m-kf" at "/model/m-kmlfence.xqy";
import module namespace lk = "http://derickson/kmlalert/lib/kml" at "/lib/l-kml.xqy";

declare namespace kml ="http://www.opengis.net/kml/2.2";
declare namespace gx ="http://www.google.com/kml/ext/2.2";

mkf:insert-fence(
lk:get-google-map("https://maps.google.com/maps/ms?msid=2041XXXXXX Your URL GOES HERE")
)
</code></pre>


A quick test of the alert by manually inserting a ping:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";
import module namespace mp = "http://derickson/kmlalert/model/m-ping" at "/model/m-ping.xqy";


mp:store( mp:gen-ping( cts:point(38.XXXX,-77.XXX) ) )</code></pre>


For the video above, I&#8217;m driving the geo alert from my phone with a quick Ember.js + HTML5 geo alerting + JQuery.ajax application.  The main code to look at is the [Geo.js][6] file (which I stripped from something else I&#8217;m working on, so pardon the strange variable names)

&nbsp;

 [1]: http://www.front2backdev.com/2012/01/06/geo-reverse-query-performance/ "Geo Reverse Query Performance"
 [2]: http://www.front2backdev.com/2013/02/03/controlling-philips-hue-with-xquery/ "Controlling Philips Hue with XQuery"
 [3]: https://github.com/derickson/kmlfence
 [4]: https://github.com/derickson/kmlfence/blob/master/script/installAlert.xqy
 [5]: /wp-content/uploads/2013/02/kml.jpg
 [6]: https://github.com/derickson/kmlfence/blob/master/xquery/static/js/Geo.js