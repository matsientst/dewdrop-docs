---
layout: default
title: Command flow
nav_order: 4
description: "Command flows"
permalink: /command-flow/
---

# Commands
Eric Evans’ book Domain Driven Design describes an abstraction called “aggregate”:

> “An aggregate is a cluster of associated objects that we treat as a unit for the purpose of data changes. Each aggregate has a root and a boundary.”

Dewdrop follows this idea assuming that to mutate state we create an AggregateRoot that is used as a unit of persistent to group together related events. For example, if you have a `User` aggregate you would create a `UserAggregate` that then contains all the command handling and event handling for mutating state.

To create an AggregateRoot in Dewdrop, you need to create a class and then decorate it with the `@AggregateRoot` annotation. This tells the framework that this is the aggregateRoot object where the logic for state mutation exists. This is an example of a simple AggregateRoot: 

```java
@Aggregate
public class DewdropAccountAggregate {
    @AggregateId
    UUID accountId;
    String name;
    BigDecimal balance = BigDecimal.ZERO;

    public DewdropAccountAggregate() {}

    @CommandHandler
    public List<DewdropAccountCreated> handle(DewdropCreateAccountCommand command) {
        if (StringUtils.isEmpty(command.getName())) { throw new IllegalArgumentException("Name cannot be empty"); }
        if (command.getAccountId() == null) { throw new IllegalArgumentException("AccountId cannot be empty"); }
        if (command.getUserId() == null) { throw new IllegalArgumentException("UserId cannot be empty"); }

        return new DewdropAccountCreated(command.getAccountId(), command.getName(), command.getUserId());
    }

    @EventHandler
    public void on(DewdropAccountCreated event) {
        this.accountId = event.getAccountId();
        this.name = event.getName();
    }
}
```

To explain what is going first we need to talk about the `@Aggregate` annotation. This is a marker annotation that tells the framework that this is an aggregateRoot. The framework will use this to get the class from DI or create a class that is then wrapped by the `AggregateRoot` class. The `AggregateRoot` class is a base class that provides the basic functionality for an aggregateRoot.

The key here is to understand that there is a lifecycle to modifying an AggregateRoot. The first step is to create a command to modify the AggregateRoot. This is done by creating a class that extends the `Command` class.

### Command
```java
public class DewdropCreateAccountCommand extends DewdropAccountCommand {
    private String name;
    private UUID userId;

    public DewdropCreateAccountCommand(UUID accountId, String name, UUID userId) {
        super(accountId);
        this.name = name;
        this.userId = userId;
    }
}
```
Which as you can see extends `DewdropAccountCommand` which is a base class that wraps the `@AggregateRootId`.
```java
public abstract class DewdropAccountCommand extends Command {
    @AggregateId
    private UUID accountId;

    public DewdropAccountCommand(UUID accountId) {
        super();
        this.accountId = accountId;
    }
}
```
The `@AggregateRootId` is the identifier of the AggregateRoot. Currently the framework only supports UUID as an identifier. This id is used to read/write to the appropriate stream for the AggregateRoot. For example, if you have an AggregateRoot like the one listed above `DewdropAccountAggregate`, and a UUID of `e4cb9a9c-308f-4fb7-a3df-8ada95749c7e`, the framework would save this to a stream called `DewdropAccountAggregate-e4cb9a9c-308f-4fb7-a3df-8ada95749c7e`. And in turn, for each command that is run on this AggregateRoot after it's creation, you'll need to pass in the appropriate UUID as the `@AggregateRootId` for the framework to know which AggregateRoot to load.

In this situation we have created an abstract base class called `DewdropAccountCommand` to hold this value. And all commands that impact this AggregateRoot should extend this class. We suggest you create you're own version of this base command class for each of your AggregateRoots. 

The `Command` class is a base class that is used to create commands. All the command classes MUST extend this object for the framework to function correctly. 

This relates directly to the CQRS pattern. The `Command` class is used to create a command that is then sent to the aggregateRoot. The aggregateRoot then modifies the state of the aggregateRoot and then publishes an event that is then sent to the event store.

Here's an example test:
```java
    @Test
    private DewdropCreateAccountCommand createAccount(Dewdrop dewdrop, UUID userId) {
        DewdropCreateAccountCommand command = new DewdropCreateAccountCommand(UUID.randomUUID(), "test", userId);
        dewdrop.executeCommand(command);
        return command;
    }
```
This test creates a command to create an account and then sends it to the Dewdrop framework. The framework then sends the command to the aggregateRoot and is then handled by the `@CommandHandler` method on the AggregateRoot that has the `DewdropCreateAccountCommand` as it's first parameter.
```java
    @CommandHandler
    public List<DewdropAccountCreated> handle(DewdropCreateAccountCommand command) {
        DewdropValidator.validate(command);

        return new DewdropAccountCreated(command.getAccountId(), command.getName(), command.getUserId());
    }
```

### Validation
Part of maintaining a robust event hierarchy is jealously guarding when we create events. What this means is that it's CRITICAL to validate the commands in your AggregateRoot before you create an event. If you inadvertently create an erroneous event it becomes much harder to handle than it is to be very thorough in your validation logic in your `@CommandHandler`. Dewdrops opinion on validation logic is that ALL of the validations should run and be returned as a full collection of issues, rather than one at a time for the user to discover as they get further through your code. 

With this in mind we created the `DewdropValidator`. This is a validation object that makes it SUPER easy to validate as you wish. If you decorate your `Command` object with JSR-303 annotations, then `DewdropValidator` will go ahead and validate those appropriately. 


For example:

```java
public class DewdropUserCommand extends Command {
    @NotNull(message = "UserId is required")
    @AggregateId
    private UUID userId;

    public DewdropUserCommand(UUID userId) {
        super();
        this.userId = userId;
    }
}
```
As you can see, we have the `@NotNull` annotation which the `DewdropValidator` will validate for you. This cuts down on logic and allows for a consistent strategy on how to validate.  

However, there are times where this is not nearly enough. We aren't just concerned with if a value is present or if it's a blank string etc. In those cases, you can use `DewdropValidator.withRule()` method:

```java
DewdropValidator.withRule(() -> Validate.notBlank(command.getUsername(), "username is required"))
    .andRule(() -> Validate.notNull(command.getUserId(), "userId is required"))
    .validate();
```
You can pass any logic you want in the lambda and it will run that. If there is an exception it will collect those exceptions and throw a `ValidationException` with all of the exceptions that were found. 

### Next checkout out events work:

[Event Flow](/event-flow){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }