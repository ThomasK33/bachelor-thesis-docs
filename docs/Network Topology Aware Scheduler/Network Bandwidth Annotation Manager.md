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
- Deploy the annotation manager using [this manifest](https://github.com/ThomasK33/network-bandwidth-annotation-manager/blob/main/deployments/manager.yaml).
- Add network-related resources to nodes as outlined in [[Scheduling Using Extended Resources#Adding Network Related Resources To Nodes]]
- Deploy examples located at: <https://github.com/ThomasK33/network-bandwidth-annotation-manager/blob/main/examples>


[Network Bandwidth Annotation Manager]: <https://github.com/ThomasK33/network-bandwidth-annotation-manager/>
