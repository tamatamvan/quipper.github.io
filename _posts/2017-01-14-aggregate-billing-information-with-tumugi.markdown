---
layout: post
title:  "Aggregate billing information with Tumugi"
date:   2017-01-16 10:00:00
author: kamatama41
comments: true
---

Today technology companies use a lot of cloud services to accelerate their business, of course Quipper is also.
We are using Heroku, AWS, GCP and so on. 

TODO: image for cloud service icons that we are using.

In using them, cost management is one of the biggest concern for us.
As the straightforward approach, we can see the billing page on each website every month,
but it's quite troublesome if we have to go around more than 10 websites.

TODO: image for billing page of AWS, Heroku, GCP

I want to see the all billing information at a glance.
To realize it, we chose [Datadog](https://www.datadoghq.com/) as the dashboard. 
Datadog is a cloud service used for system monitoring such as CPU, memory, disk, networking, etc.
