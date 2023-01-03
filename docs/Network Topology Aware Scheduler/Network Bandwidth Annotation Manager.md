---
tags:
  - Kubernetes
  - extensible-admission-controllers
---

# Network Bandwidth Annotation Manager

As part of this thesis and to create a generic way of handling networking-related resource requests, I made a small mutating webhook admission controller for Kubernetes.

The [Network Bandwidth Annotation Manager], abbreviated to `nbam`, is an admission webhook that enables one to alter pod objects before the apiserver persists them to its storage, thus before any scheduling has taken place.

The primary motivation behind creating nbam is the ability to use extended resource FQDNs in pod resource requests, as many helm charts or other packaged Kubernetes deployments do not allow setting custom pod annotations, as required by the CNI spec.
Yet, one can usually set CPU and memory limits in helm charts or Kubernetes primitives. Thus nbam takes care of rewriting those to the corresponding pod annotations in multiple modes.

## Setup

- Setup a local Kubernetes cluster using [[k3d]]
- Install [[cert-manager]].
- Build and push an OCI image using the following:

  ```bash
  # Build the OCI image
  docker build -t nbam:latest "."
  # Tag it to use the local k3d managed registry
  docker tag nbam:latest k3d-default-registry.localhost:9090/nbam:latest
  # Push it to the local registry
  docker push k3d-default-registry.localhost:9090/nbam:latest
  ```

- Deploy the annotation manager using [this manifest](https://github.com/ThomasK33/network-bandwidth-annotation-manager/blob/main/deployment.yaml).

- Add network-related resources to nodes as outlined in [[Scheduling Using Extended Resources#Adding Network Related Resources To Nodes]]

- Deploy a test pod using the following manifest and then check for set annotations:

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: nbam-test
    labels:
      nbam-mode: "overwrite"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    namespace: nbam-test
  spec:
    containers:
    - name: my-container
      image: k3d-default-registry.localhost:9090/nbam:latest
      resources:
        requests:
          cpu: 2
          # if the pod contains ingress and egress bandwidth annotations
          # the requests will be automatically set to the annotations values
          networking.k8s.io/ingress-bandwidth: 1M
          networking.k8s.io/egress-bandwidth: 1M
        limits:
          cpu: 4
          # Limits the ingress bandwidth to 1Mbit/s
          networking.k8s.io/ingress-bandwidth: 2M
          # Limits the egress bandwidth to 1Mbit/s
          networking.k8s.io/egress-bandwidth: 2M
  ```

[Network Bandwidth Annotation Manager]: https://github.com/ThomasK33/network-bandwidth-annotation-manager/
