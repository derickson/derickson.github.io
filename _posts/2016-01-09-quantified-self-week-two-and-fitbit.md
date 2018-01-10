---
layout: post
title:  "Quantified Self: Week two progress and 'Near Real-Time' Fitibit activity"
date:   2016-01-09 18:00:00
tags: projects
permalink: /2016/01/09/quantified-self-week-two-and-fitbit/
published: true
---

![Dashboard](/images/posts/2016-01-09-dash.jpg "Qantified Self Dashboard")

I'm about a week into quantified self tracking.  At this point my IFTTT and Zapier feeds are stable and I have good trust that the new data will have arrived in my Kibana dashboard each morning.  None of my behaviors and habits have changed of course.  Activity level-wise, they've probably gotten worse with all the hacking I've been doing to set up the heroku metrics service.  At least I'll have the data to show improvement from a hacking baseline.  ;)

## Pulling Fitbit data in real time

I spent time this week pulling data from from Fitbit's APIs and had to get deeper on Oauth2 integration.  Once my production site has a token I can use transfer it to my local dev setup and use it for about an hour, which was a good discovery.  I'd like to thank [@Peebles](https://github.com/peebles) for the excellent example of simple Oauth2 registration for personal fitbit data with auto-token refresh built in: [fitbit-oauth2 code](https://github.com/peebles/fitbit-oauth2) [1].  The result of the work is a [fitbit module](https://github.com/derickson/metrics-rest-service/blob/86ba54a271b93a9869be90ec5bc1538cfc5f2c67/fitbitApp.js) [2] for my metrics-rest-service for polling fitbit services  backed by a [modification of Peeples' project](https://github.com/derickson/metrics-rest-service/blob/86ba54a271b93a9869be90ec5bc1538cfc5f2c67/fitbit-oauth2/Fitbit.js) [3] with a bug fixed.

Fitbit only offers fine grained activity data for personal projects and close commerical partners.  As someone who works in the opens source space but is more of a Maker in my spare time want to applaud Fitbit for making their APIs so Maker approachable.  It's easy to have everything locked down and unapproachable except to the koolaid drinkers or deeply integrating partners.  Having an open platform where data can both go in *AND* out is such a big deal.

>"Fitbit is very supportive of non-profit research and personal projects."
>
>-- Fitbit documentation on personal data feeds: [link](https://dev.fitbit.com/docs/activity/#get-activity-intraday-time-series) [4] 

It's quotes like that that make people want to work with your stuff. So thanks Fitbit.

![Fitbit data](/images/posts/2016-01-09-steps.jpg "Intra-day data from Fitbit")

I found through experimentation that Fitibit has about a 10 minute latency for making data that has been posted by your devices/phone to their services available to the fine grained APIs.  It will falsely report 0 steps for recent mintues.  This is a problem as I want to do an append only write workload to Elasticsearch if possible.  Going back over old data isn't a huge problem, but I'd like to avoid it and as a result I have my code set up to not ask for data less than 20 minutes old and only check for data every 10 minutes or so.  The algorithm for doing this and backfilling data to the beginning of the year took a bit of event-based / asynchronous thinking.

Every time I start hacking with NodeJS I have to get back into asynchronous thinking.  The biggest productivity pauses were all related to double-callback mistakes (some in borrowed code) and flooding connection pools by things not being serialized enough.  This is similar to me having to teach myself python again every year.  I need an async.map equivalent with a pooled job maximum so I stop flooding my Elasticsearch instance with concurrent connections.  I've probably found one before and just forgotten about it.  I'll have to go back through my old code.


Links:

* [1] [@Peeble's fitbit-oauth2 code example](https://github.com/peebles/fitbit-oauth2)
* [2] and [3] [Snapshot of metrics serivce with fitbit ouath2 added](https://github.com/derickson/metrics-rest-service/tree/86ba54a271b93a9869be90ec5bc1538cfc5f2c67)
* [4] [fitbit manual for intraday data](https://dev.fitbit.com/docs/activity/#get-activity-intraday-time-series)