---
layout: default
title: DewdropCache
nav_order: 9
description: "DewdropCache"
permalink: /dewdrop-cache/
---

# DewdropCache
This annotation marks the field as a cache. This is used to cache the data in the ReadModel. 
```java
 @DewdropCache
 Map<UUID, DewdropAccountDetails> cache;
```
This is one of the huge benefits of Dewdrop. Dewdrop will automatically based on the ReadModel and streams decorating the class be able to read the events from the event store and populate the ReadModel. This is a powerful and elegant way to get up and running with event sourcing without having to build a HUGE amount of plumbing.
These events are replayed back to the ReadModel and the ReadModel is updated. You can choose NOT to have a cache for a ReadModel by adding not adding the `@DewdropCache` annotation field. In this scenario, you should create the `@EventHandler` on the ReadModel itself.


There are two types of caches:
* Map - This is a map that is used to cache the data in the ReadModel. The key is the id of the object and the value is the DTO that you want to map to (more on this later).
* Single item - This is a single item that is used to cache the data in the ReadModel. There is no key since this becomes an accumulator. So if, for example, you want to cache the total balance of all the accounts in the system you can use this.

This is a ridiculously useful feature and is very useful. However, be careful about what is stored in cache since it is not persisted and can get quite large.

#### How to use the cache
Both types of caches (map and single) assume that you have a DTO that you are writing state to. This should be a simple POJO, but it will need the `@EventHandler` methods decorating the methods to update state (much like the AggregateRoot).
The Cache DTO is made of a few annotations that dewdrop uses to know how to create and manage the cache:
* `@DewdropCache` - This is the annotation that marks the field as a cache. This is used to cache the data in the ReadModel.
* `@PrimaryKey` - This is the annotation that marks the field as the primary key. This is used to identify the primary key of the cache.
* `@SecondaryKey` - This is the annotation that marks the field as a secondary key. This is used to identify the secondary key of the cache.
* `@CreationEvent` - This is the annotation that marks the field as the event that marks the creation of the cache item. This is used to know to create the DTO and starting using it for that key.
* `@EventHandler` - This is the annotation that marks the method that updates the cache. This is used to update the DTO with the event.

For example:
```java
public class DewdropAccountDetails {
    @PrimaryCacheKey
    private UUID accountId;
    private String name;
    private BigDecimal balance = BigDecimal.ZERO;
    @ForeignCacheKey
    private UUID userId;
    private String username;

    @EventHandler
    public void on(DewdropAccountCreated event) {
        this.accountId = event.getAccountId();
        this.name = event.getName();
        this.userId = event.getUserId();
    }

    @EventHandler
    public void on(DewdropFundsAddedToAccount event) {
        this.balance = this.balance.add(event.getFunds());
    }

    @EventHandler
    public void on(DewdropUserCreated userCreated) {
        this.username = userCreated.getUsername();
    }
}
```
In this example, we are reading from two streams as outlined above in the ReadModel example:
```java
@Stream(name = "DewdropAccountAggregate", subscribed = true)
@Stream(name = "DewdropUserAggregate", subscribed = false)
```

This object becomes the intersection of those two streams and replays events from each. The framework will automatically call the `@EventHandler` methods to update the state of the object based on the event received. 
In this case, we have an event `DewdropAccountCreated` which we create a method to handel like:
```java
    @EventHandler
    public void on(DewdropAccountCreated event) {
        this.accountId = event.getAccountId();
        this.name = event.getName();
        this.userId = event.getUserId();
    }
```
#### @EventHandler
You can implement any `@EventHandler` methods to handle any events you wish to from a stream. You can handle them all, or ignore the ones that are not relevant.

The events that are handled here are the same events we created when modifying the `DewdropAccountAggregate`.

This event is a special event, so let's dive a little deeper into it. 
```java
@CreationEvent
public class DewdropAccountCreated extends DewdropAccountEvent {
    private String name;
    private UUID userId;

    public DewdropAccountCreated(UUID accountId, String name, UUID userId) {
        super(accountId);
        this.name = name;
        this.userId = userId;
    }
}
```
#### @CreationEvent
This event has the annotation of `@CreationEvent` which means that this event is the event that is used to create the cache item. Without it, Dewdrop has no idea which event is the starting point and when to create this object.

#### @PrimaryCacheKey
The `@PrimaryCacheKey` annotation is used to identify the field that is used as the key for the cache. This relates to the `@AggregateId` annotation. The framework cache first looks for the `@PrimaryCacheKey` annotation and if it finds it, it uses that field as the key and then looks for an `@AggregateId` annotation that matches it. 

#### @ForeignCacheKey
The `@ForeignCacheKey` annotation is used to identify the field that is the foreign key used as the key for the cache. For example, In this example, we have a `@ForeignCacheKey` annotation on the `userId` field. This means that when it finds the `userId` field in the events it will relate to that field.

### No Cache
If you decide to not use a cache, then you should skip adding the `@DewdropCache` field. When you do this, you should add the `@EventHandler` method to the ReadModel itself. If you want to persist the current state in a local datastore you can make your ReadModel a spring object and then inject your repository into the ReadModel. Then on each event you can update your repository with the new state.

You also need to add a `@StartFromPosition` decorated method that returns a long to retrieve the last version number from your cache to tell the framework where to start for that Stream. The `@StreamStartPosition` name and streamType must match the `@Stream` name and type on the same ReadModel. 

```java
    @StreamStartPosition(name = "DewdropAccountAggregate")
    public Long startPosition() {
        Optional<Long> position = accountDetailsRepository.lastAccountVersion();
        return position.orElse(0L);
    }
```

### Next checkout out how to pull it all together:

[All Together](/all-together){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }