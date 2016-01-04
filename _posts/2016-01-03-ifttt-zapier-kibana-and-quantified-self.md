---
layout: post
title:  "Quantified Self: IFTTT, Zapier, and Elasticsearch"
date:   2016-01-03 18:00:00
categories: tech, iot, projects, quantified_self, elasticsearch, found
permalink: /2016/01/03/quantified-self-ifttt-zapier-elasticsearch/ 
published: true
---

![Architecture](/images/posts/2016-01-03-arch.jpg "Basic Architecture diagram linking IFTTT Zapier Heroku, and Found")

As a new years resolution I'm embarking on a Quantified Self project. 

It is known that I love tracking things.  As an example: 3 years of my beer drinking history from Untappd exported to CSV as a static dataset and loaded into Kibana:

![Untappd visualization](/images/posts/2016-01-03-beer.jpg "3 Years of Untappd data in Kibana")

Dashboards and visualizations are fun, but in order to truly embrace life logging (and to eat a bit more dogfood at work) I need these feeds to be *real time* rather than static data sets manually loaded when I happen to have the time.

##Inspiration

Last year for new years I built an infographic at [Piktochart](http://piktochart.com/) [1] to show my year's accomplishments.  This is pretty easy for me given that most of my life is recorded in a google calendar somewhere or inside a dashboard for one of the services I use like Tripit, Untappd, or Fitbit.  I started a new visualization and it became really obvious that

* Manually combining all these datasources was a pain in the ass
* I spent most of last year working and my life was way out of balance
* What's the point of setting a new years resolution if you can't track how well things are going in Real Time.

Searching around people's blogs and the various life logging apps, I stumbled upon a movement / fixation out there called "[Quantified Self](http://quantifiedself.com/)" [2] that is related to the recent tech trends of personal data, wearables, and internet of things.  I found some tools that I liked, but rather than spend a whole bunch of time wrangling static data sets at the end of the year I set a challenge for myself:

* Quantify as many things as possible in a centralized place 
* Have as much of the data as possible loaded in real time
* Host as many things as possible in the cloud (don't run a server in my apartment)

##Getting Started: Centralizaing Data

I wanted to get going quickly managing real time data and building dashboards.  No better tool combination than Elasticsearch and Kibana, and no easier place to host that than [Found](https://found.elastic.co) [3].

Setting up a new 1 node of the latest Elasticsearch with SSL, basic authentication, automated snapshots and the with the latest build of Kibana was point and click easy.  Best part is that I'll never have to worry about upgrade automation etc as Found handles all of that.

##Some Starting Data Sources

I actually had quite a large number of datasources already available to me, rather than start with custom API pollers I decided to build integration to tools like *If This Then That* ([IFTTT](http://ifttt.com/)) [4] and [Zapier](https://zapier.com) [5] which act as *codeless* service brokers on the public web.  These two tools combined had out of the box integration to useful data sources and trigger management.

My top two desires for data sources were my Fitbit step counter stats and the local weather.  Both occur every day and are simple to work with numeric metrics and text categories.

The problem with these 'consumer' oriented data hubs is that they are meant for human consumption and not clean data integrations. IFTTT added a "Maker" channel recently which allows it to make outbound REST calls, which is excellent. And Zapier has a pretty rich weather data source with a JSON output, but both offer very little in the way of transformation or normalization of things like dates, which is essential to centralized dashboarding.

To solve the problem I built a custom Heroku REST Service ([code here](https://github.com/derickson/metrics-rest-service)) [6] to manage and transform my data coming from IFTTT and Zapier into Elasticsearch.    I also wanted to keep a full log of all the raw data in case I changed my mind on index settings for Elasticsearch and wanted to quickly regenerate. data can be posted to /raw or /metric endpoints.  /raw will just capture data that isn't ready for centralized idnexing.  /metic will both record the raw feed as well as do normalization and centralized indexing.  /metricOnly will only do the normalization transform and central indexing.  By doing a scan and scroll off of the raw-* index target and resubmitting data to /metricOnly the data can be quickly reindexed.

IFTTT Fitbit recipe:

![IFTTT Fitbit](/images/posts/2016-01-03-ifttt.jpg "IFTTT recipe")

Zapier Weather recipe:

![Zapier Weather](/images/posts/2016-01-03-zapier.jpg "Zapier recipe")

##Tougher Questions

A new tool that I picked up is a custom QS app called [Reporter App](https://itunes.apple.com/us/app/reporter-app/id779697486?mt=8) [7] from Nicholas Feltron, who does some pretty impressive end of year [infographics](http://feltron.com/) [8].  It does JSON formatted randomly timed surveys using both phone entry and the various sensors on my phone.  It can do live updates to Dropbox, so it would be something useful for tracking # of coffee's, sodas, days spent working, etc for later analaysis.  Also, given that the data is well formed JSON I'll be able to rig a script to mine the dropbox folder incrementally, which will be a good task for later.

##Next Steps

I used some gift cards from the holidays to pick up an [Automatic](https://www.automatic.com/home/) [9] car IOT device.   I've already got IFTTT logging car trips I make to a Google spreadsheet, I think that will be a fun one to put into Elasticsearch next.

Links:

* [1] [Piktochart](http://piktochart.com/)
* [2] [Quantified Self](http://quantifiedself.com/)
* [3] [Found](https://found.elastic.co)
* [4] [IFTTT](http://ifttt.com/)
* [5] [Zapier](https://zapier.com)
* [6] [Heroku REST Serivice source code (Github)](https://github.com/derickson/metrics-rest-service)
* [7] [Reporter App](https://itunes.apple.com/us/app/reporter-app/id779697486?mt=8)
* [8] [Feltron Infographics](http://feltron.com/)
* [9] [Automatic](https://www.automatic.com/home/)