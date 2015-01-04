---
published: false
layout: post
title: "Single Page App"
date: 2015-01-03 19:11:25 -0800
comments: true
categories:
---
* New SpineJS UI, recently released
* Statically-deployed JavaScript and CSS assets to CDN
* iOS in ObjectiveC, recently released

* Rails-based API
* Cluster that processes property listings

* VMs
  * Database
    * One PostgreSQL master
  * Web
    * Four web servers running Ngnix and Rails
    * 6 Sidekiq instances
    * Redis instance
    * Memcached instance
    * Elasticsearch instance
    * HTML generator for search bots
    * IP-Geo service
  * Datafeed processing:
    * Five servers, each with one Sidekiq process
    * One Redis instance, mostly for Sidekiq
    * Rails web server for monitoring
  * Miscellaneous
    * Chef server
    * WebUI deploy
    * Jenkins cluster
    * Staging servers

* Pain points
  * Limited database connections
  * Large database, slow queries

* Future architecture evolution:
  * Expand use of service classes
  * Expand pub-sub event system
  * Continue to break out standalone services
  * Photo download and processing service
  * Simplified ActiveRecord models
    * Just an ORM
    * No callbacks. Use service classes instead.
    * Remove validations around presence of associated objects. Use service
      classes and tests instead.
