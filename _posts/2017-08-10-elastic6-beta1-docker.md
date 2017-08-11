---
layout: post
title:  "End-to-end docker-compose example for Elastic Stack 6 beta"
date:   2017-08-10 8:00:00
categories: elasticsearch, docker, beta
permalink: /2017/08/10/elastic6-beta1-docker/
published: true
---

![Elastic stack docker](/images/posts/2017-08-10-diagram.png "docker diagram")

I took a few minutes this morning and updated my end-to-end docker example of the Elastic stack to the 6.0 beta1 which released this week.  There are new official docker images for the 6.0 betas for Elasticsearch, Logstash, Kibana, and Filebeat (the last of which I have not yet incorporated)

* [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docker.html)
* [Kibana](https://www.elastic.co/guide/en/kibana/6.0/_configuring_kibana_on_docker.html)
* [Logstash](https://www.elastic.co/guide/en/logstash/6.0/docker.html) (instructions somewhat sparse)
* [Filebeat](https://www.elastic.co/guide/en/beats/filebeat/6.0/running-on-docker.html)

Note that I've had to enable ```xpack.security.enabled=false``` as ES 6 has new bootstrap checks for security.  Consider this example a dev only playground.  


