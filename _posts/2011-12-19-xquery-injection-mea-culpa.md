---
title: XQuery Injection Mea Culpa
author: Dave
layout: post
permalink: /2011/12/19/xquery-injection-mea-culpa/
categories:
  - Software Tip
tags:
  - MarkLogic
  - security
  - XQuery
---
There was a paper at the 2011 Balisage about XQuery Injection attacks.  The paper focuses on attacks against eXist but got me thinking.  
<http://www.balisage.net/Proceedings/vol7/html/Vlist02/BalisageVol7-Vlist02.html>

Usually when we talk about Injection attacks at ML, we focus on xdmp:eval() and xdmp:value() and making sure that the input string is not derived from user supplied data.

With injection attacks in mind, I turned my attention to source code I wrote for a project going live this month and found the following:

<pre>let $uri := xdmp:get-request-field($cfg:URI_PARAM, ())
let $doc := fn:doc($uri)/element()
return
   v:render( $doc )</pre>

So the web client clicks on a link  something like

<pre>&lt;a href="/page/render?uri=%2Fdoc%2F1.xml"&gt;link&lt;/a&gt;</pre>

and gets an XHTML rendering of the document at URI &#8220;/doc/1.xml&#8221;.   The only things in the content database of this HTTP server are  <doc:doc> XML documents which 100% of the user base are allowed to see and <audit:audit> XML audit logs, which are really meant only for admins.  It may not the biggest attack vulnerability but a user could URL-BASH to see other user&#8217;s audit logs.    Making the following change solves the problem:

<pre>let $doc := fn:doc($uri)/doc:doc</pre>

I then got paranoid and put in 2 or 3 other safeguards, but you get my point.  Here are some tips I have compiled:

  * Watch out for passing user contributed strings into fn:doc(), fn:collection() as well as xdmp:eval() and xdmp:value() if you aren&#8217;t using document permissions for security.
  * Prefer XPath to fn:doc() when selecting documents for update.
  * Never store a user&#8217;s password as plaintext in XML.  If you aren&#8217;t using the security database users, 1-way hash+salt those passwords or database hipsters will call you names.
  * Throw in an extra xdmp:request-timestamp() value assert to force read-only transactions where appropriate.  This is most likely to help protect you down the road in O&M when others are quickly modifying the code base to solve problems or add features.

Citation:

van der Vlist, Eric. “XQuery Injection: Easy to exploit, easy to prevent&#8230;.” Presented at Balisage: The Markup Conference 2011, Montréal, Canada, August 2 &#8211; 5, 2011. In *Proceedings of Balisage: The Markup Conference 2011*. Balisage Series on Markup Technologies, vol. 7 (2011). doi:10.4242/BalisageVol7.Vlist02.