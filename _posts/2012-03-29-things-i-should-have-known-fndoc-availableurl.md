---
title: 'Things I should have known: fn:doc-available($url)'
author: Dave
layout: post
permalink: /2012/03/29/things-i-should-have-known-fndoc-availableurl/
categories:
  - Thing I Should Have Known
tags:
  - filtering
  - MarkLogic
  - performance
  - XQuery
---
Sometimes you run across something in a technology or domain that you really should have known long long ago but didn&#8217;t.  This is one of those things.  Thank you to the person who pointed it out to me.

fn:exists(fn:doc($url)) causes MarkLogic to pull a fragment from storage to &#8220;post-filter&#8221; confirm the existence of a document node.  The faster way of doing this is fn:doc-available($url) or xdmp:exists(fn:doc($url)).

xdmp:exists( &#8230; ) is basically equivalent to xdmp:estimate( &#8230; ) gt 0

The way to confirm for yourself that one has less impact on storage than the other is to turn on xdmp:query trace before running each command.  Compare the MarkLogic error log after running the following

<pre>xquery version "1.0-ml";
xdmp:document-insert( "/1.xml", &lt;a&gt;1&lt;/a&gt;);</pre>

<pre>xquery version "1.0-ml";
xdmp:query-trace(fn:true()),
fn:exists(fn:doc("/1.xml"));</pre>

<pre>xquery version "1.0-ml";
xdmp:query-trace(fn:true()),
fn:doc-available("/1.xml")</pre>

The error log should have something like this:

<pre>Analyzing path: fn:doc("/1.xml")
Step 1 is searchable: fn:doc("/1.xml")
Path is fully searchable.
Gathering constraints.
Step 1 contributed 1 constraint: fn:doc("/1.xml")
Executing search.
<strong>Selected 1 fragment to filter</strong></pre>

<pre>Analyzing path: fn:doc("/1.xml")
Step 1 is searchable: fn:doc("/1.xml")
Path is fully searchable.
Gathering constraints.
Step 1 contributed 1 constraint: fn:doc("/1.xml")
Executing search.</pre>

It&#8217;s that &#8220;Selected 1 fragment to filter&#8221; that tells touching the storage was necessary for fn:exists().  You want to avoid this for most storage solutions.  Interestingly enough, this is true whether or not I turn on the URI Lexicon.
