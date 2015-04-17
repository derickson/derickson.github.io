---
title: Maps and Profiling Performance
author: Dave
layout: post
permalink: /2011/12/07/maps-and-profiling-performance/
categories:
  - Software Tip
tags:
  - map
  - MarkLogic
  - performance
  - profiling
  - XQuery
---
Kurt Cagle over at XML Today [posted a great blog][1] about the map:map library in MarkLogic.  [map:map][2] is a hashtable-like map implementation inside MarkLogic that has measurable performance advantages over raw sequences and predicates in XQuery.  Wait &#8230; performance advantages?  How can one tell?  The quickest way is to punch up a code example in XQuery and run it in CQ or the new MarkLogic5 Query Console using the &#8220;Profile&#8221; mode.  You&#8217;ll get the runtime and a breakdown of each step of the XQuery evaluation and how it contributed to the total time.

Here&#8217;s a trick with maps that Kurt didn&#8217;t mention.  You can subtract two maps quickly *diff* two lists of information.  First the slower XQuery sequences approach:

<pre>let $a := for $i in (1 to 10000) return xs:string($i)
let $b := for $i in (1 to 10000) return if (xdmp:random(1000) &gt; 990) then () else xs:string($i)
return
$a[ fn:not( . = $b ) ]</pre>

This has around a 2.8 to 3 second run time on my laptop using the Query Console profile feature.  There are probably better ways to do this with XQuery, but I&#8217;m trying to demonstrate how cool maps are, so I&#8217;ll let that go for now.

Here is the same code implemented with maps.  This has about a 0.1 second average run time my laptop.  The performance divide just gets wider as I increase the size of the lists being compared.

<pre>let $a := for $i in (1 to 10000) return xs:string($i)
let $b := for $i in (1 to 10000) return if (xdmp:random(1000) &gt; 990) then () else xs:string($i)
let $mapa := map:map()
let $mapb := map:map()
let $_ := for $av in $a return map:put($mapa, $av, $av)
let $_ := for $bv in $b return map:put($mapb, $bv, $bv)
return
map:keys( $mapa - $mapb )</pre>

&nbsp;

 [1]: http://www.xmltoday.org/content/map-maker-map-maker-make-me-map
 [2]: http://developer.marklogic.com/pubs/5.0/apidocs/map.html