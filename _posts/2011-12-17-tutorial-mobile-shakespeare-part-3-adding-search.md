---
title: 'Tutorial: Mobile Shakespeare (Part 3 &#8211; Adding Search)'
author: Dave
layout: post
permalink: /2011/12/17/tutorial-mobile-shakespeare-part-3-adding-search/
categories:
  - Tutorial
tags:
  - HTML5
  - JQuery Mobile
  - MarkLogic
  - Mobile
  - Mobile Web
  - mobshake
  - phrase search
  - Phrase-Around
  - Phrase-Through
  - REST
---
[<img class="alignleft size-full wp-image-284" title="searchscreen" src="/wp-content/uploads/2011/12/searchscreen.png" alt="Mobile Shakespeare search screen" width="276" height="537" />][1]In the [last part of this tutorial][2] we skinned the Mobile Shakespeare app to be more memorable and distinctive.  Now it&#8217;s time to add some search functionality.

The complete code base for this sample is now up in gitub for your reference:

<a href="https://github.com/derickson/shake/tree/master/xquery2" target="_blank">github/derickson/shake/xquery2</a>

I&#8217;ve cleaned up the XQuery for readability and will be linking only portions of the code here.

This is the app we are going to build: *BROKEN LINK TO DEMO*

## REST

First we&#8217;ll set up a few new REST targets in /lib/config.xqy


  <pre><code class="language-xquery xquery">&lt;get path="play/:id/act/:act/scene/:scene/speech/:speech"&gt;&lt;to&gt;play#scene&lt;/to&gt;&lt;/get&gt;
        
&lt;get path="search"&gt;&lt;to&gt;search#get&lt;/to&gt;&lt;/get&gt;
&lt;post path="search"&gt;&lt;to&gt;search#get&lt;/to&gt;&lt;/post&gt;</code></pre>


Note, I&#8217;ve added the ability to jump to a specific SPEECH back in the /play resource.  You can check out this code yourself in the github copy of the code.

## Search Resource

Next we&#8217;ll make a new resource for the search REST targets in /resource/search.xqy .  At the top of this file we are going to import the MarkLogic Search API, which is a high level XQuery library that sits ontop of the core MarkLogic search function in the cts:* library.  The Search API is a great place to start when building XQuery web apps because it does so much for you.  Like all high level tools, you may eventually outgrow parts of the Search API and decide to use the cts:functions directly ( I do this all the time for intricate multi-tiered facets).  The cts functions are incredibly easy to compose and work much like boolean functions, but for now we&#8217;ll stick to the Search API.  So &#8230; that import statement


  <pre><code class="language-xquery xquery">import module namespace search = "http://marklogic.com/appservices/search" at "/MarkLogic/appservices/search/search.xqy";
</code></pre>


The most basic call of the Search API is

<pre>search:search( $searchTerm, $options)</pre>

## The Search API $options parameter

The $searchTerm is a one-line string that fits a grammar specified in the second parameter, $options.  If you omit the options XML parameter the Search API defaults do a good job of emulating a &#8220;Google-like&#8221; search syntax, but I want to make some modifications.  Rather than start from scratch learning how to construct the XML that make up these option we can get the Search API defaults to use as a starting point by calling the following function in the Query Console or cq. (Don&#8217;t forget the import of the Search API module)

<pre>search:get-default-options()</pre>

You can start editing from the Search API standard functionality.  Here is my finished $options object:


  <pre><code class="language-xquery xquery">(: Search API options :)
declare variable $options :=
    &lt;options xmlns="http://marklogic.com/appservices/search"&gt;
        &lt;concurrency-level&gt;8&lt;/concurrency-level&gt;
        &lt;debug&gt;0&lt;/debug&gt;
        &lt;page-length&gt;10&lt;/page-length&gt;
        &lt;search-option&gt;score-logtfidf&lt;/search-option&gt;
        &lt;quality-weight&gt;1.0&lt;/quality-weight&gt;
        &lt;return-constraints&gt;false&lt;/return-constraints&gt;
        &lt;!-- Turning off the things we don't use --&gt;
        &lt;return-facets&gt;false&lt;/return-facets&gt;
        &lt;return-qtext&gt;false&lt;/return-qtext&gt;
        &lt;return-query&gt;false&lt;/return-query&gt;
        &lt;return-results&gt;true&lt;/return-results&gt;
        &lt;return-metrics&gt;false&lt;/return-metrics&gt;
        &lt;return-similar&gt;false&lt;/return-similar&gt;
        &lt;searchable-expression&gt;//SPEECH&lt;/searchable-expression&gt;
        &lt;sort-order direction="descending"&gt;
            &lt;score/&gt;
        &lt;/sort-order&gt;
        &lt;term apply="term"&gt;
            &lt;!-- "" $term returns no results --&gt;
            &lt;empty apply="no-results" /&gt;
            &lt;!-- Not sure why this isn't a default --&gt;
            &lt;term-option&gt;case-insensitive&lt;/term-option&gt;
        &lt;/term&gt;
        &lt;grammar&gt;
            &lt;quotation&gt;"&lt;/quotation&gt;
            &lt;implicit&gt;
                &lt;cts:and-query strength="20" xmlns:cts="http://marklogic.com/cts"/&gt;
            &lt;/implicit&gt;
            &lt;starter strength="30" apply="grouping" delimiter=")"&gt;(&lt;/starter&gt;
            &lt;starter strength="40" apply="prefix" element="cts:not-query"&gt;-&lt;/starter&gt;
            &lt;joiner strength="10" apply="infix" element="cts:or-query" tokenize="word"&gt;OR&lt;/joiner&gt;
            &lt;joiner strength="20" apply="infix" element="cts:and-query" tokenize="word"&gt;AND&lt;/joiner&gt;
            &lt;joiner strength="30" apply="infix" element="cts:near-query" tokenize="word"&gt;NEAR&lt;/joiner&gt;
            &lt;joiner strength="30" apply="near2" consume="2" element="cts:near-query"&gt;NEAR/&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint"&gt;:&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint" compare="LT" tokenize="word"&gt;LT&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint" compare="LE" tokenize="word"&gt;LE&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint" compare="GT" tokenize="word"&gt;GT&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint" compare="GE" tokenize="word"&gt;GE&lt;/joiner&gt;
            &lt;joiner strength="50" apply="constraint" compare="NE" tokenize="word"&gt;NE&lt;/joiner&gt;
        &lt;/grammar&gt;
        &lt;!-- Custom rendering code for "Snippet" --&gt;
        &lt;transform-results apply="snippet" ns="http://framework/lib/l-util" at="/lib/l-util.xqy" /&gt;
    &lt;/options&gt;;</code></pre>


Let&#8217;s go through it.  I turn off return of data I won&#8217;t be using for rendering.  For example my app has no facets:

<pre>&lt;!-- Turning off the things we don't use --&gt;
&lt;return-facets&gt;false&lt;/return-facets&gt;</pre>

I want our search results to be the SPEECH tags inside the PLAY root elements.  In cts we would specify a &#8220;searchable expression&#8221; as the first param of cts:search. In the search API we add the following:

<pre>&lt;searchable-expression&gt;//SPEECH&lt;/searchable-expression&gt;</pre>

Lastly i change some of the default text term options.  When a user doesn&#8217;t type anything, I&#8217;ll omit executing the search, rather than just pass back the first SPEECH in document order in the repository (which they can do from the Play button on the new home page anyways).  Also when testing the app I found that searches for &#8220;My kingdom for a horse&#8221; returned zero results.  That&#8217;s because the default for the Search API is &#8220;case-sensitive&#8221;.  (Note, it&#8217;s a good idea to turn on the index for fast case sensitive search if this is what you want)  But my user&#8217;s might type in &#8220;My&#8221; kingdom for a horse, so I&#8217;ll set a term option to case-insensitive:

<pre>&lt;term apply="term"&gt;
    &lt;!-- "" $term returns no results --&gt;
    &lt;empty apply="no-results" /&gt;
    &lt;!-- Not sure why this isn't a default --&gt;
    &lt;term-option&gt;case-insensitive&lt;/term-option&gt;
&lt;/term&gt;</pre>

The last step is to specify a custom snippeting library function.  The Search API assumes I am searching on text that is too large to present in a result, but I&#8217;d like my users to see the whole SPEECH in order to give the highlighted words context.  I&#8217;ll let you look at the highlight code yourself in github under /lib/l-util.xqy, but the portion of the $options that specifies which code to use is:

<pre>&lt;!-- Custom rendering code for "Snippet" --&gt;
&lt;transform-results apply="snippet"
    ns="http://framework/lib/l-util"
    at="/lib/l-util.xqy" /&gt;</pre>

## Wrapping up the Page

Next I&#8217;ll need a good search form.  I decided to have a Phrase Search toggle switch because mobile users often don&#8217;t have the quotatin marks of the standard Search API grammar on their keyboards without going to a SHIFT alternate keyboard:


  <pre><code class="language-xquery xquery">&lt;!-- Search Form --&gt;
                &lt;form action="/search" method="get" data-transition="fade" class="ui-body ui-body-b ui-corner-all"&gt;
                    &lt;fieldset &gt;
                        &lt;label for="search-basic"&gt;Search all lines:&lt;/label&gt;
                        &lt;input type="search" name="term" id="term" value="{$term}" data-theme="b" /&gt;    
                    &lt;/fieldset&gt;
                    &lt;div data-role="fieldcontain"&gt;
                        &lt;label for="slider2"&gt;Phrase search:&lt;/label&gt;
                        &lt;select name="phrase" id="phrase" data-role="slider" &gt;
                            &lt;option value="off"&gt;
                                Off
                            &lt;/option&gt;
                            &lt;option value="on"&gt;
                                {
                                    (: Dynamic inline attribute of the option element :)
                                    if($phrase eq "on") then 
                                        attribute selected {"selected"} 
                                    else 
                                        ()
                                }
                                On
                            &lt;/option&gt;
                        &lt;/select&gt;
                    &lt;/div&gt;
                    &lt;button type="submit" data-theme="b" data-transition="fade"&gt;Submit&lt;/button&gt;
                &lt;/form&gt;</code></pre>


And I&#8217;ll actually need to call the search.  I pass the Search API results XML object to a transform function which you can go through on github:


  <pre><code class="language-xquery xquery">&lt;p&gt;
            {
                (: Search Results Area :)
                
                (: 
                 Modify the typed search term.  
                 Add Quotes if the $phrase flag is "on"
                 If the term is empty sequence, use ""
                :)
                let $searchTerm := 
                    if(fn:exists($term)) then 
                        if($phrase eq "on" and fn:not( fn:starts-with($term,'"') and fn:ends-with($term,'"'))) then
                            fn:concat('"',$term,'"')
                        else
                            $term
                    else 
                        ""
                return
                
                    (: 
                      Think Functionally ...
                      XQuery invokes passes the evaluation of search:search
                      to transform-results
                    :) 
                    
                    (: transform results into HTML5 :)
                    local:transform-results( 
                        (: execute the search with the Search API :)
                        search:search($searchTerm, $options)//search:result 
                    )
            }
            &lt;/p&gt;</code></pre>


So now in one XQuery script I have a basic search app that is already optimized for Mobile browsers.  I&#8217;m happy with the performance of the site on my relatively new Android phone, but I imagine we&#8217;d want to pay close attention to some of the HTML can can caching response headers being returned from MarkLogic given that the content is completely static .  Here are some items for improvement and exploration I could think of:

  * Check performance after &#8220;Phonegapping&#8221; the HTML5 into a native iOS or Android App
  * Add the HTML5 meta tags for specifying an Apple icon when this site is bookmarked on iOS home screen.
  * Add paging to the search screen
  * Add the ability to search within a specific play (this could be done quickly with a Search API constraint and a drop down)
  * Allow users to &#8220;star&#8221; lines as their favorites (no login really necessary) and put links to the most popular lines on the Mobile Shakespeare home screen.

## Phrase-Through and Phrase-Around

However, instead of spending time on that, let&#8217;s tune the MarkLogic indices slightly.  One of the strengths of MarkLogic over other full text search indexers is that MarkLogic preserves structure.  As a result, MarkLogic can do inferred metadata search and full text search out of the same indices without extra configuration or integration (it scales well too!). Here&#8217;s one my my favorite lines from Macbeth

[<img class="alignnone size-full wp-image-286" title="ripped" src="/wp-content/uploads/2011/12/ripped.jpg" alt="Quote From Macbeth Act 5 Scene 8" width="381" height="268" />][3]

Some search engines flatten the text in their documents before generating search indexes.  This makes resolving relevance based on the surrounding XML or HTML tags very difficult imagine searching for the phrase &#8220;Untimely ripp&#8217;d Accursed be&#8221; in the following examples:


  <pre><code class="language-html html">1.)
&lt;div&gt;... was &lt;b&gt;untimely&lt;/b&gt; ripp'd. Accursed be ...&lt;/div&gt;

2.)
&lt;div&gt;&lt;p&gt;... was untimely ripp'd.&lt;/p&gt;&lt;p&gt;Accursed be ...&lt;/p&gt;

3.)
&lt;div&gt;... was untimely
  &lt;annotation class="hidden tooltip"&gt;Awesome!&lt;/annotation&gt;
ripp'd.  Accursed b...&lt;/div&gt;

4.)
&lt;SPEECH&gt;
  &lt;LINE&gt;Tell thee, Macduff was from his mother's womb&lt;LINE&gt;
  &lt;LINE&gt;Untimely ripp'd.&lt;/LINE&gt;
&lt;/SPEECH&gt;
&lt;SPEECH&gt;
  &lt;LINE&gt;Accursed be the tongue ...</code></pre>


The first sample might lead you to believe that flattening the text is a good idea.  By ignoring all structure, we get a relevant result because the words &#8220;untimely,&#8221; &#8220;ripp&#8217;d,&#8221; &#8220;accursed,&#8221; and &#8220;be&#8221; are adjacent.  The <b> tags represent style, not substance!  MarkLogic has something called a &#8220;Phrase-Through&#8221; index setting which tells the indexer to determine phraases through an element separation.  Obvious examples from XHTML, wordml, and other common namespaces come preconfigured so you don&#8217;t have to worry about them.

The second sample destroys the notion of text flattening.  Ripp&#8217;d and Accursed aren&#8217;t just in different sentences (as the period might inform some indexers), they are in different paragraphs and do not form a semantic &#8220;phrase&#8221;.  MarkLogic won&#8217;t Phrase-Through a <p> tag unless we tell it to so we get the correct behavior.

The third sample above is trickier.  Embedded into the text is markup that represents an inline annotation of the semi-structued data.  If I flatten the text, the word &#8220;Awesome&#8221; messes up our phrase, but a default parse of the XML structure also breaks up the semantics of the &#8220;untimely ripp&#8217;d&#8221; phrase.  MarkLogic can solve this with a &#8220;Phrase-Around&#8221; on the annotation tag.  A Phrase-Around setting tells MarkLogic to indexer to link the words before and after into a phrase but ignore the words inside.

The fourth sample is our data from the Shakespeare XML demo.  To get a good phrase search on &#8220;was from his mother&#8217;s womb untimely ripp&#8217;d&#8221; we need to set up a Phrase-Through on the LINE element in the Databases > shake > Phrase-Throughs setting on the MarkLogic admin menu.  Once we&#8217;ve done this a phrase enabled search for &#8220;was from his mother&#8217;s womb untimely ripp&#8217;d&#8221; results in:

[<img class="alignnone size-full wp-image-287" title="highlight" src="/wp-content/uploads/2011/12/highlight.jpg" alt="correctly highlighted quote" width="411" height="188" />][4]

Not bad.  By allowing Phrase-Through and Phrase-Around flexibility on the source XML schema we don&#8217;t have to transform the data to index it.  We get to preserver structure and have full text search at the same time!

&#8212; Dave

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

 [1]: /wp-content/uploads/2011/12/searchscreen.png
 [2]: http://www.front2backdev.com/2011/12/14/tutorial-mobile-shakespeare-part-2-the-reskin/ "Tutorial: Mobile Shakespeare (Part 2 – The Reskin)"
 [3]: /wp-content/uploads/2011/12/ripped.jpg
 [4]: /wp-content/uploads/2011/12/highlight.jpg