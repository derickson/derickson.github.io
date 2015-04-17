---
title: Turn OFF Automatic Directory Creation
author: Dave
layout: post
permalink: /2011/12/05/turn-off-automatic-directory-creation/
categories:
  - Software Tip
tags:
  - locking
  - MarkLogic
  - settings
---
[<img class="aligncenter size-full wp-image-66" title="directorycreation" src="/wp-content/uploads/2011/12/directorycreation.jpg" alt="directory creation: manual" width="558" height="47" />][1]

Consider this an opinionated pro-tip.  When you create a new database in MarkLogic there is a setting called *directory creation* that defaults to &#8220;automatic.&#8221;  You don&#8217;t want that for your content databases.  You want &#8220;manual.&#8221;

*Directory creation:* automatic creates directory property fragments for inferred parent folders as you insert documents into URIs that resemble a file system path structure.  The property fragments exist for the explicit purpose of supporting WebDAV access to the database.  Setting it to manual does not affect the behavior of cts:directory-query(), xdmp:directory(), or other &#8220;directory&#8221; XQuery functions.

The problem with leaving this setting in &#8220;automatic&#8221; mode is that the inferred parent directory property fragments will get involved in locking anytime you touch a URI during an update transaction.  Because it is highly likely that any two documents in your database will share a directory in their inferred parentage, say &#8220;/&#8221; for example, you are now effectively getting into locking contention every time two activities in your application want to write in parallel.  You don&#8217;t want this.  You want a &#8220;shared-nothing&#8221; architecture that minimizes locking.

So set *directory creation* to &#8220;manual&#8221; immediately after you create your content databases so you don&#8217;t have to delete the directory property fragments after you have loaded all your data.  Giving up WebDAV on content databases is far preferable to having performance problems once you start testing with multiple concurrent users or multi-threaded data loaders.

While you are at it, turn on the *URI Lexicon* and disable *maintain last modified* unless you need it.  The *URI Lexicon* is a requirement for using cts:uris() and working with tools like XQSync and CoRB, which you will most likely need later in your project.  *Maintain last modified* creates document property fragments.  If you weren&#8217;t doing anything with these property fragments you should probably turn this off to keep the number of fragments in your database down.

&#8212; Dave

 [1]: /wp-content/uploads/2011/12/directorycreation.jpg