---
title: 'Tutorial: Mobile Shakespeare (Part 2 &#8211; The Reskin)'
author: Dave
layout: post
permalink: /2011/12/14/tutorial-mobile-shakespeare-part-2-the-reskin/
categories:
  - Tutorial
tags:
  - CSS3
  - Dreamweaver
  - JQuery Mobile
  - Mobile
  - mobshake
  - pencil prot
  - tutorial
  - web design
---
[<img class="aligncenter size-full wp-image-232" title="blogtutorial2banner" src="/wp-content/uploads/2011/12/blogtutorial2banner.png" alt="" width="700" height="340" />][1]

&nbsp;

In the [first part of this tutorial][2] we put together a quick JQuery Mobile App and used XQuery to render our HTML5 dynamically from XML.  Now let’s take “Mobile Shakespeare” one step further by investigating some tools and web design steps to personalize the site and help users have a rewarding and memorable experience using the program.

Right now the site is too generic.  Users are dropped straight into a list of Shakespeare play titles, and may not understand what the app is about or what it can do.  Also, we are still using the JQuery Mobile default theme!  The site has no identity and probably won’t be remembered by users the next time they want to look up their favorite Shakespeare quotes (or want to follow an XQuery tutorial).

A good first step in any web reskin is to build a pencil prototype.  The idea is to make a basic line drawing of your website that focuses on big picture content, behavior, and flow.  Because we are starting with such low-fidelity rendering of the website we can avoid getting stuck in the details and even show it to sample users or stakeholders without having to worry about customers arguing about fonts and colors in committee.  You can do a pencil prototype with physical pencil and paper, but I prefer using a prototyping tool because they allow me to quickly distribute samples and incorporate feedback.  These were made with [Balsamiq][3], but I know people who use the free “Pencil” Firefox pluging, Omigraffle, Visio, or even just PowerPoint.

[<img class="aligncenter size-full wp-image-185" title="balsamiq" src="/wp-content/uploads/2011/12/balsamiq.jpg" alt="UI mockups" width="600" height="354" />][4]

I want to have a line portrait of the Bard in the background, so I&#8217;ll make a set of semi-transparent PNG files in Photoshop in various sizes.  I will use CSS3 media queries to customize the appearance of the app depending on the mobile or desktop browser size.

Let&#8217;s get started.  First we&#8217;ll ditch the default JQuery Mobile blue button theme and make our own over at [JQuery Mobile Themeroller][5].  As of writing this, the Themeroller program is still generating templates made for a slightly older version of JQuery Mobile (1.0 RC2) so I&#8217;ll just cross my fingers and hope that downgrading slightly doesn&#8217;t introduce errors that have been fixed in the latest JQuery Mobile release.

[<img class="aligncenter size-full wp-image-186" title="themeroll" src="/wp-content/uploads/2011/12/themeroll.jpg" alt="Making a new theme in ThemeRoller" width="698" height="569" />][6]

Because I want the Shakespeare head to look like a faded print on a piece of parchment, I do the following steps in Photoshop (I&#8217;m not sure how well these translate into other tools like GIMP):

  * Background layer is transparent
  * Create a new layer and fill it with a rusty auburn color that is darker than the theme&#8217;s background
  * Give the dark layer an Opacity Mask.
  * In the channels navigator, disable RGB and enable the Alpha layer.  Paste in the black and white image of Shakespeare and use the move tool and Transform (Win-CTRL / COMMAND &#8211; T) command to place the image as desired
  * Select the entire alpha layer (Win-CTRL / COMMAND &#8211; A) and then Menu > Image > Adjustments > Invert to make the portrait a negative image.  The had image should now act as a masking layer so that only the white strokes making up Will&#8217;s face allow alpha opacity for the auburn color.  The rest is transparent.
  * Switch back to editing just the RGB channels and change the layer&#8217;s opacity o 39%.
  * Save as PNG, which preserves the alpha channel.

[<img class="aligncenter size-full wp-image-248" title="alphas2" src="/wp-content/uploads/2011/12/alphas2.jpg" alt="" width="754" height="511" />][7]

I&#8217;ve also selected some fonts using [Google Web Fonts][8].  With the font link copied, the theme exported to a zip file, and my extra transparent images ready to go, it&#8217;s now time to integrate the changes to our HTML5 template


  <pre><code class="language-html html">&lt;meta name="viewport" content="width=device-width, initial-scale=1" /&gt;
        
&lt;link href='http://fonts.googleapis.com/css?family=Aguafina+Script|Vast+Shadow' rel='stylesheet' type='text/css'/&gt;
&lt;link rel="stylesheet" href="/css/mobshake-jm.min.css"/&gt;
&lt;link rel="stylesheet" href="http://code.jquery.com/mobile/1.0rc2/jquery.mobile.structure-1.0rc2.min.css"/&gt;
        
&lt;link rel="stylesheet" href="/css/mobshake.css" /&gt;
        
&lt;script src="http://code.jquery.com/jquery-1.6.4.min.js"&gt;&lt;/script&gt;
&lt;script src="http://code.jquery.com/mobile/1.0rc2/jquery.mobile-1.0rc2.min.js"&gt;&lt;/script&gt;</code></pre>


And some Media aware CSS3 to help layout the app slightly differently depending on the screen width of the mobile device


  <pre><code class="language-css css">#homepage {
	width:auto;
	text-align:center;
}

h1 {
	font-family: 'Aguafina Script', cursive; 
	font-size:1.2em;
}

#homepage h1 {
	font-family: 'Aguafina Script', cursive; 
	font-size:3em;
}	

#mobshake h2, #mobshake h3 {
	font-family: 'Vast Shadow', cursive;
}	

#play-search {
	
}

@media only screen and (max-width: 300px) {
	.ui-page {
		background:#e3b778 url(../img/shaketranstiny.png) no-repeat top center fixed;
	}
	
	#homepage h1 {
		font-family: 'Aguafina Script', cursive; 
		font-size:1.3em;	
	}	
	
	#play-search{
		margin-top:160px;	
	}
}



@media only screen and (min-width:301px) and (max-width:789px){

	.ui-page {
		background:#e3b778 url(../img/shaketranssmall.png) no-repeat top center fixed;
	}
	
	#homepage h1 {
		font-family: 'Aguafina Script', cursive; 
		font-size:2em;	
	}	
	
	#play-search{
		margin-top:250px;	
	}

}

@media only screen and (min-width: 790px) {

	.ui-page {
		background:#e3b778 url(../img/shaketransbig.png) no-repeat top right fixed;
	}

}
</code></pre>


I tend to keep a non-XQuery version of the template around so that I can work with a tool like Dreamweaver directly in HTML5.  The newest version of Dreamweaver has a WebKit browser directly embedded into the tool.  This allows a &#8220;Multiscreen&#8221; preview that mimics the behavior of 3 device sizes at a time, which can be a big productivity boost.

[<img class="aligncenter size-large wp-image-195" title="dreamweaver" src="/wp-content/uploads/2011/12/dreamweaver-1024x734.jpg" alt="Dreamweaver Multiscreen" width="510" height="365" />][9]

I am really amazed at the wide range of mobile screen resolutions that we have to consider.  Dreamweaver&#8217;s default &#8220;Tablet&#8221; size is actually the &#8220;Portrait View&#8221; of an 768 by 1024 px iPad 1.  Because my CSS3 media query stylesheet switches to it&#8217;s desktop style area at 789px, the iPad will switch which stylesheet rules are applied when I rotate it&#8217;s orientation.  My 540 x 960 px Android phone will actually use a &#8220;Desktop&#8221; stylesheet if I am in &#8220;Landscape&#8221; view, so I&#8217;ll have to be careful not assume a user is on a computer at that resolution.  Perhaps a real desktop version of the app should start at 1280px?

Well, I didn&#8217;t get to the Search API calls for easy full text search this time, but the site is looking much nicer.  [See you in part three, where we use the MarkLogic api and even tune the search indices slightly for more relevant searches!][10]

&#8211;Dave

 [1]: /wp-content/uploads/2011/12/blogtutorial2banner.png
 [2]: http://www.front2backdev.com/2011/12/10/tutorial-mobile-shakespeare-part-1/
 [3]: http://www.balsamiq.com/products/mockups
 [4]: /wp-content/uploads/2011/12/balsamiq.jpg
 [5]: http://jquerymobile.com/themeroller/
 [6]: /wp-content/uploads/2011/12/themeroll.jpg
 [7]: /wp-content/uploads/2011/12/alphas2.jpg
 [8]: http://www.google.com/webfonts
 [9]: /wp-content/uploads/2011/12/dreamweaver.jpg
 [10]: http://www.front2backdev.com/2011/12/17/tutorial-mobile-shakespeare-part-3-adding-search/ "Tutorial: Mobile Shakespeare (Part 3 – Adding Search)"