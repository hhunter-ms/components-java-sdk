---
type: docs
title: "Advanced uses of the Dapr pluggable components Java SDK"
linkTitle: "Advanced"
weight: 2000
description: How to use advanced techniques with the Dapr pluggable components Java SDK
is_preview: true
---

While not typically needed by most, these guides show advanced ways you can configure your Java pluggable components.

## Component lifetime

A pluggable component can host multiple components of varying types. You might do this:

- To minimize the number of sidecars running in a cluster  
- To group related components that are likely to share libraries and implementation, such as:  
  - A database exposed both as a general state store, and
  - Output bindings that allow more specific operations.  

Each Unix Domain Socket can manage calls to one component of each type. To host multiple components of the same type, you can spread those types across multiple sockets. The SDK binds each socket to a “service”, with each service composed of one or more component types

## Registering multiple services

Each call to `.registerComponent` binds a socket to a registered pluggable component. One of each component type (input/output binding, pub/sub, and state store) can be registered per socket.

```java
public class PluggableComponentServer {

  public static void main(String[] args) throws IOException {

    final PluggableComponentServer server = new PluggableComponentServer();
    server.registerComponent(PluggableComponent
            .withName("my-state-store")
            .withStateStore(new StateStoreName())
        )
        .registerComponent(PluggableComponent
            .withName("my-output-binding")
            .withOutputBinding(new BindingName())
        )
    server.registerComponent(PluggableComponent
            .withName("another-state-store")
            .withStateStore(new StateStoreName())
        )

        .run();
  }
}
```

## Configuring multiple components

Configuring Dapr to use the hosted components is the same as for any single component - the component YAML refers to the associated socket. 

```yaml
#
# This component uses the state store associated with socket ``
#
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: my-state-store
spec:
  type: state.service
  version: v1
  metadata: []
```

```yaml
#
# This component uses the state store associated with socket ``
#
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: another-state-store
spec:
  type: state.service
  version: v1
  metadata: []
```

## Next steps
- Learn more about implementing:
  - [Bindings]({{< ref java-bindings >}})
  - [State]({{< ref java-state-store >}})
  - [Pub/sub]({{< ref java-pub-sub >}})