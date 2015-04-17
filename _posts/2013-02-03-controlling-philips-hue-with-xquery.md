---
title: Controlling Philips Hue with XQuery
author: Dave
layout: post
permalink: /2013/02/03/controlling-philips-hue-with-xquery/
categories:
  - Experiments
tags:
  - JSON
  - MarkLogic
  - Phillips Hue
  - REST
  - XQuery
---
Playing around with a set of Philips Hue light bulbs.  Turns out you can control them with simple REST calls.

[l-mlhue.xqy][1] library on github

Simple Controller:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

import module namespace lh = "http://derickson/lib/l-mlhue"
  at "/lib/l-mlhue.xqy";

for $i in (1 to 10)
for $state in (
  $lh:RED, $lh:MAGENTA, $lh:PURPLE, 
  $lh:BLUE, $lh:GREEN, $lh:YELLOW, 
  $lh:ORANGE
)
return (
  lh:put-all-state( $state ),
  xdmp:sleep(1000)
)


</code></pre>


Results
<iframe width="560" height="315" src="https://www.youtube.com/embed/gN7UR7CFueg" frameborder="0" allowfullscreen></iframe>


Thanks to: <http://rsmck.co.uk/hue> and <http://blog.ef.net/2012/11/02/philips-hue-api.html>

 [1]: https://github.com/derickson/ml-utils/blob/master/xquery/lib/l-mlhue.xqy