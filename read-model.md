---
layout: default
title: Read Models
nav_order: 7
description: "Read Models"
permalink: /read-models/
---

# Read Models
As you can see we have annotated this class with `@ReadModel`. Dewdrop on startup looks to identify ALL the classes that it can find that implement `@ReadModel` and then keeps track of them.
When it finds a ReadModel it will assume that we automatically want to create that ReadModel unless we tell it not to.

If we have a situation where we don't want a ReadModel to be created and live forever, we can mark the ReadModel as `ephemeral` which tells Dewdrop to only create this class when we see a query executed for this. Consider it a lazy ReadModel, or a just in time ReadModel. If we want to have an `ephemeral` ReadModel we can then choose what that means by adding the `destroyInMinutesUnused` field to our `@ReadModel` annotation on our class.
For example:
```java
@ReadModel(ephemeral = true, destroyInMinutesUnused = ReadModel.DESTROY_IMMEDIATELY)
@Stream(name = "DewdropAccountAggregate", subscribed = true)
@Stream(name = "DewdropUserAggregate", subscribed = false)
public class DewdropAccountDetailsReadModel {
    @DewdropCache
    Map<UUID, DewdropAccountDetails> cache;

    @QueryHandler
    public Result<DewdropAccountDetails> handle(DewdropGetAccountByIdQuery query) {
        DewdropAccountDetails dewdropAccountDetails = cache.get(query.getAccountId());
        if (dewdropAccountDetails != null) { return Result.of(dewdropAccountDetails); }
        return Result.empty();
    }
}
```
Here we've told Dewdrop that we want to create this ReadModel when we see a query executed for this ReadModel. We also told Dewdrop that we want to destroy this ReadModel immediately after it was used. This is a good way to keep your ReadModels from growing too large and eating up memory. However, this would be a bad strategy to use in a stream that has a lot of events associated with. This would become an expensive operation, and would take a long time to load if there were thousands of events.
The three states of `destroyInMinutesUnused` to note are:
* `DESTROY_IMMEDIATELY` - This will destroy the ReadModel immediately after it was used.
* `NEVER_DESTROY` - This will add the ReadModel to the cache, but it will never be destroyed until restart (just like a non-ephemeral ReadModel).
* `n` - Just add a value here for the count of minutes you want it to live on for before destruction.

Destroying a ReadModel after an hour of use is a good idea for situations where you have a UI that is being heavily used by a user and you want to keep the ReadModel alive for snappy performance. However, after that user is done we can then destroy this ReadModel and clear up memory.

### Next checkout out how Streams work:

[Streams](/streams){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }