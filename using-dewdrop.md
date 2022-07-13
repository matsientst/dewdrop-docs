---
layout: default
title: Using Dewdrop
nav_order: 3
description: "How to use dewdrop"
permalink: /using-dewdrop/
---

# Using Dewdrop
## Core Concepts of Dewdrop
Dewdrop follows a few architectural principles religiously to make it easy to build a system that is event driven. The following are the core concepts of Dewdrop:
* DDD - Domain Driven Design
* CQRS - Command Query Responsibility Separation
* Event sourcing - Event driven architecture

If you are unfamiliar with these principles, we highly suggest diving deeper and understanding them before jumping into Dewdrop. It will make much more sense when you're familiar with these concepts.

## Messages
At it's heart Dewdrop has an interface `Message` to represent both commands and events. This root interface is implemented by an abstract class `AbstractMessage` that then go down the chain until we get to the `events.dewdrop.structure.api.Command` object and the `events.dewdrop.structure.api.Event` object.
To create a command or an event you must extend one of these classes.

## Aggregates
The next step is creating an AggregateRoot that can be used by the system:
```java
@Aggregate
public class DewdropAccountAggregate {
    @AggregateId
    UUID accountId;
    String name;
    BigDecimal balance = BigDecimal.ZERO;

    public DewdropAccountAggregate() {}

    @CommandHandler
    public List<DewdropAccountCreated> handle(DewdropCreateAccountCommand command) throws ValidationException {
        DewdropValidator.validate(command);

        return List.of(new DewdropAccountCreated(command.getAccountId(), command.getName(), command.getUserId()));
    }

    @CommandHandler
    public List<DewdropFundsAddedToAccount> handle(DewdropAddFundsToAccountCommand command) {
        if (command.getAccountId() == null) { throw new IllegalArgumentException("Id cannot be empty"); }

        DewdropFundsAddedToAccount dewdropFundsAddedToAccount = new DewdropFundsAddedToAccount(command.getAccountId(), command.getFunds());
        return List.of(dewdropFundsAddedToAccount);
    }

    @EventHandler
    public void on(DewdropAccountCreated event) {
        // validate here as well different
        // check that teh aggregate invariance are always true
        this.accountId = event.getAccountId();
        this.name = event.getName();
        // DewdropAccountAggregate.from(this).with();
    }

    @EventHandler
    public void on(DewdropFundsAddedToAccount event) {
        // this.accountId = event.getAccountId();
        this.balance = this.balance.add(event.getFunds());
    }
}
```


## Using Dewdrop
The easiest way to use Dewdrop is to use the `Dewdrop` class which encapsulates the entire framework. The crazy simple API is:

`dewdrop.executeCommand(command)`

`dewdrop.executeSubsequentCommand(command, earlierCommand)`

`dewdrop.executeQuery(query)`

The framework will take care of the rest.

### Next checkout out commands work:

[Command Flow](/command-flow){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }