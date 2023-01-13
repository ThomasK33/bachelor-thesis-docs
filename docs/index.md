---
tags:
  - Kubernetes
  - thesis
  - uni
---

# Bachelor Thesis

## Goal

The goal of this Bachelor thesis is the creation of a network-topology-aware scheduler for [Kubernetes](https://kubernetes.io/).

## Comparable Work

### Topology Aware Scheduling

In 2020 a general-purpose [[Topology Aware Scheduling]] algorithm was implemented in the [[Kubernetes Scheduler]].

The original use case focuses on implementing NUMA-aware scheduling and introducing generalized [[Custom Resources]] to provide capacity information to the scheduler.

## State of the Art

Currently, scheduling is available with [[Network Topology Aware Scheduler/index#Scheduling Using Extended Resources|extended resources]].
In upcoming Kubernetes versions, which will stabilize [Device Plugins], using extended resources will become a viable scheduling mechanism for, e.g., FPGA devices, NICs, and GPUs.
Yet, many things could be improved regarding the nature of network bandwidth, which can burst at most major cloud providers.

Thus creating a specific scheduler plugin that can incorporate particular network bandwidth characteristics into its ratings is desirable.
For example, pods managed by a Job or CronJob controller can still be placed onto nodes at their baseline, as those pods will have a start and an ending, thus enabling the option to use network burst without impacting the running pods in any way.

## Target Implementation

Subject to change if needed.

1. Define annotations to be added to nodes and pods to define networking bandwidth limits. --> see [[Network Topology Aware Scheduler/index|Network Topology Aware Scheduler]]
2. A [[Mutating Webhooks|mutating webhook]] that overrides the [`schedulerName`](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#scheduling) field in a pod's object and forces it to use a specific scheduler. --> see [[Network Bandwidth Annotation Manager]]
3. Then extend the [[Kubernetes Scheduler|kube-scheduler]] to use the annotations, extending the scheduling algorithm while drawing inspiration from the [[Topology Aware Scheduling]] already present in the [[Kubernetes Scheduler|kube-scheduler]].
4. The extended scheduler can run in parallel, as one can configure [multiple schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/) in a cluster.

TODO:
- Create custom metrics for the [[Metrics Server]] containing a node's maximum network bandwidth and, optionally, current bandwidth usage.

## Resources

- <https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/>
- <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#extended-resources>

[Device Plugins]: <https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/>
