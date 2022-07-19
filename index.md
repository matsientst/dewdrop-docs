---
layout: default
title: Dewdrop Home 
nav_order: 1
permalink: /
author: Matt Macchia
image: /assets/images/dewdrop.jpg
description: "Dewdrop is an opinionated, simple and powerful framework for implementing event sourcing in Java. The idea of Dewdrop is to make it easy to build an event driven system easily and quickly by pushing all the complex reading, writing and marshalling of events down deep into the framework allowing your team to focus on building out the business logic in terms of AggregateRoot behavior, Query logic, and ReadModel composition."


---

# Dewdrop
{: .fs-9 }
## The Java Event Sourcing Framework
Dewdrop is an opinionated, simple and powerful framework for implementing event sourcing in Java. The idea of Dewdrop is to make it easy to build an event driven system easily and quickly by pushing all the complex reading, writing and marshalling of events down deep into the framework allowing your team to focus on building out the business logic in terms of AggregateRoot behavior, Query logic, and ReadModel composition. 

If you're new to event sourcing we highly suggest understanding the main concepts around the ideas of CQRS, DDD, Event Storming and Event Modeling.

Dewdrop is in its early stages of development and is not yet production ready. If you're interested in using it in a production environment, please reach out. We are looking to continue testing and devloping until it's ready for prime time. If you're only interested in using Dewdrop as a tool to understand event sourcing or for a simple project, then Dewdrop is perfect for you. We'd also love to hear your feedback!

Currently, Dewdrop only supports using [EventStore](https://www.eventstore.com/) as its backing store. We will be expanding it to use other data stores and would love your feedback on what is most interesting to you in terms of next steps.

Requirements:
* Java 11 or later
* EventStoreDB 

{: .fs-6 .fw-300 }

[Get started now](getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View it on GitHub](https://github.com/matsientst/dewdrop){: .btn .fs-5 .mb-4 .mb-md-0 }
