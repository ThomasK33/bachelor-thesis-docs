---
tags:
  - Kubernetes
  - development-environment
---

# k3d

> k3d is a lightweight wrapper to run [k3s](https://github.com/rancher/k3s) (Rancher Lab’s minimal Kubernetes distribution) in docker.
> k3d makes it very easy to create single- and multi-node [k3s](https://github.com/rancher/k3s) clusters in docker, e.g. for local development on Kubernetes.[^1]

## Installation

For installation options, refer to the official k3d documentation: <https://k3d.io/v5.4.6/#installation>

## Usage

### Registry Creation

Create a local registry by uring:

```bash
k3d registry create default-registry.localhost --port 9090
```

### Cluster Creation

Create a local development cluster consisting of 3 nodes using:

```bash
k3d cluster create default --servers 3 --registry-use k3d-default-registry.localhost:9090
```

This will also setup a k3d-managed registry[^2], yet it requires changes made to the local hosts file[^3].

### Teardown

Delete a local development cluster by running:

```bash
k3d cluster delete default
k3d registry delete default-registry.localhost
```

[^1]: <https://k3d.io/v5.4.6/>
[^2]: <https://k3d.io/v5.2.1/usage/registries/#using-k3d-managed-registries>
[^3]: <https://k3d.io/v5.2.1/usage/registries/#preface-referencing-local-registries>
