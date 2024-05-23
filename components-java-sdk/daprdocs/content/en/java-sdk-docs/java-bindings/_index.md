---
type: docs
title: "Implementing a Java input/output binding component"
linkTitle: "Bindings"
weight: 1000
description: How to create an input/output binding with the Dapr pluggable components Java SDK
no_list: true
is_preview: true
---

Creating a binding component requires just a few basic steps.

## Import bindings packages

Create the file `components/inputbinding.java` and add `import` statements for the state store related packages.

```java
package io.dapr.components.wrappers;

import dapr.proto.components.v1.Bindings;
import dapr.proto.components.v1.InputBindingGrpc;
import io.dapr.components.domain.bindings.InputBinding;
import io.dapr.components.domain.bindings.ReadRequest;
import io.dapr.components.domain.bindings.ReadResponse;
```

## Input bindings: Implement the `InputBinding` interface

Create a class that implements the `InputBinding` interface.

```java
public class InputBindingGrpcComponentWrapper extends InputBindingGrpc.InputBindingImplBase {

  private final InputBinding inputBinding;

  private final Scheduler scheduler = Schedulers.boundedElastic();

  public InputBindingGrpcComponentWrapper(InputBinding inputBinding) {
    this.inputBinding = Objects.requireNonNull(inputBinding);
  }

  @Override
  public void init(Bindings.InputBindingInitRequest request,
                   StreamObserver<Bindings.InputBindingInitResponse> responseObserver) {
    // Called to initialize the component with configured metadata
  }

  @Override
  public void ping(ComponentProtos.PingRequest request, StreamObserver<ComponentProtos.PingResponse> responseObserver) {
    // Until cancelled, checks the underlying store for messages and delivers them to the Dapr runtime
  }

}
```

Calls to the `Bindings.ReadRequest` are “long-lived”, in that it's not expected to return until cancelled. As messages are read from the underlying store of the component, they are delivered to the Dapr runtime via the `StreamObserver` callback. Delivery allows the component to receive notification if/when the application (served by the Dapr runtime) acknowledges processing of the message.

```java
@Override
public StreamObserver<Bindings.ReadRequest> read(StreamObserver<Bindings.ReadResponse> responseObserver) {
  // Convert the input requests to first request and acknowledgments Flux
  final RequestStreamToFluxAdaptor<Bindings.ReadRequest> requestAdaptor = new RequestStreamToFluxAdaptor<>();
  // Convert requests to the local domain.
  final Flux<ReadRequest> acksFlux = requestAdaptor.flux().map(ReadRequest::fromProto);

  // Wrap everything in a Flux. 
  Flux.just(acksFlux)
      // Push these requests to the component.
      .flatMap(inputBinding::read)
      .subscribeOn(scheduler, false)
      .map(ReadResponse::toProto)
      .subscribe(responseObserver::onNext, responseObserver::onError, responseObserver::onCompleted);

  // Finally, return the StreamObserver
  return requestAdaptor.requestStreamObserver();
}
```

## Output bindings: Implement the `OutputBinding` interface

Create a class that implements the `OutputBinding` interface.

```java
public class OutputBindingGrpcComponentWrapper extends OutputBindingGrpc.OutputBindingImplBase {

  private final OutputBinding outputBinding;

  public OutputBindingGrpcComponentWrapper(OutputBinding outputBinding) {
    this.outputBinding = Objects.requireNonNull(outputBinding);
  }

  @Override
  public void init(Bindings.OutputBindingInitRequest request,
                   StreamObserver<Bindings.OutputBindingInitResponse> responseObserver) {
    // Called to initialize the component with its configured metadata
  }

  @Override
  public void ping(ComponentProtos.PingRequest request, StreamObserver<ComponentProtos.PingResponse> responseObserver) {
    // Until cancelled, checks the underlying store for messages and delivers them to the Dapr runtime
  }

  @Override
  public void invoke(Bindings.InvokeRequest request, StreamObserver<Bindings.InvokeResponse> responseObserver) {
    // Called to invoke a specific operation
  }

  @Override
  public void listOperations(Bindings.ListOperationsRequest request,
                             StreamObserver<Bindings.ListOperationsResponse> responseObserver) {
    // Called to list the operations that can be invoked
  }
}
```

## Input and output binding components

A component can be _both_ an input _and_ output binding. Simply implement both interfaces and register the component as both binding types.

## Register binding component

In the main application file (for example, `main.java`), register the binding component with the application.

```java
public class PluggableComponentServer {

  public static void main(String[] args) throws IOException {

    final PluggableComponentServer server = new PluggableComponentServer();
    server.registerComponent(PluggableComponent
            .withName("my-input-binding")
            .withInputBinding(new BindingName())
        )
        .registerComponent(PluggableComponent
            .withName("my-output-binding")
            .withOutputBinding(new BindingName())
        )

        .run();
  }
}

```

## Next steps
- [Advanced techniques with the pluggable components Java SDK]({{< ref java-advanced >}})
- Learn more about implementing:
  - [State]({{< ref java-state-store >}})
  - [Pub/sub]({{< ref java-pub-sub >}})