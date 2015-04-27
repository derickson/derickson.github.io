---
layout: post
title:  "Converting To Jekyll"
date:   2015-04-26 12:00:00
categories: work, tech, jekyll
permalink: /2015/04/26/converting-to-jekyll/ 
published: true
---

This blog started life as a Wordpress site backed by a LAMP server over at Hostmonster.  While the service was awesome, I decided to get more onboard with what the cool kids were doing and convert to a "compiled" static site rather than one backed by a database and CMS tool that would need constant patching, etc.  While that database was certainly useful five years ago for:

* collaboration between multiple authors
* having snapshots of the site in case I messed something up
* caching frequently used content in RAM rather than having go to disk every time

honestly, having to triage spam commenters, update the Wordpress software, and the fear of losing access to the content was pretty annoying. The annoyance was compounded by the fact that flat fext files + git + someone else automating a distributed webcache would do just about the same thing if not better as every commit would be recoverable and all my content would be in markdown in a simple folder structure, which for my needs is fine for archival purposes.

### Step zero: Picking a Tool

I looked around for a bit, but settled on [Github Pages](https://pages.github.com/) + [Jekyll](http://jekyllrb.com/).  I wanted good version control, which alternatives that worked off of Dropbox didn't really provide.  While I'm not familiar with the CSS compiler used by Jekyll, it wasn't too hard to control after playing around for a bit. 

### Step one: Export Wordpress Content

By installing a [Jekyll Exporter](https://wordpress.org/plugins/jekyll-exporter/) as a plugin to my wordpress site I was able to convert the entire site with photos, code samples, etc over into a fully contained Jekyll site, which downloaded to my desktop as a zip file.

### Step two: Cleaning up the HTML and escaped Javascript

So my old Wordpress site used [gists](https://gist.github.com/) to neatly embed code samples.  While Jekyll did support embedding gists, the exporter had basically just rendered the final product out to Markdown. My code samples included enough The code samples in my blog, especially the HTML that contained code samples of [HTML5 Boilerplate](https://html5boilerplate.com/) with javascript escaped within two levels of HTML given that the Jekyll export had added its own outer envelope that I needed to go and clean up some things.  Luckily it was all fairly straightforward to fix using TextMate's Find & Replace across an entire project.

### Step three: Branding

Some [quick CSS work](https://github.com/derickson/derickson.github.io/commit/6141a22308e7b6e488796bd9c81b03d594513ed6) carried over the graph paper background from the old site.

I uploaded to my username.github.io project on github, [added google analytics](https://github.com/derickson/derickson.github.io/commit/b38d95f96ea0c2e512da048721c69d9d61bcba3a) to the footer, gave [Github a CNAME](https://help.github.com/articles/adding-a-cname-file-to-your-repository/) project entry, and switched my DNS settings to point my domain to the github project hosting the page.

### Step four: Pagination and Archive

I found jekyll's explanation of adding pagination to the site frustrating.  The key code to get it working is currently in my [index.html](https://github.com/derickson/derickson.github.io/blob/master/index.html) and [_includes/header.hmtl](https://github.com/derickson/derickson.github.io/blob/master/_includes/header.html) page where I had to tag items I wanted in my navigation header so the rendered pagination pages wouldn't show up there.  Note the ```page.useInNav``` check.  To get a 50 character preview, I shamelessly stole code from my [co-worker Peter's Jekyll blog](http://peter.mistermoo.com/).  I also added an article archive to make it easier for me to find older posts.  Code thanks to [Reyhan](http://reyhan.org/2013/03/jekyll-archive-without-plugins.html).

And that's it.  I might mess around a bit more to get a tag cloud or a preferred way to embed gists, but I'm fairly satisfied.  To post, all I have to do is add a page to my _posts folder, preview it with a quick ```jekyll serve```, and then upload with a git add + commit + push.  Certainly faster than dealing with the underpowered LAMP hosting. 


