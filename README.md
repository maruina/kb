# kb

Knowledge base for a software engineer.

## Kubernetes

### Pods

- A pod is in a terminal state if `.status.phase in (Failed, Succeeded) is true`.

### API Server

#### Admission Controller

- An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. The controllers are compiled into the kube-apiserver binary, and may only be configured by the cluster administrator.
- Admission controllers may be `validating`, `mutating`, or both. Mutating controllers may modify the objects they admit; validating controllers may not.
- The admission control process proceeds in two phases. In the first phase, mutating admission controllers are run. In the second phase, validating admission controllers are run. Note again that some of the controllers are both.
- In addition to sometimes mutating the object in question, admission controllers may sometimes have side effects, that is, mutate related resources as part of request processing. Incrementing quota usage is the canonical example of why this is necessary. Any such side-effect needs a corresponding reclamation or reconciliation process, as _a given admission controller does not know for sure that a given request will pass all of the other admission controllers_.

### ResourceQuota

- A resource quota, defined by a `ResourceQuota` object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.
- The `ResourceQuota` admission controller will observe the incoming request and ensure that it does not violate any of the constraints enumerated in the `ResourceQuota` object in a `Namespace`.
- Use cases

  1. Ability to enumerate resource usage limits per namespace.
  1. Ability to monitor resource usage for tracked resources.
  1. Ability to reject resource usage exceeding hard quotas.

- The rationale for accounting for the requested amount of a resource versus the limit is the belief that a user should only be charged for what they are scheduled against in the cluster.
- If one wants to restrict the ratio between the request and limit, it is encouraged that the user define a LimitRange with LimitRequestRatio to control burst out behavior. This would in effect, let an administrator keep the difference between request and limit more in line with tracked usage if desired.
- A `resource quota controller` monitors observed usage for tracked resources in the `Namespace`. If there is observed difference between the current usage stats versus the current ResourceQuota.Status, the controller posts an update of the currently observed usage metrics to the ResourceQuota via the `/status` endpoint.
- The `resource quota controller` is the only component capable of monitoring and recording usage updates after a DELETE operation since admission control is incapable of guaranteeing a DELETE request actually succeeded.
- The ResourceQuota plug-in introspects all incoming admission requests: it makes decisions by evaluating the incoming object against all defined `ResourceQuota.Status.Hard` resource limits in the request namespace. If acceptance of the resource would cause the total usage of a named resource to exceed its hard limit, the request is denied.
- If the incoming request does not cause the total usage to exceed any of the enumerated hard resource limits, the plug-in will post a ResourceQuota.Status document to the server to atomically update the observed usage based on the previously read ResourceQuota.ResourceVersion. This keeps incremental usage atomically consistent, but does introduce a bottleneck (intentionally) into the system.
- ResourceQuotas are independent of the cluster capacity.

#### Reference

- [https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
- [https://github.com/kubernetes/community/blob/master/contributors/design-proposals/resource-management/admission_control_resource_quota.md](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/resource-management/admission_control_resource_quota.md)

### `vm.swappiness` on a Kubernetes node

By default `kubelet` disable swap on a Linux machine with [this commit](https://github.com/kubernetes/kubernetes/commit/f4edaf2b8c32463d6485e2c12b7fd776aef948bc). This is in line with [Docker best practices](https://success.docker.com/article/node-using-swap-memory-instead-of-host-memory):

> using a value of vm.swappiness=0 for Docker environments, which prevents swapping except in the case of an OOM (OutOfMemory) condition.

### RBAC

- A `Role` defines rules that specify a set of resources and a set of verbs, which are actions that can be taken on those objects. But they say nothing about who can perform those actions on those objects.
- A RoleBinding links a role to an identity. This might be a user, a group, or a service account. This adds the “who” part.
- Roles and RoleBindings apply to a namespace. There are also cluster-wide equivalents called ClusterRoles and ClusterRoleBindings.
- To verify users permissions: `kubectl auth can-i <verb> <resource> –as=<identity>`

## Golang

### Basics

- Go code is organized into packages, which are similar to libraries or modules in other languages. A package consists of one or more `.go` source files in a single directory that define what the package does.
- Package `main` is special. It defines a standalone executable program, not a library. Within package main the function main is also special—it’s where execution of the program begins. Whatever main does is what the program does.

### Packages and Imports

- A program will not compile if there are missing imports or if there are unnecessary ones.
- Packages in Go serve the same purposes as libraries or modules in other languages, supporting modularity, encapsulation, separate compilation, and reuse.
- Each package serves as a separate name space for its declarations.
- Packages also let us hide information by controlling which names are visible outside the package, or exported. In Go, exported identifiers start with an upper-case letter.

### Variables

- If it is not explicitly initialized, it is implicitly initialized to the zero value for its type, which is 0 for numeric types and the empty string "" for strings.
- `_` is the blank identifier. It may be used whenever syntax requires a variable name but program logic does not, for instance to discard an unwanted loop index when we require only the element value.
- There are several ways to declare a string variable

  ```go
  s := ""
  var s string
  var s = ""
  var s string = ""
  ```

  The first form may be used only within a function, not for package-level variables. In practice, you should generally use one of the first two forms, with explicit initialization to say that the initial value is important and implicit initialization to say that the initial value doesn’t matter.

#### Pointers

- A variable is a piece of storage containing a value. Variables created by declarations are identified by a name, such as x, but many variables are identified only by expressions like x[i] or x.f. All these expressions read the value of a variable, except when they appear on the left-hand side of an assignment, in which case a new value is assigned to the variable.
- A `pointer` value is the address of a variable. A pointer is thus the location at which a value is stored. Not every value has an address, but every variable does. With a pointer, we can read or update the value of a variable indirectly, without using or even knowing the name of the variable, if indeed it has a name.
  If a variable is declared var x int, the expression `&x` ("address of x") yields a pointer to an integer variable, that is, a value of type *int, which is pronounced "pointer to int." If this value is called p, we say "p points to x". The variable to which p points is written *p. The expression \*p yields the value of that

- Another way to create a variable is to use the built-in function `new`. The expression `new(T)` creates an unnamed variable of type T, initializes it to the zero value of T, and returns its address, which is a value of type \*T.
- The lifetime of a package-level variable is the entire execution of the program. By contrast, local variables have dynamic lifetimes: a new instance is created each time the declaration statement is executed, and the variable lives on until it becomes unreachable, at which point its storage may be recycled. Function parameters and results are local variables too; they are created each time their enclosing function is called.
- How does the garbage collector know that a variable’s storage can be reclaimed? The basic idea is that every package-level variable, and every local variable of each currently active function, can potentially be the start or root of a path to the variable in question, following pointers and other kinds of references that ultimately lead to the variable. If no such path exists, the variable has become unreachable, so it can no longer affect the rest of the computation.
- A name declared inside a syntactic block is not visible outside that block.

### Data Types

#### Strings

- A string is an immutable sequence of bytes. The built-in `len` function returns the _number of bytes_ in a string, and the index operation `s[i]` retrieves the _i-th byte_ of string s, where 0 ≤ i < len(s).
- Immutability means that it is safe for two copies of a string to share the same underlying memory, making it cheap to copy strings of any length. Similarly, a string s and a substring like s[7:] may safely share the same data, so the substring operation is also cheap.
- Four standard packages are particularly important for manipulating strings: `bytes`, `strings`, `strconv`, and `unicode`.
  - The strings package provides many functions for searching, replacing, comparing, trimming, splitting, and joining strings.
  - The bytes package has similar functions for manipulating slices of bytes, of type []byte, which share some properties with strings. Because strings are immutable, building up strings incrementally can involve a lot of allocation and copying. In such cases, it’s more efficient to use the bytes.Buffer type.
  - The strconv package provides functions for converting boolean, integer, and floating-point values to and from their string representations, and functions for quoting and unquoting strings.
  - The unicode package provides functions like IsDigit, IsLetter, IsUpper, and IsLower for classifying runes.

#### Array

- An array is a fixed-length sequence of zero or more elements of a particular type.
- In an array literal, if an ellipsis `...` appears in place of the length, the array length is determined by the number of initializers.
- The size of an array is part of its type, so [3]int and [4]int are different types. The size must be a constant expression.

#### Slices

- Slices represent variable-length sequences whose elements all have the same type. A slice type is written `[]T`, where the elements have type T.
- A slice has three components: a pointer, a length, and a capacity. The pointer points to the first element of the array that is reachable through the slice, which is not necessarily the array’s first element. The length is the number of slice elements; it can’t exceed the capacity, which is usually the number of elements between the start of the slice and the end of the underlying array. The built-in functions len and cap return those values.

#### Map (dict)

- A `map` holds a set of key/value pairs and provides constant-time operations to store, retrieve, or test for an item in the set. The key may be of any type whose values can be compared with `==`, strings being the most common example; the value may be of any type at all. Think about `map` as a Python dictionary.
- `map` is a reference to a hash table, and a map type is written `map[K]V` (or `map[string]int{}`), where K and V are the types of its keys and values. All of the keys in a given map are of the same type, and all of the values are of the same type, but the keys need not be of the same type as the values. The key type K must be comparable using ==, so that the map can test whether a given key is equal to one already within it.
- You’ll often see these two statements combined, like this:

  ```go
  if age, ok := ages["bob"]; !ok { /* ... */ }
  ```

  Subscripting a map in this context yields two values; the second is a boolean that reports whether the element was present. The boolean variable is often called ok, especially if it is immediately used in an if condition.

#### Structs

- A struct is an aggregate data type that groups together zero or more named values of arbitrary types as a single entity. Each value is called a field.

#### JSON

- A `field tag` is a string of metadata associated at compile time with the field of a struct.

  ```go
  Year  int  `json:"released"`
  Color bool `json:"color,omitempty"`
  ```

### Functions

```go
func name(parameter-list) (result-list) {
    body
}
```

- The type of a function is sometimes called its signature. Two functions have the same type or signature if they have the same sequence of parameter types and the same sequence of result types
- Every function call must provide an argument for each parameter, in the order in which the parameters were declared. Go has no concept of default parameter values, nor any way to specify arguments by name, so the names of parameters and results don’t matter to the caller except as documentation.

### Goroutine

- A `goroutine` is a concurrent function execution.
- A `channel` is a communication mechanism that allows one goroutine to pass values of a specified type to another goroutine.
- `“ch := make(chan string)` creates a channel of strings.
- When one goroutine attempts a send or receive on a channel, it blocks until another goroutine attempts the corresponding receive or send operation, at which point the value is transferred and both goroutines proceed.

## Prometheus

- A metric with `container_name="POD"` refers to the `pause` container.

## Envoy

### Glossary

- **Host**: An entity capable of network communication (application on a mobile phone, server, etc.).
- **Downstream**: A downstream host connects to Envoy, sends requests, and receives responses; it's the _client_
- **Upstream**: An upstream host receives connections and requests from Envoy and returns responses; it's the _server_
- **Cluster**: A cluster is a group of logically similar upstream hosts that Envoy connects to.

### Service discovery

When an upstream cluster is defined in the configuration, Envoy needs to know how to resolve the members of the cluster. This is known as service discovery.

#### Strict DNS

When using strict DNS service discovery, Envoy will continuously and asynchronously resolve the specified DNS targets. Each returned IP address in the DNS result will be considered an explicit host in the upstream cluster. Note that Envoy never synchronously resolves DNS in the forwarding path. At the expense of eventual consistency, there is never a worry of blocking on a long running DNS query.

#### Endpoint discovery service (EDS)

The endpoint discovery service is a xDS management server based on gRPC or REST-JSON API server used by Envoy to fetch cluster members. The cluster members are called “endpoint” in Envoy terminology.

### API

#### core.Locality

Identifies location of where either Envoy runs or where upstream hosts run.

```json
{
  "region": "...",
  "zone": "...",
  "sub_zone": "..."
}
```

where:

- region: (string) Region this zone belongs to.
- zone: (string) Defines the local service zone where Envoy is running. Though optional, it should be set if discovery service routing is used and the discovery service exposes zone data, either in this message or via --service-zone. The meaning of zone is context dependent, e.g. Availability Zone (AZ) on AWS, Zone on GCP, etc.
- sub_zone: (string) When used for locality of upstream hosts, this field further splits zone into smaller chunks of sub-zones so they can be load balanced independently. This can be for example a cluster.
