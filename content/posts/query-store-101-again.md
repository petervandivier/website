+++ 
draft = true
date = 2024-06-12T12:47:18-04:00
title = ""
description = ""
slug = "query-store-101-again"
authors = ["Peter Vandivier"]
tags = ["sql-server","query-store"]
categories = []
externalLink = ""
series = []
+++

I was fortunate enough to be on an early adopter team when SQL Server Query Store was released. By fall of 2017 we were using QS in production for alerting, regression analysis, and we even introduced a weekly rotation of reviewing the built-in dashboards for prophylactic query tuning. 

Recently I find myself at a gig that hasn't yet Heard the Good Word of Query Store and have taken it upon myself to Preach said Word Unto Them. As I recall, sensible settings on Query Store make it functionally non-impacting to a production workload. That _said_... it's been 7 years since I did actual setup tuning so I kind of need to remind myself of what those sensible settings _actually are_...



As a reminder: 
