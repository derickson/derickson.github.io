---
layout: post
title: "2017 Quantified Year in Review"
date: 2018-01-01 12:00:00
categories: quantified self, visualization
permalink: /2018/01/01/year-in-review-17/
img: posts/2018-01-01-qs12.jpg
published: true
---

Another year another Quantified Self blog post

## Weather

My magic mirror logged the temperature to Elasticsearch every 10 minutes this year.  This is my quick visualization of actual vs predicted daily high and low temperatures.  The data source is wunderground.

![Weather](/images/posts/2018-01-01-qs01.jpg)

You can even see the affect of the partial solar eclipse that happened in North America on August 21st, 2017

![Weather Eclipse](/images/posts/2018-01-01-qs02.jpg)￼

## Weight

I ended the year heavier than I started it, having spent the summer slightly lighter.  I use an Fitbit Aria scale connected to my Elasticsearch cluster with IFTTT

![Weight](/images/posts/2018-01-01-qs03.jpg)

## Travel

Adding up the hours I spent 6 days in the air this year.  I track my travel in Tripit and used jetitup.com's tools for visualization and summarization

![Travel Map](/images/posts/2018-01-01-qs04.jpg)

![Travel Stats](/images/posts/2018-01-01-qs05.jpg)
￼

## Beer

I taste a lot of beer and log all the beer I drink with the Untappd mobile app.  I use a custom app running in Heroku to poll my Untappd API for new checkins and log the data to Elasticsearch.  While many of these checkins are half beers tasted with my wife of part of flights while visiting a brewery, it still represents a solid pattern of calorie consumption I’m going to need to change if I want to drop some weight.

![Beer daily average](/images/posts/2018-01-01-qs06.jpg)

![Beer per week](/images/posts/2018-01-01-qs07.jpg)

![Beer around the world](/images/posts/2018-01-01-qs08.jpg)

![Beer top breweries](/images/posts/2018-01-01-qs09.jpg)

## Driving 

I have an Automatic IOT sensor on my car, and log all car rides using an IFTTT trigger to a google spreadsheet which I then parse into Elasticsearch.  The visualizations out the standard dashboards are so pretty though, so I’ll just put up my favorites here.

![Driving Stats](/images/posts/2018-01-01-qs10.jpg)

![Driving Large Map](/images/posts/2018-01-01-qs11.jpg)

![Driving Area Map](/images/posts/2018-01-01-qs12.jpg)
￼
## Movies

While I didn’t track these in real time, Fandango receipts and my viewing history from Netflix and Amazon video were enough to back-populate my letterboxd account.  I paid for the pro account to get their beautiful visualizations.  I’ve watched a lot of TV shows rather than movies in 2017.  Supposedly we live in the golden age of TV, so I’m still considering whether or not I want to track the TV I watch.  Hours of screen time with TV and which shows I liked might be better.

[https://letterboxd.com/azathought/year/2017/](https://letterboxd.com/azathought/year/2017/)

![Movie Stats](/images/posts/2018-01-01-qs13.jpg)

![Movie Tops](/images/posts/2018-01-01-qs14.jpg)

## Books

I read lots of comic books this year using the Marvel Unlimited service.  I really only read 6 real Books in 2017, so I’ve set a goal for next year of 12 and will use GoodReads to track my progress

* Artemis by Andy Weir
* Who Thought This Was a Good Idea?: And Other Questions You Should Have Answers to When You Work in the White House by Alyssa Mastromonaco
* French Revolutions For Beginners by Michael J LaMonica
* What If?: Serious Scientific Answers to Absurd Hypothetical Questions by Randall Munroe
* Leviathan Wakes by James S.A. Corey
* Armada by Ernest Cline

