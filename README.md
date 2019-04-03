# kb

Knowledge base for a software engineer.

## Kubernetes

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
  var s string = "”
  ```

  The first form may be used only within a function, not for package-level variables. In practice, you should generally use one of the first two forms, with explicit initialization to say that the initial value is important and implicit initialization to say that the initial value doesn’t matter.
