---
tags:
  - Kubernetes
  - kube-scheduler
  - scheduler
---

# Network Topology Aware Scheduler

## Node Annotations

- `node.kubernetes.io/network-limit: 1Gbps` - Indicates the maximum network bandwidth of a node.

Potentially also add the following annotations:

- `node.kubernetes.io/network-baseline: 1.25Gbps` - Indicates a node's guaranteed network bandwidth baseline
- `node.kubernetes.io/network-burst: 10Gbps` - Indicates a node's potential burst bandwidth limit

## Scheduling Using Extended Resources

One can perform static scheduling with network-topology-related metrics by employing the steps outlined in [[Scheduling Using Extended Resources]].

While static scheduling is feasible to achieve that way, multiple shortcomings become apparent.

- One has to implement a controller updating each node's bandwidth resource capacities and allocatable amounts.
- As outlined in the pod definition section, the CNI specification does not consider custom resource limits. It would require creating an additional [[Mutating Webhooks||mutating webhook]] that intercepts the creation of a pod resource and sets the annotations accordingly.

Additionally, [[Scheduling Using Extended Resources]] does not allow overcommitting resources and thus cannot take network burst into account. Therefore, it would halt the scheduling of pods onto nodes exceeding their network baseline.
In contrast, pods could still be scheduled on those nodes, bursting the network throughput and returning to the network baseline.
