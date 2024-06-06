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
package io.dapr.components.wrappers;

import dapr.proto.components.v1.PubSubGrpc;
import dapr.proto.components.v1.Pubsub;
import io.dapr.components.domain.pubsub.PubSub;
import io.dapr.components.domain.pubsub.PublishRequest;
import io.dapr.components.domain.pubsub.PullMessageAcknowledgement;
import io.dapr.components.domain.pubsub.PullMessagesResponse;
import io.dapr.components.domain.pubsub.Topic;
import io.dapr.v1.ComponentProtos;
import io.grpc.stub.StreamObserver;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Scheduler;
import reactor.core.scheduler.Schedulers;
```

## Implement the `PubSub` interface

Create a class that implements the `PubSub` interface.

```java
public class PubSubGrpcComponentWrapper extends PubSubGrpc.PubSubImplBase {

  @Override
  public void init(Pubsub.PubSubInitRequest request, StreamObserver<Pubsub.PubSubInitResponse> responseObserver) {
    // Called to initialize the component with its configured metadata
  }

  @Override
  public void publish(Pubsub.PublishRequest request, StreamObserver<Pubsub.PublishResponse> responseObserver) {
    // Send the message to the topic
  }

  @Override
  public StreamObserver<Pubsub.PullMessagesRequest> pullMessages(
      StreamObserver<Pubsub.PullMessagesResponse> responseObserver) {
    // Until cancelled, check the topic for messages and deliver them to the Dapr runtime

}
```

Calls to the `StreamObserver` method are "long-lived", in that the method is not expected to return until canceled (for example, via the cancellationToken). The “topic” from which messages should be pulled is passed via the topic argument, while the delivery to the Dapr runtime is performed via the `need` callback. Delivery allows the component to receive notification if/when the application (served by the Dapr runtime) acknowledges processing of the message.

```java
  @Override
  public StreamObserver<Pubsub.PullMessagesRequest> pullMessages(
      StreamObserver<Pubsub.PullMessagesResponse> responseObserver) {
    // First, convert the input requests to first request and acknowledgments Flux,
    // so we can feed it to our component
    final FirstAndRestRequestStreamToFluxAdaptor<Pubsub.PullMessagesRequest> requestAdaptor;
    requestAdaptor = new FirstAndRestRequestStreamToFluxAdaptor<>((firstProto, acksProtoFlux) -> {
      // Alright... what to do when we start receiving requests?
      // First, let's convert those requests to the local domain.
      final Topic topic = Topic.fromProto(firstProto);
      final Flux<PullMessageAcknowledgement> acksFlux = acksProtoFlux.map(PullMessageAcknowledgement::fromProto);

      // Wrap everything in a Flux. This will keep uniformity with other handlers and will allow for
      // delegating multithreading processing of requests to another thread by means of subscribeOn(scheduler)
      Flux.just(acksFlux)
          // Push these requests to the component.
          .flatMap(acks -> pubSub.pullMessages(topic, acks))
          // Move processing to a different thread -- otherwise we would get stuck in the line above
          // and this method would not return. See
          // https://projectreactor.io/docs/core/release/reference/#producing.create
          .subscribeOn(scheduler, false)
          // ... connect its response flux to the output stream from this RPC
          .map(PullMessagesResponse::toProto)
          .subscribe(responseObserver::onNext, responseObserver::onError, responseObserver::onCompleted);
    });
}
```


## Register pub/sub component

In the main application file (for example, `main.java`), register the pub/sub component with the application.

```java
public PluggableComponent withPubSub(PubSub pubSub) {
  Objects.requireNonNull(pubSub);
  assert !alreadyAddedPubSub; // No, you cannot add multiple PubSubs with the same name.
  // This will be the only PubSub added to this component.
  alreadyAddedPubSub = true;

  exposedServices.add(new PubSubGrpcComponentWrapper(pubSub));

  return this;
}
```

## Next steps
- [Advanced techniques with the pluggable components Java SDK]({{< ref java-advanced >}})
- Learn more about implementing:
  - [Bindings]({{< ref java-bindings >}})
  - [State]({{< ref java-state-store >}})