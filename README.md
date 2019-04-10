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

## Golang

### Basics

- Go code is organized into packages, which are similar to libraries or modules in other languages. A package consists of one or more `.go` source files in a single directory that define what the package does.
- Package `main` is special. It defines a standalone executable program, not a library. Within package main the function main is also special—it’s where execution of the program begins. Whatever main does is what the program does.

### Packages and Imports

- A program will not compile if there are missing imports or if there are unnecessary ones.

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
- A `map` holds a set of key/value pairs and provides constant-time operations to store, retrieve, or test for an item in the set. The key may be of any type whose values can be compared with `==`, strings being the most common example; the value may be of any type at all. Think about `map` as a Python dictionary.