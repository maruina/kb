# kb

Knowledge base for a software engineer.

## Kubernetes

### `vm.swappiness` on a Kubernetes node

By default `kubelet` disable swap on a Linux machine with [this commit](https://github.com/kubernetes/kubernetes/commit/f4edaf2b8c32463d6485e2c12b7fd776aef948bc). This is in line with [Docker best practices](https://success.docker.com/article/node-using-swap-memory-instead-of-host-memory):

> using a value of vm.swappiness=0 for Docker environments, which prevents swapping except in the case of an OOM (OutOfMemory) condition.
