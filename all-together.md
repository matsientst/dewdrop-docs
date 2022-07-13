---
layout: default
title: All Together
nav_order: 10
description: "All togeter"
permalink: /all-together/
---

# Querying
To bring this all together, to query your ReadModel all you need to do is create a query object and then call the `query` method on the ReadModel.

```java
public class DewdropGetAccountByIdQuery {
    private UUID accountId;
}
```
The query objects are nothing special and can be whatever type of objects you wish. They do not extend any special classes. They are just plain old POJO's.

To query a ReadModel, you just pass your query object to the `executeQuery(query)` method on dewdrop.

```java
DewdropGetAccountByIdQuery query = new DewdropGetAccountByIdQuery(accountId);
dewdrop.executeQuery(query);
```
The framework will automatically find the ReadModel associated with the Query, construct it (if needed) and then call the `@QueryHandler` method on it.
Then, the `@QueryHandler` method will be called on the ReadModel and it will return the result of the query.

```java
    @QueryHandler
    public Result<DewdropAccountDetails> handle(DewdropGetAccountByIdQuery query) {
        DewdropAccountDetails dewdropAccountDetails = cache.get(query.getAccountId());
        if (dewdropAccountDetails != null) { return Result.of(dewdropAccountDetails); }
        return Result.empty();
    }
```
In this query, since there is a `@DewdropCache`, you just need to look into the cache to find the object and return it!

If you don't have cache, then you can query against a local datastore or however, you want to architect it.

### That's it!
If you have any questions or issues feel free to reach out to [Better Than Brian](mailto:info@betterthanbrian.com)