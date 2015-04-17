---
title: XQuery and HTML5
author: Dave
layout: post
permalink: /2011/12/03/xquery-and-html5/
categories:
  - Experiments
  - Software Tip
tags:
  - HTML5
  - Mobile
  - MVC
  - XQuery
  - XTHML
---
[<img class="alignleft size-medium wp-image-55" title="xqueryhtml5" src="/wp-content/uploads/2011/12/xqueryhtml51-e1322935113762-229x300.png" alt="XQuery generated HTML5" width="229" height="300" />][1]XQuery is amazing at generating server-side dynamic XHTML.  PHP, Java, .Net and the like are good too but don&#8217;t have the advantage of a seamless connection to a storage model.  However, they do hav a big advantage over XQuery when it comes to HTML5 because they can serialize non-XML compliant HTML text.

With XQuery and XHTML the following code works just fine:

<pre>xquery version "1.0";
xdmp:set-response-content-type("text/html; charset=UTF-8"),
'&lt;!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"&gt;',
&lt;html xmlns="http://www.w3.org/1999/xhtml"&gt;
    &lt;head&gt;
        &lt;meta name="description" content="Awesome" /&gt;
        &lt;title&gt;Title&lt;/title&gt;
        &lt;script type="text/javascript" src="js/app.js" /&gt;
...</pre>

The advantage of building your XHTML in XQuery is now we can stick this code in a &#8220;View&#8221; as part of a Model-View-Controller software pattern.  I can write the view once and reuse it as a code component throughout the project, or across multiple projects.  However HTML5, the latest and greatest web technology with tons of momentum in the mobile web arena, prefers serialization that looks something more like the following:

<pre>&lt;!doctype html&gt;
&lt;html&gt;
    &lt;head&gt;
        &lt;meta content="description" content="Awesome"&gt;
        &lt;title&gt;Title&lt;/title&gt;
        &lt;script src="js/app.js"&gt;
...</pre>

This is a problem for XQuery.  The un-closed tags are invalid XML, so XQuery causes an evaluation error that looks something like:

<pre>XDMP-UNEXPECTED: (err:XPST0003) Unexpected token syntax error, unexpected $end, expecting EndTagOpen_</pre>

The problem is that HTML not only allows for un-closed tags like <script> and <meta>, but actually almost *requires* them.  I believe this traces back to the SGML origins of HTML.  The following HTML creation attempts in XQuery are not parsed correctly by browsers:

<pre>&lt;script src="js/app.js"/&gt;   or   &lt;script src="js/app.js"&gt;&lt;/script&gt;</pre>

Because the XQuery running in MarkLogic is running in the &#8220;default&#8221; XML serialization mode, both of the script tags above are collapsed to the self closed tag

<pre>&lt;script src="js/app.js"/&gt;</pre>

This is a problem for many browsers which will tend to include the rest of the HTML body as a child of the script tag, either causing the read of the javascript in the header no to be parsed or actually eating the rest of the page and never rendering the body DOM.  I have found two solutions: Either write your XQuery a non-empty script tags so that XQuery won&#8217;t collapse them:

<pre>&lt;script src="js/app.js"&gt;&nbsp;&lt;/script&gt;</pre>

or change the output serialization to HTML mode in the declare section at the top of your script:

<pre>declare option xdmp:output "method=html";</pre>

You&#8217;ll have to remember to do this for every module that generates HTML because other tags, like <textarea>, have the same pitfall.  You could also do this for a MarkLogic app server across the board by changing it&#8217;s settings in the admin console.  I prefer the declare and see it as just another reason to have a separate &#8220;View&#8221; section of your MVC app.

The next hurdle to tackle is getting inline javascript and [JQuery plugins and templating libraries][2] which like putting curly braces { } in HTML attribute values to work well.  I&#8217;ll leave that for another day.  Here is my current take at an XQuery [HTML5 Boilerplate][3] view library:

<pre>xquery version "1.0-ml";

module namespace v-h5bp = "http://framework/view/v-h5bp";

(:  Stick this at the top of any module that generates HTML so that
    empty tags don't get truncated to non-empty tags :)
declare option xdmp:output "method=html";

(:
    HTML5 Template
    @author Dave http://www.front2backdev.com
    XQuery adaptation Public Domain.
        html5 boilerplate: Public Domain
        jQuery: MIT/GPL license
        Modernizr: MIT/BSD license
        Respond.js: MIT/GPL license
        Normalize.css: Public Domain

    Basis of this code is the HTML5 BOILERPLATE project
    Checkout http://html5boilerplate.com/mobile
:)

(:
    HTML5 Mobile Boilerplate layout

    $title -- The html head title of the page
    $script -- extra HTML5 nodes for the html head
        carful with your self-closed tags like &lt;script/&gt;, these may not parse correctly
        in browsers.  Instead make sure XML serialization creates something like the following:
            &lt;script src="url to source"&gt;&lt;/script&gt;
        If you don't put something inside the tag, MarkLogic will serialize this to
            &lt;script src="url to source"/&gt;
        which is supported by xhtml but not html5 parsers
    $html  -- HTML5 nodes to put in the body
    $meta-description -- meta description text
    $meta-content -- meta conent text
    $goog-analytics-id -- your UA-* Id for google analytics
    $cache-version -- iterate this variable to ignore cached CSS and JS
:)
declare function v-h5bp:mbp-page-layout(
    $title as xs:string,
    $script as node()*,
    $html as node()*,
    $meta-description as xs:string?,
    $meta-content as xs:string?,
    $google-analytics-id as xs:string?,
    $cache-version as xs:double
) {

xdmp:set-response-content-type("text/html"),
'&lt;!doctype html&gt;',
&lt;!-- Conditional comment for mobile ie7 http://blogs.msdn.com/b/iemobile/ --&gt;,
&lt;!--[if IEMobile 7 ]&gt;    &lt;html class="no-js iem7"&gt; &lt;![endif]--&gt;,
&lt;!--[if (gt IEMobile 7)|!(IEMobile)]&gt;&lt;!--&gt;, &lt;html class="no-js"&gt; &lt;!--&lt;![endif]--&gt;
&lt;head&gt;
    &lt;meta charset="utf-8"/&gt;

    &lt;title&gt;{$title}&lt;/title&gt;
    &lt;meta name="description" content="{$meta-description}"/&gt;
    &lt;meta name="author" content="{$meta-content}"/&gt;

    &lt;meta name="HandheldFriendly" content="True"/&gt;
    &lt;meta name="MobileOptimized" content="320"/&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"/&gt;

    &lt;link rel="apple-touch-icon-precomposed" sizes="114x114" href="img/h/apple-touch-icon.png"/&gt;
    &lt;link rel="apple-touch-icon-precomposed" sizes="72x72" href="img/m/apple-touch-icon.png"/&gt;
    &lt;link rel="apple-touch-icon-precomposed" href="img/l/apple-touch-icon-precomposed.png"/&gt;
    &lt;link rel="shortcut icon" href="img/l/apple-touch-icon.png"/&gt;

    &lt;meta name="apple-mobile-web-app-capable" content="yes"/&gt;
    &lt;meta name="apple-mobile-web-app-status-bar-style" content="black"/&gt;
    &lt;script&gt;/* &lt;![CDATA[ */(function(a,b,c){if(c in b&&b[c]){var d,e=a.location,f=/^(a|html)$/i;a.addEventListener("click",function(a){d=a.target;while(!f.test(d.nodeName))d=d.parentNode;"href"in d&&(d.href.indexOf("http")||~d.href.indexOf(e.host))&&(a.preventDefault(),e.href=d.href)},!1)\}\})(document,window.navigator,"standalone")/* ]]&gt; */&lt;/script&gt;
    &lt;link rel="apple-touch-startup-image" href="img/l/splash.png"/&gt;

    &lt;meta http-equiv="cleartype" content="on"/&gt;
    &lt;link rel="stylesheet" href="/css/style.css?v={$cache-version}"/&gt;

    &lt;script src="js/libs/modernizr-custom.js"&gt;&nbsp;&lt;/script&gt;
    &lt;script&gt;/* &lt;![CDATA[ */Modernizr.mq('(min-width:0)') || document.write('&lt;script src="js/libs/respond.min.js"&gt;\x3C/script&gt;')/* ]]&gt; */&lt;/script&gt;

    {$script}  

&lt;/head&gt;

&lt;body&gt;

    {$html}

    &lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.min.js"&gt;&nbsp;&lt;/script&gt;
    &lt;script&gt;/* &lt;![CDATA[ */window.jQuery || document.write('&lt;script src="js/libs/jquery-1.6.2.min.js"&gt;&lt;\/script&gt;')/* ]]&gt; */&lt;/script&gt;

    &lt;script src="js/script.js?v={$cache-version}"&gt;&lt;/script&gt;
    &lt;script src="js/mylibs/helper.js"&gt;&lt;/script&gt;
    &lt;script&gt;/* &lt;![CDATA[ */MBP.scaleFix();/* ]]&gt; */&lt;/script&gt;

    &lt;script&gt;
      var _gaq=[["_setAccount","{$google-analytics-id}"],["_trackPageview"]];
      (function(d,t)\{\{var g=d.createElement(t),s=d.getElementsByTagName(t)[0];g.async=1;
        g.src=("https:"==location.protocol?"//ssl":"//www")+".google-analytics.com/ga.js";
        s.parentNode.insertBefore(g,s)\}\}(document,"script"));
    &lt;/script&gt;

&lt;/body&gt;
&lt;/html&gt;

};</pre>

<pre></pre>

&nbsp;

* * *

Updates:

In the effort of leaving around the best code samples possible I will likely be updating my posts as I learn better / cleaner ways of working with XQuery.  Here&#8217;s to progressive enhancement!

12/3/2011 : Using XQuery HTML output serialization to avoid empty tag collapsing.

12/4/2011 : Forgot helper script, better inlining of google analytics code.  Note double \{\{ and \}\} which XQuery serializes to a single { and } .

 [1]: /wp-content/uploads/2011/12/xqueryhtml51-e1322935113762.png
 [2]: http://learn.knockoutjs.com/#/?tutorial=templates
 [3]: http://html5boilerplate.com/