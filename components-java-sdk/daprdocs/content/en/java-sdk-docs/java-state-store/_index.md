---
type: docs
title: "Implementing a Java state store component"
linkTitle: "State Store"
weight: 1000
description: How to create a state store with the Dapr pluggable components Java SDK
no_list: true
is_preview: true
---

Creating a state store component requires just a few basic steps.

## Import state store packages

Create the file `components/statestore.java` and add `import` statements for the state store related packages.

```java
package io.dapr.components.wrappers;

import com.google.protobuf.ByteString;
import dapr.proto.components.v1.State;
import dapr.proto.components.v1.State.BulkDeleteRequest;
import dapr.proto.components.v1.State.BulkGetRequest;
import dapr.proto.components.v1.State.BulkGetResponse;
import dapr.proto.components.v1.State.BulkSetRequest;
import dapr.proto.components.v1.State.BulkStateItem;
import dapr.proto.components.v1.State.DeleteRequest;
import dapr.proto.components.v1.State.Etag;
import dapr.proto.components.v1.State.GetRequest;
import dapr.proto.components.v1.StateStoreGrpc;
import io.dapr.components.domain.state.BulkGetError;
import io.dapr.components.domain.state.GetResponse;
import io.dapr.components.domain.state.SetRequest;
import io.dapr.components.domain.state.StateStore;
import io.dapr.v1.ComponentProtos;
import io.dapr.v1.ComponentProtos.FeaturesResponse;
import io.grpc.stub.StreamObserver;
import reactor.core.publisher.Mono;
```

## Implement the `StateStore` interface

Create a class that implements the `StateStore` interface.

```java
public class StateStoreGrpcComponentWrapper extends StateStoreGrpc.StateStoreImplBase {

  @Override
  public void init(final State.InitRequest request, final StreamObserver<State.InitResponse> responseObserver) {
    // Called to initialize the component with its configured metadata
  }


  @Override
  public void features(final ComponentProtos.FeaturesRequest request,
                       final StreamObserver<FeaturesResponse> responseObserver) {
    // ??
  }

  @Override
  public void ping(final ComponentProtos.PingRequest request,
                   final StreamObserver<ComponentProtos.PingResponse> responseObserver) {
    // Until cancelled, checks the store for messages and delivers them to the Dapr runtime
  }

  @Override
  public void delete(final DeleteRequest request, final StreamObserver<State.DeleteResponse> responseObserver) {
    // Delete the requested key from the state store
  }

  @Override
  public void get(final GetRequest request, final StreamObserver<State.GetResponse> responseObserver) {
    // Get the requested key value from the state store
  }


  @Override
  public void set(final State.SetRequest request, final StreamObserver<State.SetResponse> responseObserver) {
    // Set the requested key to the specified value in the state store
  }



}
```

## Register state store component

In the main application file (for example, `main.java`), register the state store with an application service.

```java

  public PluggableComponent withStateStore(StateStore stateStore) {
    Objects.requireNonNull(stateStore);
    assert !alreadyAddedStateStore; // No, you cannot add multiple stateStores with the same name.
    alreadyAddedStateStore = true;

    exposedServices.add(new StateStoreGrpcComponentWrapper(stateStore));
    // Register other facets of a stateStore, like QueriableStateStore and TransactionalStateStore
    // if the current stateStore object supports those facets.
    if (stateStore instanceof QueriableStateStore queriableStore) {
      exposedServices.add(new QueriableStateStoreComponentWrapper(queriableStore));
    }
    if (stateStore instanceof TransactionalStateStore transactionalStateStore) {
      exposedServices.add(new TransactionalStateStoreComponentWrapper(transactionalStateStore));
    }

    return this;
  }
```

## Bulk state stores

State stores that intend to support bulk operations should implement the optional interfaces. Its methods mirror those of the base `StateStore` interface, but include multiple requested values.

```java
  @Override
  public void bulkDelete(final BulkDeleteRequest request,
                         final StreamObserver<State.BulkDeleteResponse> responseObserver) {
    // Delete all of the requested values from the state store
  }

  @Override
  public void bulkGet(final BulkGetRequest request,
                      final StreamObserver<BulkGetResponse> responseObserver) {
    // Return the values of all of the requested values from the state store
  }

  @Override
  public void bulkSet(final BulkSetRequest request, final StreamObserver<State.BulkSetResponse> responseObserver) {
    // Set all of the valeus of the requested keys in the state store
  }
```


## Transactional state stores

State stores that intend to support transactions should implement the optional `TransactionalStateStore` interface. Its `transact()` method is passed a request with a sequence of delete and/or set operations to be performed within a transaction. 

```java
```

## Queryable state stores

State stores that intend to support queries should implement the optional `QueriableStateStore` interface. `query()` is passed details about the query, such as the filter(s), result limits, pagination, and sort order(s) of the results. The state store uses those details to generate a set of values to return as part of its response.

```java
public class QueriableStateStoreComponentWrapper extends QueriableStateStoreGrpc.QueriableStateStoreImplBase {

  // ...

  @Override
  public void query(State.QueryRequest request, StreamObserver<State.QueryResponse> responseObserver) {
    // Generate and return results
  }
}
```

## ETag and other semantic error handling

The Dapr runtime has additional handling of certain error conditions resulting from some state store operations. State stores can indicate such conditions by returning specific errors from its operation logic:

| Error | Applicable Operations | Description
|---|---|---|
| `NewETagError(state.ETagInvalid, ...)` | Delete, Set, Bulk Delete, Bulk Set | When an ETag is invalid |
| `NewETagError(state.ETagMismatch, ...)`| Delete, Set, Bulk Delete, Bulk Set | When an ETag does not match an expected value |
| `NewBulkDeleteRowMismatchError(...)` | Bulk Delete | When the number of affected rows does not match the expected rows |

## Next steps
- [Advanced techniques with the pluggable components Java SDK]({{< ref java-advanced >}})
- Learn more about implementing:
  - [Bindings]({{< ref java-bindings >}})
  - [Pub/sub]({{< ref java-pub-sub >}})