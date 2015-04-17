---
title: Hosting multiple MarkLogic app servers on port 80
author: Dave
layout: post
permalink: /2011/12/12/hosting-multiple-marklogic-app-servers-on-port-80/
categories:
  - Software Tip
tags:
  - apache
  - apache2
  - hosting
  - MarkLogic
  - mod_proxy
---
Hosting more than one MarkLogic site on a server can be tricky to the uninitiated in Apache configuration.  This is not my area of expertise by any means; so given that I had to figure it out yesterday after once again forgetting how it works I think I&#8217;ll write it down here for the next time I need it.  Each MarkLogic application server listens on a different high numbered port.  Given the potential of draconian firewalls rules for both your hosting provider and users, typically the only reliable HTTP port will be 80.  Here&#8217;s what I do on Linux and Mac OSX (sorry Windows):

  * Set up my MarkLogic machine in cloud hosting (I&#8217;ve used both Amazon EC2 and Rackspace) or on on a spare machine with a dependable internet connection / power supply.
  * Running 40GB instances of MarkLogic in production is now free with the [MarkLogic Express License][1].  Woot!

  * Register <domain>.com with a hosting provider or a domain registry
  * The hosting provider usually will have a website where I can edit my DNS rules
  * Create an A rule for <subdomain1> and direct it to my ML machine&#8217;s IP
  * Create a second A rule for <subdomain2> and direct it to the ML machine&#8217;s IP

  * On the ML machine, install Apache2 and edit the /etc/apache2/httpd.conf file to load the following modules (add to the end of the LoadModule section if it isn&#8217;t loaded already):

<pre>LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so</pre>

  * Add the following virtual host records to the very bottom of the file

<pre>NameVirtualHost *
&lt;VirtualHost *&gt;
ServerName subdomain1.domain.com
ProxyRequests Off
    &lt;Proxy *&gt;
        Order deny,allow
        Allow from all
    &lt;/Proxy&gt;

    ProxyPass / http://localhost:9001/
    ProxyPassReverse / http://localhost:9001/
    &lt;Location /&gt;
        Order allow,deny
        Allow from all
    &lt;/Location&gt;
&lt;/VirtualHost&gt;

&lt;VirtualHost *&gt;
ServerName subdomain2.domain.com
ProxyRequests Off
    &lt;Proxy *&gt;
        Order deny,allow
        Allow from all
    &lt;/Proxy&gt;

    ProxyPass / http://localhost:9002/
    ProxyPassReverse / http://localhost:9002/
    &lt;Location /&gt;
        Order allow,deny
        Allow from all
    &lt;/Location&gt;
&lt;/VirtualHost&gt;</pre>

  * When done editing the file restart Apache2 with the following command:

<pre>sudo /usr/sbin/apachectl restart</pre>

<div>
  Now requests for subdomain1.domain.com will go to port 80 of my ML machine and then internally within the server be proxied (reverse proxied actually) to port 9001.
</div>

<div>
  &#8211;Dave
</div>

 [1]: http://developer.marklogic.com/licensing