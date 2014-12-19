---
published: false
layout: post
title: "The Datafeed Pipeline"
date: 2014-12-18 20:48:03 -0800
comments: true
categories:
---
* Starts with a multi-gigabyte XML file.
* Go program streams through file and creates a Sidekiq job for each listing
* A series of Sidekiq workers process a single listing
* Property addresses are normalized and matched with geo location data
* Listing photos are downloaded until a good one is found
* Property is published when everything is valid
* Process is littered with metrics that feed graphs that indicate health of the system.
