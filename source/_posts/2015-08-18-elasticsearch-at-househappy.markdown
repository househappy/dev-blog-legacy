---
layout: post
title: "How We use Elasticsearch at Househappy"
date: 2015-08-18 16:36:26 -0700
comments: true
categories:
---
## How we use Elasticsearch

Earlier in 2015, Househappy switched its search autocomplete from
[Soulmate](https://github.com/seatgeek/soulmate) to
[Elasticsearch](https://www.elastic.co/products/elasticsearch).
More recently, we switched the main search results from PostgreSQL to
Elasticsearch. Here, I'll describe the Elasticsearch integration.

## Technical Specs

* Househappy uses one Elasticsearch daemon, running on a VPS
* The daemon uses 1.4 GB of memory
* The VPS uses 38 GB of storage

### Search Autocomplete

![Autocomplete UI](http://cl.ly/image/45331v01272P/portland-autocomplete-ui.jpg)

The autocomplete endpoint returns Property Listings, schools, zip codes, neighborhoods, cities, metro areas and states that match the
phrase the user is typing into the search box.

The JavaScript frontend calls an API, which in turn builds and executes the Elasticsearch queries and returns the results. For Property Listings,
the response from the API for the above query looks like this:

![Property Listings matching "port"](http://cl.ly/image/1C0m312t0u3d/download/autocomplete-listings-endpoint.jpg)

For cities, it looks like this:

![Cities matching "port"](http://cl.ly/image/0w1u2j3g0y3x/cities-autocomplete-ui.jpg)

For autocomplete, The Elasticsearch query is a fully-analyzed text search.

![Autocomplete Query to Elasticsearch](http://cl.ly/image/0T1K1n0R3j3W/autocomplete_es_query.jpg)

See the entire [JSON query here](https://gist.github.com/moxley/d8935387133476db0ba0).
It's 174 lines of formatted JSON. The Elasticsearch's query DSL is not the
prettiest language I've seen. It kind of reminds me of [Postscript](https://en.wikipedia.org/wiki/PostScript), in that it is a language
meant to be written by programs.

### "Gridview" Search Results

![Primary Search Results](http://cl.ly/image/1M2M1a2s3Y1L/search-results.jpg)

The primary ("gridview") search generates a much more simple Elasticsearch
query than the autocomplete. It finds properties with an exact-match by foreign key:

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "in": {
            "city_id": "32"
          }
        },
        {
          "in": {
            "public_status": "available"
          }
        }
      ]
    }
  }
}
```

In the above example, it searches Property Listings belonging to `city_id=32` (Portland, OR),
with a public status of `available`. We also add filters and sorting to this
query, based on the filters and sorts the user selects.

Before using Elasticsearch for the primary search, we queried PostgreSQL
directly. The primary table has millions of records, and the search query joined as
many as 10 other tables. After switching to Elasticsearch for the query, using
denormalized data, site performance improved dramatically.

![Performance Before and After](https://pbs.twimg.com/media/CMDxrGWVEAA4k7a.jpg:large)

## How Documents are Added

### Converting Models to Documents

We use the amazing
[representable gem](https://github.com/apotonick/representable) to
convert an ActiveRecord instance to an Elasticsearch document. This is the
same gem we use to serialize ActiveRecord instances to API responses, and to
deserialize API requests to ActiveRecord instances.

We define a "representer" that essentially defines the schema of the
document. That is, it describes which model attributes and which associations,
and which association attributes will be in the document. It can also add
virtual attributes that don't even exist on the model.
Once the representer is defined, it's a simple piece of code to convert a model
instance the Elasticsearch document JSON.

```ruby
metro_area = MetroArea.first
representer = MetroAreaRepresenter.new(metro_area)
metro_area_json = representer.to_json
```

The resulting JSON:

```json
{
  "id": 338,
  "name": "Portland-Vancouver-Hillsboro, OR-WA",
  "location": "POINT (-122.4843457 45.6006204)",
  "viewport": "..."
}
```

We use the [elasticsearch gem](https://github.com/elastic/elasticsearch-ruby)
to save the document to Elasticsearch.

### Batch Loading

For every type of Elasticsearch document we store, we created a `rake` task
that loads every corresponding record of that type from PostgreSQL to
Elasticsearch.

### Incremental Synchronizing

We use ActiveRecord callbacks that in turn call asynchronous Sidekiq jobs to
synchronize model changes to Elasticsearch. The Sidekiq workers follow the
publish-subscribe pattern, where a model creates a job that publishes the fact it has changed, and that job creates a job for every subscriber of that type of
message. The subscribers in this case are concerned with inspecting the change,
making the decision to take action or ignore it, and if action is warranted,
generate and save a document to Elasticsearch.

For example, we maintain Property Listing documents for properties that are up for sale. These properties have one or more photos. The URLs
and meta-data of these photos are stored with the property document in
Elasticsearch. When a new photo is added, the property document must be updated
with the new photo.

In the Photo model, there is an ActiveRecord listener that generates a
"broadcast" Sidekiq worker. This worker, in turn, generates Sidekiq jobs
for all the subscribers subscribed to changes to the Photo model. One of these
subscribers is the `SyncProperty` subscriber, which takes the Photo change,
generates a property document and saves that document to Elasticsearch.

We use Sidekiq to synchronize model changes to Elasticsearch because adding
documents to Elasticsearch takes time, and it adds risk to the ActiveRecord
lifecycle. This extra time and risk is too much for batch imports that import
thousands of properties every hour.

We use the pub-sub pattern because it's unreasonable to expect the ActiveRecord
callbacks to know all the different Elasticsearch document types that need to
re-synchronize when a particular model changes.

## Challenges

* Documents made up of data from multiple ActiveRecord models
* Synchronizing changes from one model to document types of another model
* Keeping the number of synchronization jobs low
* Deploying changes to document schemas
