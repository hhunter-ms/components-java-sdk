---
type: docs
title: "Implementing a Java pub/sub component"
linkTitle: "Pub/sub"
weight: 1000
description: How to create a pub/sub component with the Dapr pluggable components Java SDK
no_list: true
is_preview: true
---

Creating a pub/sub component requires just a few basic steps.

## Import pub/sub packages

Create the file `components/pubsub.java` and add `import` statements for the pub/sub related packages.

```java
```

## Implement the `PubSub` interface

Create a type that implements the `PubSub` interface.

```java
```
Calls to the `need` method are “long-lived”, in that the method is not expected to return until canceled (for example, via the cancellationToken). The “topic” from which messages should be pulled is passed via the topic argument, while the delivery to the Dapr runtime is performed via the `need` callback. Delivery allows the component to receive notification if/when the application (served by the Dapr runtime) acknowledges processing of the message.

```java
```

## Register pub/sub component

In the main application file (for example, `main.java`), register the pub/sub component with the application.

```java
```

## Next steps
- [Advanced techniques with the pluggable components Java SDK]({{< ref java-advanced >}})
- Learn more about implementing:
  - [Bindings]({{< ref java-bindings >}})
  - [State]({{< ref java-state-store >}})