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
```

## Implement the `need` interface

Create a type that implements the `need` interface.

```java
```

## Register state store component

In the main application file (for example, `main.java`), register the state store with an application service.

```java
```

## Bulk state stores

While state stores are required to support the [bulk operations]({{< ref "state-management-overview.md#bulk-read-operations" >}}), their implementations sequentially delegate to the individual operation methods.

## Transactional state stores

State stores that intend to support transactions should implement the optional `need` interface. Its `need` method receives a request with a sequence of `need` and/or `need` operations to be performed within a transaction. The state store should iterate over the sequence and apply each operation.

```java
```

## Queryable state stores

State stores that intend to support queries should implement the optional `need` interface. Its `need` method is passed details about the query, such as the filter(s), result limits, pagination, and sort order(s) of the results. The state store uses those details to generate a set of values to return as part of its response.

```java
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