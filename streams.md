---
layout: default
title: Streams
nav_order: 8
description: "Streams"
permalink: /streams/
---

# Streams
The `@Stream` annotation is used to identify what streams to read from the event store. You can have multiple `@Stream` annotations onto a ReadModel. To identify what stream you want to read from the Stream has these fields:
* `name` - The name of the stream to read from
* `streamType` - The type of the stream you want to read from which you can use the enum `StreamType` (`EVENT`, `CATEGORY` or `AGGREGATE`) to identify. `CATEGORY` is default
* `subscribed` - Whether or not you want to subscribe to the stream. If you want to subscribe to the stream you can leave it blank since the default is `true`.
* `direction` - Which direction you want to read - you can use the enum `Direction` (`FORWARD` or `BACKWARD`)

The name annotation should NOT be the full name that you see in your event store. In this case we use a name generator to create the name that is needed. For example 
For the example above, you can see we passed in a name of 

`DewdropAccountAggregate` 

Since the Stream default type is `CATEGORY` we end up generating the name

`$ce-DewdropAccountAggregate` 

If the stream type is `EVENT` we end up generating the name

`$et-DewdropAccountAggregate` 

But, that wouldn't make sense since this is an AggregateRoot name, a better example would be:

`$et-DewdropAccountCreated`

### Next checkout out how @DewdropCache work:

[DewdropCache](/dewdrop-cache){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }