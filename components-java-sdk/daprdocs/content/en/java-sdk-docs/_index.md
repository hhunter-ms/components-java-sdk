---
type: docs
title: "Getting started with the Dapr pluggable components Java SDK"
linkTitle: "Java"
weight: 1000
description: How to get up and running with the Dapr pluggable components Java SDK
no_list: true
is_preview: true
cascade:
  github_repo: https://github.com/dapr-sandbox/components-java-sdk
  github_subdir: daprdocs/content/en/java-sdk-docs
  path_base_for_github_subdir: content/en/developing-applications/develop-components/pluggable-components/pluggable-components-sdks/pluggable-components-java/
  github_branch: main
---

Dapr offers packages to help with the development of Java pluggable components.

## Prerequisites

- Java JDK 11 (or greater):
  - [Oracle JDK](https://www.oracle.com/java/technologies/downloads), or
  - OpenJDK
- [Apache Maven](https://maven.apache.org/install.html), version 3.x.
- [Dapr 1.9 CLI]({{< ref install-dapr-cli.md >}}) or later
- Initialized [Dapr environment]({{< ref install-dapr-selfhost.md >}})
- Linux, Mac, or Windows (with WSL)

{{% alert title="Note" color="primary" %}}
Development of Dapr pluggable components on Windows requires WSL as some development platforms do not fully support Unix Domain Sockets on "native" Windows.
{{% /alert %}}

## Project creation

Creating a pluggable component starts with an empty Java project.

```bash
mvn archetype:generate 
	-DgroupId={project-packaging}
	-DartifactId={project-name}
	-DarchetypeArtifactId={maven-template} 
	-DinteractiveMode=false
```

## Add packages

Add the Dapr Java pluggable components package to your project.

```bash
mvn install io.dapr.components
```

## Create application and service

Creating a Dapr pluggable component application is similar to creating a Java application. Under the standard `src/main/java/io/dapr/components`, `Program.cs`, replace the `WebApplication` related code with the Dapr `PluggableComponent` equivalent.

```java
package io.dapr.components.server;

public final class PluggableComponent {

  public PluggableComponent(String componentName) {
    this.name = Objects.requireNonNull(componentName);
  }

  public static PluggableComponent withName(String componentName) {
    return new PluggableComponent(componentName);
  }

  // Register a new {@link StateStore} with this pluggable component.

  public PluggableComponent withStateStore(StateStore stateStore) {
    Objects.requireNonNull(stateStore);

    exposedServices.add(new StateStoreGrpcComponentWrapper(stateStore)); 

    return this;
  }

  // Repeat for each component 
  
}
```

This creates an application with a single service. Each service:

- Corresponds to a single Unix Domain Socket
- Can host one or more component types

{{% alert title="Note" color="primary" %}}
Only a single component of each type can be registered with an individual service. However, [multiple components of the same type can be spread across multiple services]({{< ref dotnet-multiple-services >}}).
{{% /alert %}}

## Implement and register components

 - [Implementing an input/output binding component]({{< ref java-bindings >}})
 - [Implementing a pub-sub component]({{< ref java-pub-sub >}})
 - [Implementing a state store component]({{< ref java-state-store >}})

## Test components locally

Pluggable components can be tested by starting the application on the command line and configuring a Dapr sidecar to use it.

To start the component, in the application directory:

```bash
mvn 
```

To configure Dapr to use the component, in the resources path directory:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <component name>
spec:
  type: state.<socket name>
  version: v1
  metadata:
  - name: key1
    value: value1
  - name: key2
    value: value2
```

Any `metadata` properties will be passed to the component via its `stateStore.init(req.getMetadata().getPropertiesMap())` method when the component is instantiated.

To start Dapr (and, optionally, the service making use of the service):

```bash
dapr run --app-id <app id> --resources-path <resources path> ...
```

At this point, the Dapr sidecar will have started and connected via Unix Domain Socket to the component. You can then interact with the component either:
- Through the service using the component (if started), or 
- By using the Dapr HTTP or gRPC API directly

## Create Container

Pluggable components are deployed as containers that run as sidecars to the application (like Dapr itself). Create the container using the following command from within your application directory:

```bash
mvn deploy
```

## Next steps

- [Learn advanced steps for the Pluggable Component Java SDK]({{< ref "java-advanced" >}})
- Learn more about using the Pluggable Component Java SDK for:
  - [Bindings]({{< ref "java-bindings" >}})
  - [Pub/sub]({{< ref "java-pub-sub" >}})
  - [State store]({{< ref "java-state-store" >}})