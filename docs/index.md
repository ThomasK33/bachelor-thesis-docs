---
tags:
  - Kubernetes
  - thesis
  - uni
---

# Bachelor Thesis

## Goal

Create a network-topology-aware scheduler for [Kubernetes](https://kubernetes.io/).

## Comparable Work

### Topology Aware Scheduling

In 2020 a general-purpose [[Topology Aware Scheduling]] algorithm was implemented in the [[Kubernetes Scheduler]].

The original use case focuses on implementing NUMA-aware scheduling and introducing generalized [[Custom Resources]] to provide capacity information to the scheduler.

## Analysis

[[Network Topology Aware Scheduler/index|Network Topology Aware Scheduler]]

## Target Implementation

Subject to change if needed.

1. Define annotations to be added to nodes and pods to define networking bandwidth limits.
2. Then extend the [[Kubernetes Scheduler|kube-scheduler]] to use the annotations, extending the scheduling algorithm while drawing inspiration from the [[Topology Aware Scheduling]] already present in the [[Kubernetes Scheduler|kube-scheduler]].
3. The extended scheduler can run in parallel, as one can configure [multiple schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/) in a cluster.
4. Furthermore, one could create a [[Mutating Webhooks|mutating webhook]] that overrides the [`schedulerName`](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#scheduling) field in a pod's object and force it to use a specific scheduler.

TODO:

- Create custom metrics for the [[Metrics Server]] containing a node's maximum network bandwidth and, optionally, current bandwidth usage.

## Resources

- <https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/>
- <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#extended-resources>
