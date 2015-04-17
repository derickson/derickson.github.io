---
title: 'Obstacle Course Tutorial 1: Loading public API data into ML'
author: Dave
layout: post
permalink: /2012/05/11/obstacle-course-tutorial-1-loading-public-api-data-into-ml/
categories:
  - Tutorial
tags:
  - loading
  - MarkLogic
  - scheduled task
---
<div id="attachment_436" class="wp-caption alignnone" style="width: 500px">
  <a href="/wp-content/uploads/2012/05/obstacle.jpg"><img class="size-full wp-image-436" title="obstacle" src="/wp-content/uploads/2012/05/obstacle.jpg" alt="" width="500" height="285" /></a>
  
  <p class="wp-caption-text">
    U.S. Air Force photo/Benjamin Faske
  </p>
</div>

I’ve written a tutorial or two for this site, but I’m going to take a shot at a different way of putting out training materials.  Call it, “training by obstacle course.”  I believe there is an audience for training that appreciates learning by challenge rather than copy-paste.  Personally I learn by doing, so in order to internalize a technology I prefer to figure things out myself using the existing API documentation and a push in the right direction.  Let me know what you think!

### Loading public data into ML challenge:

Objective:

  * Periodically populate a MarkLogic database with interesting full text content from NPR
  * Get used to looking up XQuery and MarkLogic functions in the API documentation

Prerequisitess:

  * [Introductory XQuery knowledge][1]
  * A locally installed copy of MarkLogic (like the free Express edition!)

Go to the [National Public Radio API][2] site and register a user account.  You can now get an API key to use their story API services.  These will get you an XML representation of stories in specific topics as well as full text transcriptions.  Play around with their query builders.  I decided to work on news stories from Afghanistan

  * http://api.npr.org/query?id=1149&apiKey=PUT\_YOUR\_KEY_HERE

Using your MarkLogic admin console (port 8001), create a new database called NPR.  You&#8217;ll have to create and attach a forest to this database before you&#8217;ll be able to use it.

Using your query console (http://HOSTNAME:8000/qconsole/) pull data from NPR&#8217;s API into your NPR database

  * Don&#8217;t forget to set the Query Console content source to the NPR database
  * Check out the xmp:http-get() method from the [MarkLogic XQuery API docs][3].  ([Alternate searchable version][4]) This will pull data from NPR&#8217;s HTTP based API.
  * Note: xdmp:http-get returns two items.  The second item returned is the retrieved XML.
  * Insert the data into the NPR database using the xdmp:document-insert() function

If you are using the NPR &#8220;query&#8221; API like I did, the returned XML has <story> tags which have inside them <transcript> tags that include links to secondary API calls for retrieving transcripts of the audio content.  You have the option of using the story tags as they are, deciding to follow the links to the transcripts, or both!

  * Modify your code to pull the same query, but now loop through the stories and possibly the embedded transcripts ( you could use a FLWOR statement )
  * For each story optionally retrieve the referenced transcript from the web and insert it into MarkLogic.  The URI you insert the file into should probably involved the unique story Id from within the retrieved data.
  * Check to see if the document exists already using fn:doc-available().  If the document is already there, just skip the write action ( you could use an IF / ELSE statement )

Congrats!  You have now loaded data into MarkLogic as-is from a public API.  As a next step, deploying this code as a module

  * Make the above code an .xqy script and place it on the your file system.
  * Create an HTTP server on an open port which gets its data from the NPR database and the module from the file system with the &#8220;root&#8221; option set to the absolute path of  your folder containing the module.
  * Hitting this &#8220;deployed&#8221; script with a web browser at http://HOSTNAME:PORT/SCRIPTNAME.xqy will now execute the data loading module.

Alternatively you could now configure a MarkLogic scheduled task to load live data from the NPR data feed on a regular cycle.  The scheduled task controls are in the admin console under **Configure > Groups > group_name > Scheduled Task** inside the &#8220;Create&#8221;.  Tip: you&#8217;ll probably need to run the scheduled task as your admin user for now.  In a production situation you might consider creating a separate user with only the permissions needed to write to your database.

Congratulations, you are now pulling in semi-real time data to MarkLogic.  (The only more real time would be if you could get a content provider to push you data as it happens rather than polling for it on a schedule)  Any search app we build on this data will now be instantly more interesting as it will have new data each time you visit it!

 [1]: http://www.front2backdev.com/2012/05/03/introducing-developers-to-xquery-in-marklogic/
 [2]: http://www.npr.org/api/index
 [3]: http://community.marklogic.com/pubs/5.0/apidocs/All.html
 [4]: http://api.xqueryhacker.com/