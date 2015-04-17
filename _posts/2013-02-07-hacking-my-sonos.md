---
title: Hacking my Sonos
author: Dave
layout: post
permalink: /2013/02/07/hacking-my-sonos/
categories:
  - Experiments
tags:
  - Hex
  - SOAP
  - Sonos
  - Wireshark
  - XQuery
---
The previous project of [controlling the lights in my living room with REST calls][1] has given me an idea for a home automation project which will probably get written up here over a few more posts.  Next step &#8230; controlling my sound system!  As reverse-engineering practice, I decided to do this without documentation or Google.

My home stereo is hooked up to a [Sonos CONNECT][2], which is an internet audio hub that aggregates all my various Pandora, Spotify and other streaming services.  It&#8217;s awesome, and just like the Philips Hue lights it is controlled by desktop and mobile apps across your home WiFi network.  Because I&#8217;m doing this without documentation, first step is to pull out [Wireshark][3], a packet monitoring tool to see what&#8217;s going on in the TCP/IP traffic.  With a Wireshark recording session active, I clear my Sonos queue, enqueue a playlist from Spotify, and then hit play.

[<img class="alignnone size-medium wp-image-487" alt="wireshark" src="/wp-content/uploads/2013/02/wireshark-300x237.jpg" width="300" height="237" />][4]

&nbsp;

Filtering the recording with the Sonos&#8217; IP address we can see from the above screenshot that the Sonos is controlled with UPnP SOAP messages, so not that much more complicated than the REST and JSON I sent to the Philips Hue.  Wireshark exports detailed packet extracts to an XML file type called PDML.  I load that file into a text editor and take a look.


  <pre><code class="language-xml xml">    &lt;proto name="fake-field-wrapper"&gt;
      &lt;field name="data" value="474554202f67657461613f733d3126753d782d736f6e6f732d73706f7469667925336173706f746966792532353361747261636b2532353361324f5a525278714f736f704b32764250515630794f642533667369642533643132253236666c6167732533643020485454502f312e310d0a434f4e4e454354494f4e3a20636c6f73650d0a4143434550543a202a2f2a0d0a4143434550542d454e434f44494e473a20677a69700d0a484f53543a2031302e302e312e323a313430300d0a555345522d4147454e543a20536f6e6f730d0a0d0a"&gt;
  &lt;field name="data.data" showname="Data: 474554202f67657461613f733d3126753d782d736f6e6f73..." size="210" pos="66" show="47:45:54:20:2f:67:65:74:61:61:3f:73:3d:31:26:75:3d:78:2d:73:6f:6e:6f:73:2d:73:70:6f:74:69:66:79:25:33:61:73:70:6f:74:69:66:79:25:32:35:33:61:74:72:61:63:6b:25:32:35:33:61:32:4f:5a:52:52:78:71:4f:73:6f:70:4b:32:76:42:50:51:56:30:79:4f:64:25:33:66:73:69:64:25:33:64:31:32:25:32:36:66:6c:61:67:73:25:33:64:30:20:48:54:54:50:2f:31:2e:31:0d:0a:43:4f:4e:4e:45:43:54:49:4f:4e:3a:20:63:6c:6f:73:65:0d:0a:41:43:43:45:50:54:3a:20:2a:2f:2a:0d:0a:41:43:43:45:50:54:2d:45:4e:43:4f:44:49:4e:47:3a:20:67:7a:69:70:0d:0a:48:4f:53:54:3a:20:31:30:2e:30:2e:31:2e:32:3a:31:34:30:30:0d:0a:55:53:45:52:2d:41:47:45:4e:54:3a:20:53:6f:6e:6f:73:0d:0a:0d:0a" value="474554202f67657461613f733d3126753d782d736f6e6f732d73706f7469667925336173706f746966792532353361747261636b2532353361324f5a525278714f736f704b32764250515630794f642533667369642533643132253236666c6167732533643020485454502f312e310d0a434f4e4e454354494f4e3a20636c6f73650d0a4143434550543a202a2f2a0d0a4143434550542d454e434f44494e473a20677a69700d0a484f53543a2031302e302e312e323a313430300d0a555345522d4147454e543a20536f6e6f730d0a0d0a"/&gt;
	&lt;field name="data.len" showname="Length: 210" size="0" pos="66" show="210"/&gt;
      &lt;/field&gt;
    &lt;/proto&gt;</code></pre>


The payload of the data is HEX-ified.  This makes sense given that the TCP/IP traffic could be binary.  Knowing that all the SOAP data is most likely UTF-8 encoded, I&#8217;ll just write a quick XQuery to de-hexify the data.


  <pre><code class="language-xquery xquery">xquery version("1.0-ml");

let $d := fn:doc("/wireshark.xml")
for $p in $d//packet
let $hex :=  fn:string($p//proto[@name eq "fake-field-wrapper"]/field[@name eq "data"]/@value)
let $len := fn:string-length($hex)
return

  fn:string-join((
		for $i in (1 to $len)
		where $i mod 2 eq 1
		return
		let $char := fn:substring($hex, $i, 2)
		return
			fn:codepoints-to-string( xdmp:hex-to-integer( $char )))
	,"")</code></pre>


Digging through a bunch of service calls to get album covers etc, there are three important SOAP calls that actually do the work I want to repeat.


  <pre><code class="language-text text">POST /MediaRenderer/AVTransport/Control HTTP/1.1
CONNECTION: close
ACCEPT-ENCODING: gzip
HOST: 10.0.1.2:1400
USER-AGENT: Linux UPnP/1.0 Sonos/19.4-59140 (MDCR_MacBookPro6,2)
CONTENT-LENGTH: 290
CONTENT-TYPE: text/xml; charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:AVTransport:1#RemoveAllTracksFromQueue"

&lt;s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"&gt;&lt;s:Body&gt;&lt;u:RemoveAllTracksFromQueue xmlns:u="urn:schemas-upnp-org:service:AVTransport:1"&gt;&lt;InstanceID&gt;0&lt;/InstanceID&gt;&lt;/u:RemoveAllTracksFromQueue&gt;&lt;/s:Body&gt;&lt;/s:Envelope&gt;


POST /MediaRenderer/AVTransport/Control HTTP/1.1
CONNECTION: close
ACCEPT-ENCODING: gzip
HOST: 10.0.1.2:1400
USER-AGENT: Linux UPnP/1.0 Sonos/19.4-59140 (MDCR_MacBookPro6,2)
CONTENT-LENGTH: 1246
CONTENT-TYPE: text/xml; charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:AVTransport:1#AddURIToQueue"

&lt;s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"&gt;&lt;s:Body&gt;&lt;u:AddURIToQueue xmlns:u="urn:schemas-upnp-org:service:AVTransport:1"&gt;&lt;InstanceID&gt;0&lt;/InstanceID&gt;&lt;EnqueuedURI&gt;x-rincon-cpcontainer:1006006cspotify%3auser%3adavejunk1%3aplaylist%3a1OzDDLJ98nincl2cbREWiT&lt;/EnqueuedURI&gt;&lt;EnqueuedURIMetaData&gt;&lt;DIDL-Lite xmlns:dc=&quot;http://purl.org/dc/elements/1.1/&quot; xmlns:upnp=&quot;urn:schemas-upnp-org:metadata-1-0/upnp/&quot; xmlns:r=&quot;urn:schemas-rinconnetworks-com:metadata-1-0/&quot; xmlns=&quot;urn:schemas-upnp-org:metadata-1-0/DIDL-Lite/&quot;&gt;&lt;item id=&quot;1006006cspotify%3auser%3adavejunk1%3aplaylist%3a1OzDDLJ98nincl2cbREWiT&quot; parentID=&quot;100a0064playlists&quot; restricted=&quot;true&quot;&gt;&lt;dc:title&gt;Electrofunkish Soul&lt;/dc:title&gt;&lt;upnp:class&gt;object.container.playlistContainer&lt;/upnp:class&gt;&lt;desc id=&quot;cdudn&quot; nameSpace=&quot;urn:schemas-rinconnetworks-com:metadata-1-0/&quot;&gt;SA_RINCON3079_davejunk1&lt;/desc&gt;&lt;/item&gt;&lt;/DIDL-Lite&gt;&lt;/EnqueuedURIMetaData&gt;&lt;DesiredFirstTrackNumberEn
queued&gt;0&lt;/DesiredFirstTrackNumberEnqueued&gt;&lt;EnqueueAsNext&gt;0&lt;/EnqueueAsNext&gt;&lt;/u:AddURIToQueue&gt;&lt;/s:Body&gt;&lt;/s:Envelope&gt;


POST /MediaRenderer/AVTransport/Control HTTP/1.1
CONNECTION: close
ACCEPT-ENCODING: gzip
HOST: 10.0.1.2:1400
USER-AGENT: Linux UPnP/1.0 Sonos/19.4-59140 (MDCR_MacBookPro6,2)
CONTENT-LENGTH: 266
CONTENT-TYPE: text/xml; charset="utf-8"
SOAPACTION: "urn:schemas-upnp-org:service:AVTransport:1#Play"

&lt;s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"&gt;&lt;s:Body&gt;&lt;u:Play xmlns:u="urn:schemas-upnp-org:service:AVTransport:1"&gt;&lt;InstanceID&gt;0&lt;/InstanceID&gt;&lt;Speed&gt;1&lt;/Speed&gt;&lt;/u:Play&gt;&lt;/s:Body&gt;&lt;/s:Envelope&gt;

</code></pre>


SOAP is in general annoying in that the XML payload is actually passed as an escaped string rather than just embedded XML, making it confusing to read and parse.  I believe this practice goes back to early XML web service processors doing a horrible job serializing and deserializing the XML just to  proxy it around.

BOOM! ([source code][5]) I can now play Funk music from qconsole (I&#8217;ll spare you the Youtube video until the full project is done)

 [1]: http://www.front2backdev.com/2013/02/03/controlling-philips-hue-with-xquery/ "Controlling Philips Hue with XQuery"
 [2]: http://www.sonos.com/shop/products/connect
 [3]: http://www.wireshark.org/
 [4]: /wp-content/uploads/2013/02/wireshark.jpg
 [5]: https://github.com/derickson/ml-utils/blob/master/xquery/lib/l-sonos.xqy