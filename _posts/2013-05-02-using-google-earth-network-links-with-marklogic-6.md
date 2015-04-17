---
title: Using Google Earth Network Links with MarkLogic 6
author: Mark
layout: post
permalink: /2013/05/02/using-google-earth-network-links-with-marklogic-6/
categories:
  - Guest Post
  - Tutorial
tags:
  - geospatial
  - KML
  - MarkLogic
---
*A guest post from my coworker, * [Mark][1],* who showed me this cool trick with Google Earth.  Enjoy! &#8211;Dave*

&nbsp;

By nature, I’m a visual person – I prefer graphs, charts and animations to reams of paper when it comes to analyzing information.  Geospatial data is no exception; in fact it’s where I’ve spent the last several years of my life – specifically using Google Earth as a visualization engine for a variety of data types.

If you’re like me, you probably can’t wait to dig into the geospatial capabilities of ML6 and all that it has to offer.  You may wish to use the AppBuilder, a quick proof-of-concept application builder to ingest and display your unstructured content.  AppBuilder can slice and dice its way through your data without a line of code to create rich and compelling, entry-level applications.  This includes geospatial data as well; in fact, one of the sections of the Building Applications with MarkLogic 6 training course uses the AppBuilder to create a scuba dive log app, complete with Google Maps integration:

[<img class="aligncenter size-medium wp-image-527" alt="geopic1" src="/wp-content/uploads/2013/05/geopic1-300x225.png" width="300" height="225" />][2]

This technique works very well for creating a simple application and I was pleased to see how simple it was to assemble – again, without a line of code.

Recently I was finally assigned to a geospatial project for a local government customer.  The project was fairly straightforward – ingest their geo data and deliver a REST API for search across all content within their database.   Some of the searches involved geospatial queries – e.g. return all matches within 750 meters of a specified location, or all matches within this n-sided polygon.

As mentioned previously, ML6 has a cadre of features specifically geared towards answering these types of questions.  In fact, as far as MarkLogic is concerned, a geospatial query is just one of the many types of CTS (core text search) queries which can be combined with others (full-text search, date ranges, etc.) to create some truly complex searches.

While working on this project, it was important to ensure that the geospatial data was being loaded properly and that the geo queries I specified were returning the proper results.  The best way I know to verify my work is through visualization; I turned to the most accessible tool in my geo visualization toolbox, Google Earth.  There are two primary ways I load data into Google Earth – KML files and network links – essentially a network connection to a KML (or compressed KML known as a KMZ file) data source.

The deliverables for this project required REST responses in both XML and JSON formats.  With just a few additional lines of XQuery I was able to add KML output as one of the available REST output formats.

This post will describe two methods for displaying your MarkLogic geospatial data within Google Earth – both using network links.   Note that the information provided below is just one way to accomplish these results as MarkLogic provides five distinct index types for your geo data (see [Different Kinds of Geospatial Indexes][3] for details).

<span style="text-decoration: underline;">Before We Begin</span>

For the purposes of this post, we’ll assume that your data – including your geospatial coordinates – is already loaded into MarkLogic.  The structure of the documents I’ll be working with looks similar to this:

Single Point:


  <pre><code class="language-xml xml">&lt;doc xmlns="http://www.marklogic.com/MLU/logbook"&gt;
  &lt;uniqueID&gt;xyz123&lt;/uniqueID&gt;
	&lt;Place&gt;
	  &lt;Lat&gt;37.111&lt;/Lat&gt;
	  &lt;Lon&gt; -77.123&lt;/Lon&gt;
  &lt;/Place&gt;
	...
&lt;/doc&gt;
</code></pre>


&nbsp;

<span style="text-decoration: underline;">Verifying your Setup</span>

At this point with data loaded into MarkLogic and the geospatial indexes created, we should be able to use the Query Console as a quick test to verify that geo queries are working as expected.


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";
declare namespace ns = "http://www.marklogic.com/MLU/logbook";
let $center := cts:point(37.112, -77.121)
let $circle := cts:circle(0.2, $center)
let $pointQuery := 
  cts:element-pair-geospatial-query(
    xs:QName("ns:Place"),
    xs:QName(&ldquo;ns:Lat&rdquo;), 
    xs:QName(&ldquo;ns:Lon&rdquo;),
    $circle)
return 
  cts:search(fn:doc(), $pointQuery)
</code></pre>


&nbsp;

Executing the test code reveals that the geospatial indexes are working:


  <pre><code class="language-xml xml">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;results&gt;
  &lt;doc ns="http://www.marklogic.com/MLU/logbook"&gt;
    &lt;uniqueID&gt;xyz123&lt;/uniqueID&gt;
    &lt;Place&gt;
      &lt;Lat&gt;37.111&lt;/Lat&gt;
      &lt;Lon&gt;-77.123&lt;/Lon&gt;
    &lt;/Place&gt;
  &lt;/doc&gt;
&lt;/results&gt; 
</code></pre>


&nbsp;

Now let’s find a way to see the results.

<span style="text-decoration: underline;">Two Techniques for Visualizing  MarkLogic Geospatial Results in Google Earth</span>

The decision of which of the methods to use for visualizing your data in Google Earth is dependent upon a variety of factors specific to your data, installation and environment.  The table below summarizes the two techniques described in this post.

<table border="1" cellspacing="0" cellpadding="0">
  <tr>
    <td valign="top" width="122">
      <p align="center">
        <b> </b>
      </p>
    </td>
    
    <td valign="top" width="113">
      <p align="center">
        <b>Static Network Link</b>
      </p>
    </td>
    
    <td valign="top" width="131">
      <p align="center">
        <b>Dynamic Network Link</b>
      </p>
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="122">
      <b># Expected Results </b>
    </td>
    
    <td valign="top" width="113">
      100’s of points
    </td>
    
    <td valign="top" width="131">
      10,000’s of points +
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="122">
      <b>Data Update Frequency</b>
    </td>
    
    <td valign="top" width="113">
      Infrequent
    </td>
    
    <td valign="top" width="131">
      Frequent, Persistent
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="122">
      <b>Query Cost</b>
    </td>
    
    <td valign="top" width="113">
      Noticeable
    </td>
    
    <td valign="top" width="131">
      Negligible
    </td>
  </tr>
  
  <tr>
    <td valign="top" width="122">
      <b>Network Speed</b>
    </td>
    
    <td valign="top" width="113">
      Low
    </td>
    
    <td valign="top" width="131">
      High
    </td>
  </tr>
</table>

For the purpose of this post, a static network link is one in which the results are created one time and are only updated when specifically “refreshed” by the user whereas a dynamic network link is one that changes with some degree of frequency based on a trigger – either time-based (e.g. every 5 seconds) or in response to the user navigating around the globe.  Here we’re using the latter approach for the dynamic network link, i.e. our query will be re-run whenever the user navigates around the globe and the results will be limited to only those items falling with the bounds of the Google Earth window.

<span style="text-decoration: underline;">Technique Number One – Static Network Link</span>

Static network links are best use for static queries with limited results.  Using the dive log scenario above, an example of a static query would be “all dives by Joe during 2012”.  The results of this query would yield a list of dives that can be mapped within Google Earth with some fairly straightforward XQY.  For the purposes of this post we’ll assume that we have a REST endpoint which can receive the following parameters:

divername

startdate

enddate

format

Our sample REST call will look something like this:

http://hostname:portname/divelog?divername=joe&startdate=01/01/2012&enddate=12/31/2012&format=kml

Using the XQY code below, we’ll receive a KML file which will be loaded by Google Earth directly as a network link.  To do so, follow these three simple steps:

1)     Select the “Network Link” item from the “Add” menu

2)     Give the network link a helpful name, e.g. “Joe’s 2012 Dives”

3)     Provide the link to use for the KML (given above) and press “OK”

The results will be displayed within the Google Earth Window itself.  The layer can be expanded in the Places sidebar and each returned item can be clicked on for additional information.

<span style="text-decoration: underline;">Technique Number Two – Dynamic Network Link</span>

Dynamic network links are best with large volumes of data, spread out across a large area.  As we navigate with Google Earth, a REST request will be made to the MarkLogic database requesting all records within the boundaries of the view window.  For example, using the dive log scenario above, a dynamic query would be “all dives within the current viewport”.  For the purposes of this post we’ll use the same REST endpoint as indicated above, however, we need to accept a new parameter, BBOX, which will be provided by Google Earth, e.g.:

http://hostname:portname/divelog?BBOX=-81.5329,24.5999,-81.3718,24.7454&format=kml

The BBOX parameter specifies the lower-left (LL) and upper-right (UR) coordinates of the viewport in geospatial coordinates.  Specifically the order of the comma-separated values in the BBOX parameter is lower-left longitude, lower-left latitude, upper-right longitude, and upper-right latitude.

In order for Google Earth to provide these values to our REST endpoint, follow these steps:

1)     Select the “Network Link” item from the “Add” menu

2)     Give the network link a helpful name, e.g. “All Dives”

3)     Provide the link to use for the KML (given above)

4)     On the refresh tab, select “After Camera Stops” from the View-Based Refresh option

5)     Optionally adjust the delay after the camera stops before Google Earth will call our REST endpoint.

6)     Press OK

At this point, Google Earth will call the REST endpoint for the first load of data based on the current viewport.  Note that our XQuery code must validate the coordinates before processing as invalid views are possible – for example, if one or more corners of the screen are not on the Earth.  If everything worked correctly, your locations will be displayed on the map and the icon representing the dynamic network link will display a folder with a green indicator dot.  If the dot is red, the request was not successful.  Refer to the MarkLogic access and/or error log for further information.

As you navigate around the map, a new request will be sent to the REST endpoint for a list of the items in the current viewport.  Similar to the static link, the contents of a dynamic link may be expanded from the Places sidebar.  This will be refreshed automatically in response to each request.

[<img class="aligncenter size-medium wp-image-530" alt="geopic2" src="/wp-content/uploads/2013/05/geopic2-300x175.png" width="300" height="175" />][4]

Dynamic links can be combined with other search criteria in the same way as static links.  For example, you can create a dynamic link for only Joe’s dives, or only dives during 2012.  In addition, multiple network links can be enabled simultaneously allowing for various geospatial mash-ups of your data.

<span style="text-decoration: underline;">Conclusion</span>

MarkLogic 6 has tremendous capabilities for geospatial data management and can be combined with a powerful visualization tool like Google Earth to deliver a geospatial solution with minimal development effort.  In future posts, I hope to write about customizing your KML results to provide results which will be more reflective of your data – e.g. icon shapes and colors representing some particular value range.

Feel free to contact me for any comments or questions ([Mark Ferneau][1])

<span style="text-decoration: underline;">Code Samples</span>

The simplified XQuery source code below supports both static and dynamic KML Network Links, the only difference is that Google Earth will provide the BBOX parameter for dynamic links.

For simplicity, only the geospatial component of the query is provided here – other criteria (diver name, date range, etc.) can be added.


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

declare namespace ns = "http://www.marklogic.com/MLU/logbook";
declare variable $bbox as xs:string? := xdmp:get-request-field("BBOX", ());

xdmp:set-response-content-type("text/kml; charset=utf-8"),
&lt;kml xmlns="http://www.opengis.net/kml/2.2" xmlns:gx="http://www.google.com/kml/ext/2.2"&gt;
&lt;Document&gt;
{

  (: If the bounding box was provided by Google Earth, use it for a geo query :)
  let $pointQuery := 
    if ($bbox) then  
      let $lon1 := fn:number(fn:tokenize($bbox, ",")[1])
      let $lon2 := fn:number(fn:tokenize($bbox, ",")[3])
      let $lat1 := fn:number(fn:tokenize($bbox, ",")[2])
      let $lat2 := fn:number(fn:tokenize($bbox, ",")[4])
      let $west := fn:min(($lon1, $lon2))
      let $east := fn:max(($lon1, $lon2))
      let $north := fn:max(($lat1, $lat2))
      let $south := fn:min(($lat1, $lat2))
  
      let $boundingBox := cts:box($south, $west, $north, $east)
      return cts:element-pair-geospatial-query( xs:QName("ns:Place"), 
   xs:QName("ns:Lat"), 
   xs:QName("ns:Lon"), 
   $boundingBox)
    else
      ()
   
    (: Perform the search - only geo search for simplicity :)
    let $docs := cts:search(fn:doc(), $pointQuery, "unfiltered")
  
    (: KML is Lon, Lat, Alt :)
    for $doc in $docs
      return
        &lt;Placemark&gt;
          &lt;name&gt;{$doc//ns:Site/text()}&lt;/name&gt;
          &lt;description&gt;
            {$doc//ns:DiverLName/text()}, {$doc//ns:DiverFName/text()}: 
            {$doc//ns:Divedate/text()}&lt;p&gt;{$doc//ns:Comments/text()}
          &lt;/description&gt;
          &lt;Point&gt;
            &lt;coordinates&gt;{$doc//ns:Lon/text()}, {$doc//ns:Lat/text()}&lt;/coordinates&gt;
          &lt;/Point&gt;
        &lt;/Placemark&gt;
}
&lt;/Document&gt;
&lt;/kml&gt;
</code></pre>


<span style="text-decoration: underline;">References</span>

1)     KML Standard, Open Geospatial Foundation, <http://www.opengeospatial.org/standards/kml>

2)     Geospatial Search Applications, MarkLogic Search Developer’s Guide, <http://docs.marklogic.com/guide/search-dev/geospatial#chapter>

3)     Different Kinds of Geospatial Indexes, MarkLogic Search Developer’s Guide, <http://docs.marklogic.com/guide/search-dev/geospatial#id_57072>

&nbsp;

 [1]: http://www.linkedin.com/in/ferneau
 [2]: /wp-content/uploads/2013/05/geopic1.png
 [3]: http://docs.marklogic.com/guide/search-dev/geospatial#id_57072
 [4]: /wp-content/uploads/2013/05/geopic2.png