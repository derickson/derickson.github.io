---
title: Kettles, Spoons, Kitchens, and Jobs
author: Dave
layout: post
permalink: /2012/05/18/kettles-spoons-kitchens-and-jobs/
categories:
  - Experiments
tags:
  - ETL
  - Kettle
  - MarkLogic
  - Pentaho
  - PostgreSQL
  - RDBMS
  - Spoon
  - XML
---
*Using Kettle – Spoon ETL tool to move data from an RDBMS to MarkLogic*

Don’t get me wrong.  My ETL-fu is strong.  Give me some SQL, XQuery, Java, Perl, and Python and I am a dangerous man when it comes to rapidly taking data out of your beloved RDBMS silo and putting it into a NoSQL database.  However scaling that ETL process to a large IT program that handles dozens of databases with thousands of tables can be a schedule and maintenance cost nightmare.  In this blog post I’m exploring an approach that I haven’t really tackled in a few years … using a visual ETL tool to pull data from a relational database, transform it into XML, and then upload it into a NoSQL database. (in this case MarkLogic)

I’ve had several bad experiences with open source ETL so I’ve come to demand two things:

  1. I need to be able to run the ETL jobs that I create in the tool outside of the IDE without upgrading to an enterprise edition
  2. The software needs to pass the open source “sniff test.”  By that I mean that I need to be able to download it from the internet, install it, and get a HelloWorld! working within 30 minutes.

I was able to get a basic example working with [Kettle Spoon][1] 4.2.0 (related to Pentaho Data Integration), but I had to give up on two or three other tools, before feeling like I had enough to feel justified writing a blog post.

### Setup

I have MarkLogic 5.0 and PostgreSQL 9.1 running on one laptop and Spoon running on a second Windows laptop (it wouldn’t open on my Mac, though most people’s Youtube tutorial videos seem to be on Macs).  My PostgreSQL server has a database named “dave” with a table named “call” to represent call log information:

[<img class="alignnone size-full wp-image-454" title="postgre" src="/wp-content/uploads/2012/05/postgre.jpg" alt="PostgreSQL source data" width="734" height="582" />][2]

### ETL Design

I’m going to pull data from a single PostgreSQL table using a JDBC connection from my ETL tool, map it’s columns to elements of a schema-less XML file, and then POST the file to a waiting HTTP accessible service (POX more than REST) over on my MarkLogic.  Ideally the MarkLogic endpoint would insert the file into a database, but for demo purposes I am just logging the received XML:

<pre>xquery version "1.0-ml";
xdmp:log(xdmp:quote(xdmp:get-request-body("xml")))</pre>

### Spoon Transform

After creating a pretty generic JDBC connection to my PostgreSQL database, the first step was to create a Spoon Tranform

[<img class="alignnone size-full wp-image-455" title="pg2ml" src="/wp-content/uploads/2012/05/pg2ml.jpg" alt="transform" width="803" height="402" />][3]

The first widget here is an Input > Table input running a SQL call of

<pre>SELECT * from call</pre>

against my database connection.  The second widget is a Transform > Add XML tool which takes the row data from the table and, with some clicking, auto maps it to a flat XML structure.  The phone number integer from the call table was outputting with commas until I switched it to have a formatting pattern of #.

[<img class="alignnone size-full wp-image-456" title="xmlcolumn" src="/wp-content/uploads/2012/05/xmlcolumn.jpg" alt="Add XML Column widget" width="612" height="646" />][4]

The last step is a Lookup > Rest Client.  This appears to be a relatively new feature added to support all the NoSQL databases being used out there that use REST as their primary connection mechanism.  I just have to target a HTTP exposed XQuery module with a POST call and make sure I pass the output variable the Add XML step modified.

A test run results in the following on my MarkLogic ErrorLog.txt file (the expected result of the xdmp:log call)

<pre>Info: 6099-pentaho: &lt;?xml version="1.0" encoding="UTF-8"?&gt;
Info: 6099-pentaho: &lt;call xml:lang="en"&gt;&lt;id&gt;1&lt;/id&gt;&lt;phone&gt;3235551234&lt;/phone&gt;&lt;comments&gt;Called Home&lt;/comments&gt;&lt;name&gt;Dave's Family&lt;/name&gt;&lt;/call&gt;
Info: 6099-pentaho: &lt;?xml version="1.0" encoding="UTF-8"?&gt;
Info: 6099-pentaho: &lt;call xml:lang="en"&gt;&lt;id&gt;2&lt;/id&gt;&lt;phone&gt;3235551234&lt;/phone&gt;&lt;comments&gt;Called Home Again&lt;/comments&gt;&lt;name&gt;Dave's Family&lt;/name&gt;&lt;/call&gt;
Info: 6099-pentaho: &lt;?xml version="1.0" encoding="UTF-8"?&gt;
 …</pre>

As a final step I export the Transform to the filesyetm as a .ktr file using the Export file menu item. This will help when trying to run the ETL process from the command line.

### Configuring a job for the transform

Kettle has a concept of a Job, which allows control from a flow composed of Transforms.  The following is a simple Job whose only purpose is exposing the Transform we made above.  The transform step is configured run the .ktr file we made above.

I then export this job to a .kjb using the same Export feature

### Running from the Command Line

And now the most important part!  Being able to run the ETL from outside of the client GUI.   This part is absolutely essential because it allows me to use the ETL as a scheduled job or in a continous integration cycle.

The following is the command I run from the Windows command line to run my job:

<pre>kitchen.bat /file:.\mytransform\job.kjb /level:Minimal</pre>

If I were to get my Mac version of Kettle working, this is what I would run:

<pre>./kitchen.sh -file=./mytransform/job.kjb -level=Minimal</pre>

### Next Experiments

Now I have to figure out if the tool can be used to export hierarchical XML obtained from a join across tables in the relational database.  I’ll leave that for another time.

 [1]: http://kettle.pentaho.com/
 [2]: /wp-content/uploads/2012/05/postgre.jpg
 [3]: /wp-content/uploads/2012/05/pg2ml.jpg
 [4]: /wp-content/uploads/2012/05/xmlcolumn.jpg