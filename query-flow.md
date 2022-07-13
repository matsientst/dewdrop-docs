---
layout: default
title: Query flow
nav_order: 6
description: "Query flows"
permalink: /query-flow/
---

# Queries
The query side relies on creation of ReadModels that are used to read from the event store. Think of a ReadModel as similar to a repository or a DAO in more traditional architectures. It is the object that defines how we read data from the event store. Now, your first intuition is to map it directly to your AggregateRoot, which can be correct at times, but more often it is not. 

Dewdrop will read the json events from the event store, find which ReadModels are listening to those events, convert them into the event objects and then replay those event on the ReadModel.

Usually, when we are reading from an event store we are looking to read from a complex interaction of events. If you start to think of what data you're looking to read as a collection of events it can help with architecting your ReadModel correctly. 

So, first off, let's look at an example ReadModel in Dewdrop
```java
@ReadModel
@Stream(name = "DewdropFundsAddedToAccount", streamType = StreamType.EVENT)
@Stream(name = "DewdropAccountCreated", streamType = StreamType.EVENT)
public class DewdropAccountSummaryReadModel {
    @DewdropCache
    DewdropAccountSummary dewdropAccountSummary;

    @QueryHandler
    public DewdropAccountSummary handle(DewdropAccountSummaryQuery query) {

        return dewdropAccountSummary;
    }
}
``` 
Here are the important bits:
* The `@ReadModel` annotation is used to mark the class as a ReadModel
* The `@Stream` annotation is used to identify what streams to read from the event store
* The `@DewdropCache` annotation is used to cache the data in the ReadModel
* The `@QueryHandler` annotation is used to identify the method that will handle the query



### Next checkout out how ReadModels work:

[Read Models](/read-models){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }