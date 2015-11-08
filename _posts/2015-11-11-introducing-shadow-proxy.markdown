---
layout: post
title:  "Introducing Shadow Proxy"
date:   2015-11-11 11:00:00
author: kamatama41
comments: true
---

Today I introduce a system that we call **Shadow Proxy**, which is released recently in our development environment.

## What's Shadow Proxy?
**Shadow Proxy** makes an HTTP request, which is copied from a request to production, to staging applications.
It means that it is possible to apply requests that are the same quality of production.
The purpose of this system is to check beforehand the impact of modifying an index on MongoDB.
Our service has been running for more than two years, and the data size is now enormous.
Therefore we have to take care to **add**/**remove**/**forget** a index because it might cause a serious problem on production.

This system consists of [Fluentd](http://www.fluentd.org/)(td-agent) and its [plugins](http://www.fluentd.org/plugins).
Fluentd is an open-source data collector. It has robustness and flexibility.
[td-agent](http://docs.fluentd.org/articles/install-by-rpm) is the stable distribution of Fluentd, which is provided
by [Treasure Data, Inc](http://www.treasuredata.com/).

## Architecture

![Image](https://cloud.githubusercontent.com/assets/899217/11015974/79abd19a-85b6-11e5-93b2-20a466de176d.png)

### Reverse Proxy
Proxy HTTP requests to production by Nginx. It forwards access logs to Log Aggregator
by [forward Output Plugin](http://docs.fluentd.org/articles/out_forward).

### Log Aggregator
A hub of the log stream. It receives access logs by [forward Input Plugin](http://docs.fluentd.org/articles/in_forward)
and forwards logs to [Google BigQuery](https://cloud.google.com/bigquery/), [Amazon S3](https://aws.amazon.com/s3/)
and Shadow Proxy by [BigQuery Output Plugin](https://github.com/kaizenplatform/fluent-plugin-bigquery), [S3 Output Plugin](http://docs.fluentd.org/articles/out_s3) and forward Output Plugin.

### Shadow Proxy
Receive access logs from Log Aggregator and make HTTP requests from logs
by [http shadow Output Plugin](https://github.com/quipper/fluent-plugin-http_shadow). We forked the original and added some features.

`td-agent.conf` of Shadow Proxy is like below.

```
<source>
  type forward
  port 24224
</source>

<match nginx.access>
  type http_shadow
  protocol_format https

  host_key domain
  path_format ${path}
  method_key method
  cookie_key cookie
  body_key request_body

  # Resolve staging hosts
  host_hash {
    "a-production-app-1.quipper.com": "a-staging-app-1.quipper.com",
    "a-production-app-2.quipper.com": "a-staging-app-2.quipper.com"
  }

  # Create HTTP Headers
  header_hash {
   "Referer": "${referer}",
   "User-Agent": "${agent}",
   "Authorization": "${authorization}",
   "Content-Type": "${content_type}"
  }

  # Load balancing
  max_concurrency 2
  timeout 5
  rate 100
  rate_per_host_hash {
    "a-staging-app-1.quipper.com": 25,
    "a-staging-app-2.quipper.com": 100
  }
  followlocation false

  num_threads 24
  flush_interval 1s
  buffer_type file
  buffer_path /var/log/td-agent/buffer/http_shadow

  retry_limit 0
  buffer_chunk_limit 256k
  buffer_queue_limit 3600
</match>
```

When Shadow Proxy receives the following input, 

```
nginx.access: {
  "domain":"a-production-app-1.quipper.com",
  "method":"POST",
  "path":"/some/path",
  "referer":"https://a-production-app-1.quipper.com",
  "agent":"Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36",
  "content_type":"application/json",
  "cookie":"name1=value1, name2=value2",
  "authorization":"xxxxxxxxxxxxxxxxxxxx",
  "request_body":"{\\x22key1\\x22:\\x22value1\\x22,\\x22key2\\x22:\\x22value2\\x22}"
}
```

it will send the following HTTP request to `a-staging-app-1.quipper.com`.
 
```
POST /some/path HTTP/1.1
Host: a-staging-app-1.quipper.com
Referer: https://a-production-app-1.quipper.com
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36
Authorization: xxxxxxxxxxxxxxxxxxxx
Content-Type: application/json
Cookie: name1=value1, name2=value2
{"key1":"value1","key2":"value2"}
```

## Summary
- We released **Shadow Proxy** to check the impact on MongoDB before an index on production is modified.
- Shadow proxy makes an HTTP request from Nginx access log in production by Fluentd plugin.
- Fluentd is really useful!
