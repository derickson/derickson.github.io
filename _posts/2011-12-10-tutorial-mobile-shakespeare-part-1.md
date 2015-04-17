---
title: 'Tutorial: Mobile Shakespeare (Part 1)'
author: Dave
layout: post
permalink: /2011/12/10/tutorial-mobile-shakespeare-part-1/
categories:
  - Tutorial
tags:
  - JQuery Mobile
  - Mobile
  - mobshake
  - REST
  - rewrite
  - tutorial
  - XQuery
---
The first MarkLogic tutorial I ever followed was [Clark Richey&#8217;s Shakespeare hands-on][1], hosted from the MarkLogic developer website back in the MarkLogic Server 4.0 days.  I think it is only fitting / nostaligic that the first tutorial I write for Front2BackDev.com be a similar app but with a new twist.  Let&#8217;s make an application that helps a user access the complete works of Shakespeare, but let&#8217;s use some new tech, in MarkLogic and otherwise, that wasn&#8217;t available back in 2009 such as:

  * **The MarkLogic URL rewriter**:  I&#8217;m going and Nuno Job&#8217;s [rewrite][2] library to implement a REST friendly URL scheme.
  * **Information Studio**: loading the complete works of Shakespeare (in XML form) into a database should take us no more than a few minutes.  Because we don&#8217;t really have to do any upfront data modeling in MarkLogic, we often take a &#8220;Load First .. Damn the Schemas full speed ahead!&#8221; mentality that should be refreshing to even the most jaded Agile developer.
  * In later installments of this tutorial I am going to use the **MarkLogic Search API** to build some quick but professional search functionality &#8230; we&#8217;ll get to this later.
  * And lastly rather than building an XHTML website with XQuery we&#8217;re going to take a look at **[JQuery Mobile][3]**, which takes a &#8220;markup only&#8221; approach to developing mobile web apps that have the same look and feel of a native iOS app.

This is what we are going to build: *BROKEN LINK TO OLD DEMO*

Getting started:

  * Install MarkLogic 5 ( [Install Guide][4] ).  This guide assumes you&#8217;ll be working from a developer machine that is running your MarkLogic Server.
  * From Application Services create a new database called &#8220;shake&#8221;
  * Application Services is located at http://localhost:8000/appservices/
  * At the top of the screen there is a &#8220;+ New Database&#8221; button.  Use it to create a new database called &#8220;shake&#8221;

  * Download the complete works of Shakespeare from <http://www.ibiblio.org/xml/examples/shakespeare/> to a folder on your desktop called &#8220;shake&#8221;
  * Use Application Services&#8217; Information Studio to load the Shakespeare plays into the &#8220;shake&#8221; database
  * On the Application Services screen click the &#8220;+ New Flow&#8221;  button in the &#8220;Information Studio Flows&#8221; area
  * In the new flow, type the absolute file system path to the &#8220;shake&#8221; folder on your desktop in the &#8220;Collect&#8221; section with a &#8220;Filesystem Directory&#8221; collector.
  * Upload the files by selecting the &#8220;shake&#8221; database in the &#8220;Load&#8221; section and clicking &#8220;Start Loading&#8221;
  * 37 files &#8230; should take a few seconds.

  * Check out the files you uploaded in Query Console
  * Go to http://localhost:8002/qconsole
  * Change the content source to &#8220;shake&#8221; and hit the explore button
  * Verify the shakespeare plays are there
  * Run a sample query like (/PLAY)[1]/TITLE/text()

  * Open your favorite code editor (I use the Eclipse IDE with the XQDT plugin).
  * Create a new project (the general &#8220;Project&#8221; in Eclipse) called shake, and create a new folder within that project called &#8220;xquery&#8221;
  * In the new project create a file located at /xquery/index.xqy with the content

<pre> "Hello World!"</pre>

  * Back in the admin console, create a new HTTP server on port 9001.  Set it&#8217;s database to &#8220;shake&#8221;, modules to <filesystem>, and root to the absolute path of the project/xquery folder you made.  For me this is /Users/dave/Documents/workspace/shake/xquery
  * Open up http://localhost:9001 in a browser and confim the hello world is working

<div>
  Okay now for some quick REST URL design:
</div>

<div>
  <ul>
    <li>
      / will be our home page.  We&#8217;ll list all the plays here.
    </li>
    <li>
      /play/1 should bring up the Shakespeare play with id &#8220;1&#8221;
    </li>
    <li>
      /play/1/characters should display all the characters in the play with id &#8220;1&#8221;
    </li>
    <li>
      /play/1/act/1/scene/1 should bring up act 1 scene 1 of play &#8220;1&#8221;
    </li>
  </ul>
  
  <div>
    But wait, the XML for the Shakespeare plays doesn&#8217;t have an &#8220;id&#8221;.  That&#8217;s cool, add one quickly with Query Console.  There isn&#8217;t a lot of data, so we can do it in a single transaction:
  </div>
  
  <div>
    
      <pre><code class="language-xquery xquery">for $p at $id in /PLAY
let $at := attribute id {$id}
return
xdmp:node-insert-child($p,$at)</code></pre>
    
  </div>
  
  <div>
    We&#8217;ll now be editing the shake project.  So as not to confuse anyone, here is the file layout of my shake project when this tutorial is finished:
  </div>
  
  <div>
    <a href="/wp-content/uploads/2011/12/shakedirs.jpg"><img class="aligncenter size-full wp-image-171" title="shakedirs" src="/wp-content/uploads/2011/12/shakedirs.jpg" alt="Directory layout for shake project" width="189" height="222" /></a>
  </div>
  
  <div>
    Okay, now we are ready to program the REST handler.   Download the <a href="https://github.com/dscape/rewrite">rewrite library</a> from github.  In the Admin Console (port 8001) set the &#8220;url rewriter&#8221; setting of your HTTP server to &#8220;/rewrite.xqy&#8221; .  Now place the rewrite files in the following organization inside the project
  </div>
</div>

<pre>/xquery/
/xquery/lib/
/xquery/lib/rewrite/
/xquery/lib/rewrite/helper.xqy
/xquery/lib/rewrite/routes.xqy</pre>

To implement the REST rules, I prefer using Nuno Job&#8217;s rewrite library, which has a Ruby on Rails-like routes specification for redirecting pattern based URLs to &#8220;controller&#8221; resource modules. Create the file /xquery/config.xqy with the following content:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml" ;

(:  config.xqy
    This library module holds configuration
    variables for the application
:)

module  namespace cfg = "http://framework/lib/config";

(:  The rewrite library route configuration
    For documentation see: https://github.com/dscape/rewrite
:)
declare variable $ROUTES :=
    &lt;routes&gt;
        &lt;root&gt;home#get&lt;/root&gt;
        &lt;get path="play/:id"&gt;
          &lt;to&gt;play#get&lt;/to&gt;&lt;/get&gt;
        &lt;get path="play/:id/characters"&gt;
          &lt;to&gt;play#characters&lt;/to&gt;&lt;/get&gt;
        &lt;get path="play/:id/act/:act"&gt;
          &lt;to&gt;play#get&lt;/to&gt;&lt;/get&gt;
        &lt;get path="play/:id/act/:act/scene/:scene"&gt;
          &lt;to&gt;play#get&lt;/to&gt;&lt;/get&gt;
    &lt;/routes&gt;;

declare variable $TITLE := "Mobile Shakespeare";</code></pre>


Next we need to create the /xquery/rewrite.xqy file which will select resource &#8220;controllers&#8221; based on the ROUTES configuration.  This is the script that is called by MarkLogic for every incoming HTTP request to determine which main module should be called.  We configured it when we set the &#8220;url rewriter&#8221; setting in the Admin Console:


  <pre><code class="language-xquery xquery">xquery version "1.0-ml" ;

import module namespace shake-r =
    "routes.xqy" at "/lib/rewrite/routes.xqy";
import module namespace shake =
    "http://framework/lib/config" at "/lib/config.xqy";

    (: let rewrite library determine destination URL,
       use routes configuration in config lib :)
    let $selected-url    :=
            shake-r:selectedRoute( $shake:ROUTES )
    return
            $selected-url</code></pre>


Now to get fancy with some JQuery Mobile.  First in a file located at /xquery/view/v-mob.xqy inside the project create a new MVC &#8220;view&#8221; template using the [JQuery Mobile][3] framework.


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

module namespace v-mob = "http://framework/view/v-mob";

(:  Stick this at the top of any module that generates HTML so that
    empty tags don't get truncated to non-empty tags :)
declare option xdmp:output "method=html";

(:
    HTML5 Template using JQuery Mobile
    @author Dave http://www.front2backdev.com
    XQuery adaptation Public Domain.
        jQuery and jQuery Mobile: MIT/GPL license
:)

(:
    JQuery Mobile Output Template
    $title -- The html head title of the page
    $html  -- HTML5 nodes to put in the body
:)
declare function v-mob:render(
    $title as xs:string,
    $html as node()*
) {

xdmp:set-response-content-type("text/html"),
'&lt;!DOCTYPE html&gt;',
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;{$title}&lt;/title&gt;
        &lt;meta name="viewport" content="width=device-width, initial-scale=1" /&gt;
        &lt;link rel="stylesheet" href="http://code.jquery.com/mobile/1.0/jquery.mobile-1.0.min.css" /&gt;
        &lt;script type="text/javascript" src="http://code.jquery.com/jquery-1.6.4.min.js"&gt;&lt;/script&gt;
        &lt;script type="text/javascript" src="http://code.jquery.com/mobile/1.0/jquery.mobile-1.0.min.js"&gt;&lt;/script&gt;
    &lt;/head&gt;
    &lt;body&gt; 

        &lt;div data-role="page" data-theme="b"&gt;

            &lt;div data-role="header"&gt;
                &lt;a data-rel="back"
                   data-icon="back"
                   data-iconpos="notext"
                   data-transition="slide"
                   data-direction="reverse"&gt;Back&lt;/a&gt;
                &lt;h1&gt;{$title}&lt;/h1&gt;
                &lt;a href="/"
                   data-icon="home"
                   data-iconpos="notext"
                   data-transition="fade"&gt;Home&lt;/a&gt;
            &lt;/div&gt;&lt;!-- /header --&gt;

            &lt;div data-role="content"&gt;
                {$html}
            &lt;/div&gt;&lt;!-- /content --&gt;

        &lt;/div&gt;&lt;!-- /page --&gt;

    &lt;/body&gt;
&lt;/html&gt;
};</code></pre>


Next create a MVC &#8220;controller&#8221; for the home resource  in a file located at /resource/home.xqy with the content :


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

(: Home.  List all Shakespeare plays :)

import module namespace cfg = "http://framework/lib/config" at "/lib/config.xqy";

import module namespace h = "helper.xqy" at "/lib/rewrite/helper.xqy";
import module namespace v-mob = "http://framework/view/v-mob" at "/view/v-mob.xqy";

(: Don't forget to include this so script
   tags in the VIEW are not collapsed :)
declare option xdmp:output "method=html";

declare function local:get()  {
    v-mob:render(
        $cfg:TITLE,
        &lt;ul data-role="listview" data-inset="true" data-filter="true"&gt;
        {
            (: Note this code is inefficient at scale because we
               are effectively retrieving the entire database
               to generate these links.  It's only about 8MB of data
               and it is coming out of cache, but maybe we can speed
               it up later
            :)
            for $play in /PLAY
            return
                &lt;li&gt;
                  &lt;a href="/play/{fn:string($play/@id)}"&gt;
                    {$play/TITLE/text()}
                  &lt;/a&gt;
                &lt;/li&gt;
        }
        &lt;/ul&gt;
    )
};

try          { xdmp:apply( h:function() ) }
catch ( $e ) {  h:error( $e ) }</code></pre>


So once you are good with that resource, here is another more complex one to digest.  Lastly create a MVC &#8220;controller&#8221; for the play resource  in a file located at /resource/play.xqy with the following content (Sorry this one is kind-of long, but it is doing most of the work.  We can refactor the code into some Model and Util Library code later) :


  <pre><code class="language-xquery xquery">xquery version "1.0-ml";

(: Play detail  :)

import module namespace cfg = "http://framework/lib/config" at "/lib/config.xqy";

import module namespace h =
    "helper.xqy" at "/lib/rewrite/helper.xqy";
import module namespace v-mob =
    "http://framework/view/v-mob" at "/view/v-mob.xqy";

(: Don't forget to include this so script
   tags in the VIEW are not collapsed :)
declare option xdmp:output "method=html";

declare function local:characters()  { 

    let $id := h:id()
    let $play := /PLAY[@id eq $id]
    let $title := fn:string-join((
            $play/TITLE/text(),
            "Characters"
        )," ")
    return
    v-mob:render(
        $cfg:TITLE,
            &lt;p&gt;
                &lt;h2&gt;{$title}&lt;/h2&gt;
                &lt;h3&gt;DRAMATIS PERSONAE&lt;/h3&gt;
                {
                for $node in $play/PERSONAE/node()
                return
                    typeswitch ($node)
                    case element(TITLE) return
                        ()
                    case element(PGROUP) return
                        &lt;ul data-role="listview"  data-inset="true"&gt;
                        {
                            for $p in $node/PERSONA
                            let $name := $p/text()
                            return
                                &lt;li data-theme="d"&gt;{$name}&lt;/li&gt;,
                            for $d in $node/GRPDESCR
                            return
                                &lt;li data-theme="c"&gt;{$d/text()}&lt;/li&gt;
                        }
                        &lt;/ul&gt;
                    case element(PERSONA) return
                    &lt;ul data-role="listview"  data-inset="true"&gt;
                        {
                            let $p := $node
                            let $name := $p/text()
                            return
                                &lt;li data-theme="d"&gt;{$name}&lt;/li&gt;
                        }
                        &lt;/ul&gt;
                    default return
                        ()
                }
            &lt;/p&gt;
    )
};

declare function local:get()  {
    let $id := h:id()
    let $act := xdmp:get-request-field("act",())[1]
    let $scene := xdmp:get-request-field("scene",())[1]

    let $act := if($act castable as xs:int) then
                  xs:int($act)
                else ()
    let $scene := if($scene castable as xs:int) then
                    xs:int($scene)
                  else ()

    let $play := /PLAY[@id eq $id]

    return
    v-mob:render(
        $cfg:TITLE,
        (
            &lt;p&gt;
                &lt;h2&gt;{$play/TITLE/text()}&lt;/h2&gt;
                {
                    if($act) then
                      &lt;h3&gt;{($play/ACT)[$act]/TITLE/text()}&lt;/h3&gt;
                    else (),

                    if($act and $scene) then
                      &lt;h3&gt;
                        {(($play/ACT)[$act]/SCENE)[$scene]/TITLE/text()}
                      &lt;/h3&gt;
                    else (),

                    if(fn:not( $act or $scene )) then
                        &lt;ul data-role="listview" data-inset="true" &gt;
                          &lt;li&gt;
                            &lt;a href="/play/{$id}/characters"&gt;
                              Characters
                              &lt;span class="ui-li-count"&gt;
                                {fn:count($play/PERSONAE/PERSONA)}
                              &lt;/span&gt;
                            &lt;/a&gt;
                          &lt;/li&gt;
                        &lt;/ul&gt;
                    else
                      ()
                }
            &lt;/p&gt;,

            if($act and $scene) then (
                let $paging :=
                    &lt;div data-role="controlgroup" data-type="horizontal"&gt;
                    {
                        if($scene eq 1 and $act eq 1) then
                            ()
                        else if( $scene eq 1 and fn:exists( ($play/ACT)[$act -1] )) then
                            &lt;a data-icon="arrow-l" href="/play/{$id}/act/{$act -1}/scene/{ fn:count(($play/ACT)[$act -1]/SCENE) }" data-role="button"&gt;Previous Scene&lt;/a&gt;
                        else
                            &lt;a data-icon="arrow-l"   href="/play/{$id}/act/{$act}/scene/{$scene -1}" data-role="button"&gt;Previous Scene&lt;/a&gt;,

                        &lt;a  data-icon="grid" href="/play/{$id}" data-role="button"&gt;Back to Scene Selection&lt;/a&gt;,    

                        if( fn:count(($play/ACT)[$act]/SCENE) gt $scene ) then
                            &lt;a  data-icon="arrow-r"  href="/play/{$id}/act/{$act}/scene/{$scene +1}" data-role="button"&gt;Next Scene&lt;/a&gt;
                        else if( fn:exists( ($play/ACT)[$act +1] )) then
                            &lt;a  data-icon="arrow-r"  href="/play/{$id}/act/{$act +1}/scene/1" data-role="button"&gt;Next Scene&lt;/a&gt;
                        else
                            ()
                    }
                    &lt;/div&gt;
                return (

                    $paging,

                    for $node in (($play/ACT)[$act]/SCENE)[$scene]/node()
                    return
                        typeswitch ($node)
                        case element(TITLE) return
                            &lt;p&gt;&lt;strong&gt;{$node/text()}&lt;/strong&gt;&lt;/p&gt;
                        case element(STAGEDIR) return
                            &lt;p&gt;&lt;em&gt;{$node/text()}&lt;/em&gt;&lt;/p&gt;
                        case element(SPEECH) return
                            &lt;div class="ui-body ui-body-b"
                                 style="margin:20px 0px;"&gt;
                            {
                                $node/SPEAKER/text(),
                                &lt;div style="margin-left:20px;"&gt;{
                                    for $line in $node/LINE/text()
                                    return
                                        &lt;div&gt;{$line}&lt;/div&gt;
                                }
                                &lt;/div&gt;
                            }

                            &lt;/div&gt;
                        default return
                            ()

                    ,

                    $paging
                )

            )
            else
                &lt;ul data-role="listview"  data-inset="true" &gt;
                {
                    for $act at $a in $play/ACT
                    return
                    (
                        &lt;li data-role="list-divider"&gt;{$act/TITLE/text()}&lt;/li&gt;,

                        for $scene at $s in $act/SCENE
                        return
                            &lt;li data-theme="c"&gt;
                                &lt;a href="/play/{$id}/act/{$a}/scene/{$s}"&gt;
                                    {$scene/TITLE/text()}
                                &lt;/a&gt;
                            &lt;/li&gt;
                    )
                }
                &lt;/ul&gt;
        )
    )
};

try          { xdmp:apply( h:function() ) }
catch ( $e ) {  h:error( $e ) }</code></pre>


And now for our hard earned screenshots.  In the next part of the tutorial we&#8217;ll add a real search page to search within the text of the plays.

A live demo of the app so far can be found at: *LINK TO OLD DEMO BROKEN*

Part 2 of this tutorial can be found [here][5].

[<img class="aligncenter size-large wp-image-128" title="shake" src="/wp-content/uploads/2011/12/shake-1024x992.jpg" alt="" width="510" height="494" />][6]Updates:

12/16/2011 &#8211; links to apps changed  
12/18/2011 &#8211; adding reminder to set rewriter in HTTP app server configuration

 [1]: http://developer.marklogic.com/learn/2009-01-get-started-apps
 [2]: https://github.com/dscape/rewrite
 [3]: http://jquerymobile.com/
 [4]: http://developer.marklogic.com/pubs/5.0/books/install.pdf
 [5]: http://www.front2backdev.com/2011/12/14/tutorial-mobile-shakespeare-part-2-the-reskin/ "Tutorial: Mobile Shakespeare (Part 2 – The Reskin)"
 [6]: /wp-content/uploads/2011/12/shake.jpg