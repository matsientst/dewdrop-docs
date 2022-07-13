---
layout: default
title: Event flow
nav_order: 5
description: "Event flows"
permalink: /event-flow/
---

# Events
The framework runs this method to generate the needed events. It then sends the events to the their applicable @EventHandler methods on the AggregateRoot. In this case:
```java
    @EventHandler
    public void on(DewdropAccountCreated event) {
        this.accountId = event.getAccountId();
        this.name = event.getName();
    }
```
Much like the `Command` class, the `Event` classes are DTO's used to represent the state of the AggregateRoot. The `Event` classes are used to both read/write to the event store and to publish events to the event store.

```java
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
As you can see, this class also extends a base `Event` class and if we look at that:
```java
public abstract class `DewdropAccountEvent` extends Event {
    @AggregateId
    private UUID accountId;

}
```
Like the `DewdropAccountCommand` class, the `DewdropAccountCreated` class extends the abstract `DewdropAccountEvent` class which holds the `@AggregateRootId`. Again, this is the identifier of the AggregateRoot.
We highly suggest that you create base event classes for each of your AggregateRoots. This is a good way to keep your code clean and easy to maintain.

### Lifecycle of the AggregateRoot

To understand better what dewdrop is doing we want to outline exactly what the framework is doing under the covers. To help illustrate this, here is the lifecycle that occurs:
* User created code calls `dewdrop.executeCommand(command)`
* The framework looks for the AggregateRoot that implements `@CommandHandler` with the Command object as it's first parameter
* If the command object has an `@AggregateRootId` then the framework will load the AggregateRoot from the event store and replay all existing events to rebuild state
* It then calls the `@CommandHandler` method on the AggregateRoot with the Command object as it's first parameter, which returns either a single or list of events (extends Event)
* It then calls the `@EventHandler` method on the AggregateRoot with the Event object as it's first parameter to mutate the state of the AggregateRoot
* It then persists the events to the event store


### Next checkout out queries work:

[Query Flow](/query-flow){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }