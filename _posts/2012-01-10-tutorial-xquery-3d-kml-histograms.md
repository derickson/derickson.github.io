---
title: 'Tutorial: XQuery 3D KML Histograms'
author: Dave
layout: post
permalink: /2012/01/10/tutorial-xquery-3d-kml-histograms/
categories:
  - Tutorial
tags:
  - analytics
  - geospatial
  - KML
  - MarkLogic
  - XQuery
---
[<img class="alignnone size-medium wp-image-372" style="border-style: initial; border-color: initial;" title="analytics" src="/wp-content/uploads/2012/01/analytics-300x232.jpg" alt="" width="300" height="232" />][1]

Yesterday, my blog post on [Software Engineering][2] got over 2000 hits because I posted it on [Hacker News][3] as a blogging and social news experiment. (and because I am a huge nerd)  That night, I found myself staring at the real-time geospatial view in Google Analytics and got inspired to type up a similar visualization in XQuery as a thanks to the community for taking a few moments to skim my site.

<img class="alignnone size-full wp-image-370" title="US" src="/wp-content/uploads/2012/01/US.jpg" alt="" width="927" height="398" />

Here is a tutorial for rendering a 3D KML Histogram, also known as a Choropleth diagram,  in MarkLogic.  The example will be on static data, but if the &#8220;event&#8221; content in the MarkLogic data was updated, building a real-time updating visualization would be trivial.  The code needs some tuning, but I&#8217;ll put it out here to let others use for their own purposes.

### Source Data:

I copied and pasted the table of city names from my Google Analytics dashboard to excel.  I then exported the first two columns to CSV, imported this text file using xdmp:document-load, and split the file with fn:tokenize(fn:doc($uri),&#8217;\r&#8217;) to obtain the lines and fn:tokenize($line, &#8216;,&#8217;) to get the column values.

This would be much easier if Google Analytics put the lat/lng information for its Geo view in the detail table so that I could disambiguate cities with the same name.  For exmple I know for a fact there are Hacker-blog readers in both Melbourne, Florida and Melbourne, Australia.  In a real world scenario I would derive the lat/lng from requestor&#8217;s IP myself , just as Google is doing.  I&#8217;ll accept the innacuracy of taking the first city hit off the Google geocoding service for the purposes of this demo:

http://maps.googleapis.com/maps/api/geocode/xml?sensor=false&address=Melbourne

(please copy and paste to a new tab so you don&#8217;t run up my domain&#8217;s daily quota ;) )

To simulate geospatial events being tracked in MarkLogic, I&#8217;ll then insert one geocoded document into my database for each hit that I received and make sure to place this document in a collection called &#8220;event&#8221; to keep it separate from other docs in the database. How you model this isn&#8217;t too important, but here&#8217;s how I did it:


  <pre><code class="language-xml xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;event&gt;
  &lt;name&gt;Bethesda&lt;/name&gt;
  &lt;point&gt;38.9846520, -77.0947092&lt;/point&gt;
&lt;/event&gt;</code></pre>


I place these files in a MarkLogic database that has a geospatial element index on the &#8220;point&#8221; localname.

### Desired KML

So now onto generating the KML for a heatmap.  KML is an XML standard for Google Earth.  Each &#8220;bar&#8221; in the heatmap will actually be a colored extruded polygon which will look like a semi-transparent &#8220;skyscraper&#8221; and be represented in the KML like the following:


  <pre><code class="language-xml xml">&lt;Placemark&gt;
  &lt;description&gt;
    &lt;h3&gt;4 documents in this region.&lt;/h3&gt;
  &lt;/description&gt;
  &lt;styleUrl&gt;#s0012&lt;/styleUrl&gt;
  &lt;Polygon&gt;
    &lt;extrude&gt;1&lt;/extrude&gt;
    &lt;altitudeMode&gt;relativeToGround&lt;/altitudeMode&gt;
    &lt;outerBoundaryIs&gt;
      &lt;LinearRing&gt;
        &lt;coordinates&gt;-54,-36,209600 -54,-27,209600 -63,-27,209600 -63,-36,209600 -54,-36,209600&lt;/coordinates&gt;
      &lt;/LinearRing&gt;
    &lt;/outerBoundaryIs&gt;
  &lt;/Polygon&gt;
&lt;/Placemark&gt;</code></pre>


The coordinates are triples (lon,lat,altitude) listed in counterclockwise order so the surface normals face outward and the  Google Earth lighting equations will work correctly.  The style reference is a pointer to a defined color style listed earlier in the KML.

### Stepping through the Code

My URL rewriter for MarkLogic is a one liner for this example as I want every request on the whole port to go to my map.xqy script.


  <pre><code class="language-xquery xquery">xquery version "1.0-ml" ;
(: just send everything to the KML generating script :)
"/map.xqy"</code></pre>


At the top of the module&#8217;s body I set up variables.  The first two are globals I&#8217;ll be updating with xdmp:set (which should be used carefully as it prevents XQuery from running tasks in parallel.  The $lat $lon and $count variables control the part of the globe that will be heatmapped and the rough number of gridded divisions in each dimension.


  <pre><code class="language-xquery xquery">(: variables that will track maximum values :)
let $maxfreq := 0
let $maxregion := ()

(: analytics bounds -- set to entire world :)
let $lat1 :=  -90
let $lat2 :=  90
let $lon1 :=  -180
let $lon2 :=  180
let $count := 80 </code></pre>


I then derive a $countx and a $county which will be the number of grid divisions in each dimension with an eye towards keeping the regions &#8220;square.&#8221;  Remember that longitude has twice the numerical range as latitude.


  <pre><code class="language-xquery xquery">  (: attempt to make the buckets square :)
  let $distx := ($lon2 - $lon1)  (: cts:distance( cts:point($lat1,$lon1), cts:point($lat1, $lon2) ) :)
  let $disty := ($lat2 - $lat1) (: cts:distance( cts:point($lat1,$lon1), cts:point($lat2, $lon1) ) :)
  let $mindist := fn:min(($distx,$disty))
  let $sidedist := $mindist div $count
  let $_ := xdmp:log(text{"sidedist",$sidedist},"error")
  let $countx := 
    if( fn:ceiling($distx div $sidedist) castable as xs:integer ) then
      xs:integer(fn:ceiling($distx div $sidedist))
    else
      $count * 2
  let $county :=
    if( fn:ceiling($disty div $sidedist) castable as xs:integer ) then
      xs:integer(fn:ceiling($disty div $sidedist))
    else
      $count</code></pre>


I then run the histogram analysis in MarkLogic.  This is the easy part because the Search API has a built-in function for doing just this.


  <pre><code class="language-xquery xquery">  let $searchres := 
      search:search(
          "",
      &lt;options xmlns="http://marklogic.com/appservices/search"&gt;
        &lt;additional-query&gt;
          {cts:collection-query("event")}
        &lt;/additional-query&gt;
        &lt;constraint name="mygeo"&gt;
        &lt;geo-elem&gt;
            &lt;heatmap s="{$lat1}" w="{$lon1}" n="{$lat2}" e="{$lon2}" latdivs="{$county}" londivs="{$countx}"/&gt;
            &lt;facet-option&gt;gridded&lt;/facet-option&gt; 
            &lt;element ns="" name="point"/&gt; 
        &lt;/geo-elem&gt;
        &lt;/constraint&gt;
        
        &lt;return-results&gt;false&lt;/return-results&gt;  
        &lt;return-facets&gt;true&lt;/return-facets&gt;
      &lt;/options&gt;
      )</code></pre>


I remove out of range boxes (the Search API returns regions stretching from the poles to your outer bounds, which I don&#8217;t want.


  <pre><code class="language-xquery xquery">let $boxes := 
    for $box in $searchres//search:box
    let $s := xs:float($box/@s)
    let $w := xs:float($box/@w)
    let $n := xs:float($box/@n)
    let $e := xs:float($box/@e)
    return
    if( ($s ge $lat1) and 
        ($n le $lat2) and 
        ($w ge $lon1) and 
        ($e le $lon2) ) then  
            let $_ := xdmp:set( $maxfreq,  fn:max( ($maxfreq, xs:integer( $box/@count )) ) )
            return
            $box
        else
          (: remove this box, because it is accounting for hits outside the search area :)
            ()</code></pre>


We then generate the KML markers.  I&#8217;ll keep a map of the styles so I can deduplicate them when serializing the KML.


  <pre><code class="language-xquery xquery">(:  Make sure to specify coordinates in CCW order so that KML lighting works correctly :)
declare function local:coord-from-box($box as cts:box, $alt as xs:double){
    &lt;kml:coordinates&gt;
    {
        let $south := xs:string(cts:box-south($box))
        let $west := xs:string(cts:box-west($box))
        let $north := xs:string(cts:box-north($box))
        let $east := xs:string(cts:box-east($box))
        let $alt := xs:string($alt)
        return
        (
        
            fn:string-join((
                fn:string-join(($east,$south,$alt),","),
                fn:string-join(($east,$north,$alt),","),
                fn:string-join(($west,$north,$alt),","),
                fn:string-join(($west,$south,$alt),","),
                fn:string-join(($east,$south,$alt),",")
            )," ")
        )  
    }
    &lt;/kml:coordinates&gt;
};


let $stylemap := map:map()
let $markers := 
    for $box at $x in $boxes
        let $s := xs:float($box/@s)
        let $w := xs:float($box/@w)
        let $n := xs:float($box/@n)
        let $e := xs:float($box/@e)
        let $freq := xs:integer($box/@count)
        return
      
          let $_ := if($freq eq $maxfreq) then xdmp:set($maxregion, $box) else ()
          let $alpha := if ($freq ge 1) then 0.5 else 0.1
          let $freq-prec := xs:double(fn:substring( xs:string($freq div $maxfreq), 1 , 5))
          let $color-prec := xs:double(fn:substring( xs:string((if($freq eq 0) then 1 else $freq) div $maxfreq), 1 , 5))
          let $style-name := fn:concat("s",fn:replace(xs:string($freq-prec),"\.",""))
          let $_ := if(map:get($stylemap,$style-name)) then () else map:put($stylemap, $style-name,
              &lt;Style id="{$style-name}" xmlns="http://www.opengis.net/kml/2.2"&gt;
                  &lt;PolyStyle&gt;
                      &lt;color&gt;{local:html-color-from-percentage($color-prec, $alpha)}&lt;/color&gt;
                      &lt;colorMode&gt;normal&lt;/colorMode&gt;
                  &lt;/PolyStyle&gt;
              &lt;/Style&gt;
              )
          let $alt := (200000 + (800000 * $freq-prec)) * (xs:float($sidedist) div xs:float(9.0)) 
          let $ctsbox := cts:box($s,$w,$n,$e)
          return
              &lt;Placemark  xmlns="http://www.opengis.net/kml/2.2"&gt;
                  &lt;description&gt;
                      &lt;h3&gt;{$freq} document{if($freq gt 1) then 's' else ()} in this region.&lt;/h3&gt;
                  &lt;/description&gt;
                  &lt;styleUrl&gt;#{$style-name}&lt;/styleUrl&gt;
                  &lt;Polygon xmlns="http://www.opengis.net/kml/2.2"&gt;
                      &lt;extrude&gt;1&lt;/extrude&gt;
                      &lt;altitudeMode&gt;relativeToGround&lt;/altitudeMode&gt;
                      &lt;outerBoundaryIs&gt;
                        &lt;LinearRing&gt;
                        {local:coord-from-box($ctsbox,$alt)}
                        &lt;/LinearRing&gt;
                      &lt;/outerBoundaryIs&gt;
                    &lt;/Polygon&gt;
              &lt;/Placemark&gt;</code></pre>


The last step is to return the KML


  <pre><code class="language-xquery xquery">  return (
      xdmp:set-response-content-type("application/vnd.google-earth.kml+xml"),
      '&lt;?xml version="1.0" encoding="UTF-8"?&gt;',
      
      &lt;kml xmlns="http://www.opengis.net/kml/2.2"&gt;
        &lt;Document&gt;
            &lt;name&gt;KML Heatmap Global&lt;/name&gt;
            &lt;open&gt;1&lt;/open&gt;
            
            {$camera}
            
            {
                &lt;Style id="ml"&gt;
                  &lt;IconStyle&gt;
                    &lt;color&gt;FF3122D9&lt;/color&gt;
                  &lt;/IconStyle&gt;
                &lt;/Style&gt;,
            
                for $key in map:keys($stylemap)
                return
                map:get($stylemap,$key),
            
                $markers
            }
        &lt;/Document&gt;
      &lt;/kml&gt;
      )</code></pre>


### More Screenshots created by varying the input variables:

[<img class="alignnone size-full wp-image-373" title="SF" src="/wp-content/uploads/2012/01/SF.jpg" alt="" width="600" height="296" />][4]

&nbsp;

[<img class="alignnone size-full wp-image-374" title="world" src="/wp-content/uploads/2012/01/world.jpg" alt="" width="300" height="247" />][5]

### The Complete Source:


  <pre><code class="language-xquery xquery">
xquery version "1.0-ml";

import module namespace search="http://marklogic.com/appservices/search"
                    at "/MarkLogic/appservices/search/search.xqy";

declare namespace kml = "http://www.opengis.net/kml/2.2";

(:White to red color scale in BBGGRR format :)
declare variable $COLOR_SCALE :=
(
"CCFFFF",
"A0EDFF",
"76D9FE",
"4CB2FE",
"3C8DFD",
"2A4EFC",
"1C1AE3",
"2600BD",
"2600BD"
);


(:~ 
  Converts number to single digit hex 
  I'm sure there is a better way to do this in XQuery, but I am lazy
:)
declare function local:numToHex($n) as xs:string {
    if($n gt 15) then 'f'
    else if($n gt 9) then 
      if($n eq 10) then 'a'
      else if($n eq 11) then 'b'
      else if($n eq 12) then 'c'
      else if($n eq 13) then 'd'
      else if($n eq 14) then 'e'
      else if($n eq 15) then 'f' else '0'

    else xs:string($n)
};

(: takes from 1.0 to 0.0 :)
declare function local:numToHexPair($num) {
    let $intalpha := fn:round(255 * $num)
    let $big := $intalpha idiv 16
    let $small := $intalpha mod 16
    let $bigchar := local:numToHex($big)
    let $smallchar := local:numToHex($small)
    return
    fn:concat($bigchar,$smallchar)
};

(:  Make sure to specify coordinates in CCW order so that KML lighting works correctly :)
declare function local:coord-from-box($box as cts:box, $alt as xs:double){
    &lt;kml:coordinates&gt;
    {
        let $south := xs:string(cts:box-south($box))
        let $west := xs:string(cts:box-west($box))
        let $north := xs:string(cts:box-north($box))
        let $east := xs:string(cts:box-east($box))
        let $alt := xs:string($alt)
        return
        (
        
            fn:string-join((
                fn:string-join(($east,$south,$alt),","),
                fn:string-join(($east,$north,$alt),","),
                fn:string-join(($west,$north,$alt),","),
                fn:string-join(($west,$south,$alt),","),
                fn:string-join(($east,$south,$alt),",")
            )," ")
        )  
    }
    &lt;/kml:coordinates&gt;
};

(: Color is specified in octal of hex pairs representing transparency 
   alpha and color triple AABBGGRR example: 88ff0000 :)
declare function local:html-color-from-percentage($freq-prec, $alpha) {
    let $colornum := xs:integer(  fn:ceiling($freq-prec * fn:count($COLOR_SCALE))  )
    let $colornum := if($colornum eq 0) then 1 else $colornum
    let $color := $COLOR_SCALE[$colornum]
    return
    fn:concat(
        local:numToHexPair($alpha),
        $color
    )
};



 

(: variables that will track maximum values :)
let $maxfreq := 0
let $maxregion := ()

(: analytics bounds -- set to entire world :)
let $lat1 :=  -90
let $lat2 :=  90
let $lon1 :=  -180
let $lon2 :=  180
let $count := 80 


  
  (: attempt to make the buckets square :)
  let $distx := ($lon2 - $lon1)  (: cts:distance( cts:point($lat1,$lon1), cts:point($lat1, $lon2) ) :)
  let $disty := ($lat2 - $lat1) (: cts:distance( cts:point($lat1,$lon1), cts:point($lat2, $lon1) ) :)
  let $mindist := fn:min(($distx,$disty))
  let $sidedist := $mindist div $count
  let $_ := xdmp:log(text{"sidedist",$sidedist},"error")
  let $countx := 
    if( fn:ceiling($distx div $sidedist) castable as xs:integer ) then
      xs:integer(fn:ceiling($distx div $sidedist))
    else
      $count * 2
  let $county :=
    if( fn:ceiling($disty div $sidedist) castable as xs:integer ) then
      xs:integer(fn:ceiling($disty div $sidedist))
    else
      $count
  

  let $searchres := 
      search:search(
          "",
      &lt;options xmlns="http://marklogic.com/appservices/search"&gt;
        &lt;additional-query&gt;
          {cts:collection-query("event")}
        &lt;/additional-query&gt;
        &lt;constraint name="mygeo"&gt;
        &lt;geo-elem&gt;
            &lt;heatmap s="{$lat1}" w="{$lon1}" n="{$lat2}" e="{$lon2}" latdivs="{$county}" londivs="{$countx}"/&gt;
            &lt;facet-option&gt;gridded&lt;/facet-option&gt; 
            &lt;element ns="" name="point"/&gt; 
        &lt;/geo-elem&gt;
        &lt;/constraint&gt;
        
        &lt;return-results&gt;false&lt;/return-results&gt;  
        &lt;return-facets&gt;true&lt;/return-facets&gt;
      &lt;/options&gt;
      )


let $boxes := 
    for $box in $searchres//search:box
    let $s := xs:float($box/@s)
    let $w := xs:float($box/@w)
    let $n := xs:float($box/@n)
    let $e := xs:float($box/@e)
    return
    if( ($s ge $lat1) and 
        ($n le $lat2) and 
        ($w ge $lon1) and 
        ($e le $lon2) ) then  
            let $_ := xdmp:set( $maxfreq,  fn:max( ($maxfreq, xs:integer( $box/@count )) ) )
            return
            $box
        else
          (: remove this box, because it is accounting for hits outside the search area :)
            ()

let $stylemap := map:map()
let $markers := 
    for $box at $x in $boxes
        let $s := xs:float($box/@s)
        let $w := xs:float($box/@w)
        let $n := xs:float($box/@n)
        let $e := xs:float($box/@e)
        let $freq := xs:integer($box/@count)
        return
      
          let $_ := if($freq eq $maxfreq) then xdmp:set($maxregion, $box) else ()
          let $alpha := if ($freq ge 1) then 0.5 else 0.1
          let $freq-prec := xs:double(fn:substring( xs:string($freq div $maxfreq), 1 , 5))
          let $color-prec := xs:double(fn:substring( xs:string((if($freq eq 0) then 1 else $freq) div $maxfreq), 1 , 5))
          let $style-name := fn:concat("s",fn:replace(xs:string($freq-prec),"\.",""))
          let $_ := if(map:get($stylemap,$style-name)) then () else map:put($stylemap, $style-name,
              &lt;Style id="{$style-name}" xmlns="http://www.opengis.net/kml/2.2"&gt;
                  &lt;PolyStyle&gt;
                      &lt;color&gt;{local:html-color-from-percentage($color-prec, $alpha)}&lt;/color&gt;
                      &lt;colorMode&gt;normal&lt;/colorMode&gt;
                  &lt;/PolyStyle&gt;
              &lt;/Style&gt;
              )
          let $alt := (200000 + (800000 * $freq-prec)) * (xs:float($sidedist) div xs:float(9.0)) 
          let $ctsbox := cts:box($s,$w,$n,$e)
          return
              &lt;Placemark  xmlns="http://www.opengis.net/kml/2.2"&gt;
                  &lt;description&gt;
                      &lt;h3&gt;{$freq} document{if($freq gt 1) then 's' else ()} in this region.&lt;/h3&gt;
                  &lt;/description&gt;
                  &lt;styleUrl&gt;#{$style-name}&lt;/styleUrl&gt;
                  &lt;Polygon xmlns="http://www.opengis.net/kml/2.2"&gt;
                      &lt;extrude&gt;1&lt;/extrude&gt;
                      &lt;altitudeMode&gt;relativeToGround&lt;/altitudeMode&gt;
                      &lt;outerBoundaryIs&gt;
                        &lt;LinearRing&gt;
                        {local:coord-from-box($ctsbox,$alt)}
                        &lt;/LinearRing&gt;
                      &lt;/outerBoundaryIs&gt;
                    &lt;/Polygon&gt;
              &lt;/Placemark&gt;
    

    let $camera := if($boxes) then
            &lt;LookAt id="camera1"&gt;
                &lt;longitude&gt;{fn:avg((xs:float($maxregion/@w), xs:float($maxregion/@e)))}&lt;/longitude&gt;
                &lt;latitude&gt;{fn:avg((xs:float($maxregion/@n), xs:float($maxregion/@s)))}&lt;/latitude&gt; 
                &lt;altitude&gt;0&lt;/altitude&gt;
                &lt;altitudeMode&gt;relativeToGround&lt;/altitudeMode&gt;    
                &lt;heading&gt;-10&lt;/heading&gt;
                &lt;tilt&gt;45&lt;/tilt&gt;
                &lt;roll&gt;0&lt;/roll&gt;
                &lt;range&gt;{8000000 * (xs:float($sidedist) div xs:float(9.0))  }&lt;/range&gt;
            &lt;/LookAt&gt;
        else ()
        
        
    return (
      xdmp:set-response-content-type("application/vnd.google-earth.kml+xml"),
      '&lt;?xml version="1.0" encoding="UTF-8"?&gt;',
      
      &lt;kml xmlns="http://www.opengis.net/kml/2.2"&gt;
        &lt;Document&gt;
            &lt;name&gt;KML Heatmap Global&lt;/name&gt;
            &lt;open&gt;1&lt;/open&gt;
            
            {$camera}
            
            {
                &lt;Style id="ml"&gt;
                  &lt;IconStyle&gt;
                    &lt;color&gt;FF3122D9&lt;/color&gt;
                  &lt;/IconStyle&gt;
                &lt;/Style&gt;,
            
                for $key in map:keys($stylemap)
                return
                map:get($stylemap,$key),
            
                $markers
            }
        &lt;/Document&gt;
      &lt;/kml&gt;
      )
      </code></pre>


 [1]: /wp-content/uploads/2012/01/analytics.jpg
 [2]: http://www.front2backdev.com/2012/01/08/software-engineering-lessons-i-learned-playing-the-legend-of-zelda/ "Software Engineering lessons I learned playing The Legend of Zelda"
 [3]: http://news.ycombinator.com/
 [4]: /wp-content/uploads/2012/01/SF.jpg
 [5]: /wp-content/uploads/2012/01/world.jpg