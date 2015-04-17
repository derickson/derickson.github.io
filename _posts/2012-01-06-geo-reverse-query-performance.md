---
title: Geo Reverse Query Performance
author: Dave
layout: post
permalink: /2012/01/06/geo-reverse-query-performance/
categories:
  - Experiments
tags:
  - alerting
  - geospatial
  - performance
  - reverse query
---
The MarkLogic Express License, which can be used in production for free, is most exciting for me because it includes both the geospatial and alerting features of MarkLogic server.  Combined with the ability to do reverse queries, these features make MarkLogic really stand out from ANY other technology when it comes to building real-time geospatial applications.  I&#8217;ll review a couple of concepts before diving into a performance profiling example:

**Geospatial Query**: MarkLogic has a number of geospatial construction and conversion functions built in, but querying usually boils down to finding which  geospatial points (a latitude / longitude pair) are within a geospatial region.  In the examples below I&#8217;ll be using a circle region to query for points.

**Reverse Query**: MarkLogic searches execute on [inverted search indexes][1], which make up the MarkLogic Universal Index and optional additional user specified indexes like ranges, lexicons, and geospatial.  Using these indexes you can get a sub-second response to the question of which documents in a mountain of XML match a single fairly complex query.  Reverse queries allow you to do the opposite.  If you have a mountain of queries (say, descriptions of what users are interested in, rules, or alert specifications)  you can get a sub-second response to the question of which of the millions of queries match a single document.  Basically each query is serialized as XML, by the magic of the super-composable cts:query spec, and queried as if it were a document.  The cts:reverse-query() function turns any XML document into a query that will match stored queries.

**Alerting**:  Alerting is a MarkLogic support framework for having &#8220;actions&#8221; execute when a &#8220;rule&#8221; is matched by an incoming document.  Actions are XQuery scripts to be run.  Rules are cts:queries that target incoming documents.  The alerting framework simplifies creating and managing serialized queries, database triggers, and other complex parts of building an alerting app.

An example geospatial alert could be something like a real estate website which sends an email to a user when a new property becomes available within X miles of where the user has indicated they are interested in purchasing a home.  Another example is geoclassification.

An example of a geo reverse query that isn&#8217;t involved in alerting would be a [real-time 3D choropleth map][2] analytic (aka 3D geospatial facet, image thanks to <http://thematicmapping.org/>).  By knowing the polygon border of a country, state, neighborhood etc you could use a serialized polygon query and reverse query combination to classify documents as they are inserted and facet on this information as users refine their queries.

**Performance Run**:

As a test I&#8217;m going to attempt to show the scalability advantages of MarkLogic.  One of the problems with internet-scale alerting apps on traditional platforms is that they tend to have large hardware footprints or large latency.  If you have N alerts, then every time a new document is inserted, you might have to run N queries.  This does not scale well when the number of alerts starts climbing to meet the needs of large numbers of users.  If your site has ten thousand users who each have 100 alerts, that&#8217;s one million queries to be run on every database insert.  Not going to happen!  Most sites have to resort to periodic batch runs. This works, but it&#8217;s no longer real-time.  What if that alert is mission critical?  What if the alert could save someone&#8217;s life?

My test harness for real time reverse queries in MarkLogic.

I will submit a series of points which represents the path of an object on the earth&#8217;s globe.  There will be N+1 circles distributed over the globe which represent people interested in the object checking for intersections.  N circles will be negative matches, the +1 circle is the one that matches.  I will measure the time it takes to find for circles that match the object&#8217;s path for increasing values of N.

<table width="209" border="1" cellspacing="0" cellpadding="3px">
  <!--StartFragment-->
  
  <br /> <colgroup> <col width="85" /> <col width="124" /> </colgroup> <tr>
    <td width="85" height="13">
      Random Circles (N)
    </td>
    
    <td width="124">
      Average Query Time (s)
    </td>
  </tr>
  
  <tr>
    <td align="right" height="13">
      100
    </td>
    
    <td align="right">
      0.00237
    </td>
  </tr>
  
  <tr>
    <td align="right" height="13">
      1000
    </td>
    
    <td align="right">
      0.00305
    </td>
  </tr>
  
  <tr>
    <td align="right" height="13">
      10000
    </td>
    
    <td align="right">
      0.01336
    </td>
  </tr>
  
  <tr>
    <td align="right" height="13">
      100000
    </td>
    
    <td align="right">
      0.03499
    </td>
  </tr>
  
  <tr>
    <td align="right" height="13">
      1000000
    </td>
    
    <td align="right">
      0.26044
    </td>
  </tr>
</table>

0.26 seconds to test one million alerts is fast!  The reverse query can be computed in parallel, so a MarkLogic cluster can distribute processing and the combine the answers (real-time MapReduce?) to allow for alerting on orders of magnitude more alerts without sacrificing sub-second response.

Lastly, here is my code:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

(: a whole bunch of random circles to make the reverse-query harder :)
let $n := 10000

for $count in (1 to $n)
let $lat := xdmp:random(180) - 90
let $lon := xdmp:random(360) - 180
let $query-doc := 
  
  let $a := cts:point( $lat,$lon)
  let $b := cts:point( $lat + 0.0001, $lon)
  let $circle := cts:circle( cts:distance($a, $b), $a)
  return
  &lt;query&gt;
    {cts:element-geospatial-query( xs:QName("point"), $circle , ("coordinate-system=wgs84"))}
    &lt;a&gt;{$a}&lt;/a&gt;
    &lt;b&gt;{$b}&lt;/b&gt;
  &lt;/query&gt;

return
xdmp:document-insert( fn:concat("/circle-",$count,".xml"), $query-doc );

(: the actual matching circle :)

let $query-doc := 
  
  let $a := cts:point( 10.0005,36)
  let $b := cts:point( 10.00051,36)
  let $circle := cts:circle( cts:distance($a, $b), $a)
  return
  &lt;query&gt;
    {cts:element-geospatial-query( xs:QName("point"), $circle , ("coordinate-system=wgs84"))}
    &lt;a&gt;{$a}&lt;/a&gt;
    &lt;b&gt;{$b}&lt;/b&gt;
  &lt;/query&gt;

return
xdmp:document-insert( fn:concat("/matching-cicle.xml"), $query-doc )
</code></pre>



  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

(: the line of points which will become a reverse query to match circles:)

let $a := cts:point( 10.0005, 36)
let $b := cts:point( 10.00052, 36)
let $distance := cts:distance($a, $b)
let $bearing := cts:bearing($a, $b)
let $steps := 100
let $step-distance := $distance div $steps
let $points := 
   for $i in (0 to $steps)
   return
   cts:destination($a, $bearing, $step-distance * $i)

let $doc :=
&lt;document&gt;{
  for $point in $points
  return
  &lt;point&gt;{$point}&lt;/point&gt;
}&lt;/document&gt;

return
(

cts:search( /,  cts:reverse-query( $doc ) ) ,
xdmp:elapsed-time()
)</code></pre>


 [1]: http://en.wikipedia.org/wiki/Inverted_index
 [2]: http://4.bp.blogspot.com/_yECf1Q0GlOk/SB9SnTJBivI/AAAAAAAABbw/EFMZQnaYrb4/s1600/google_earth_internet_users.png