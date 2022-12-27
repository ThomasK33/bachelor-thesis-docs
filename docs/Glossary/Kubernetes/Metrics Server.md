---
tags:
  - Kubernetes
  - metrics
  - autoscaling
---

# Metrics Server

> Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.
>
> The metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through [Metrics API](https://github.com/kubernetes/metrics) for use by [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler/). Metrics API can also be accessed by `kubectl top`, making it easier to debug autoscaling pipelines.[^1]

---

> For Kubernetes, the _Metrics API_ offers a basic set of metrics to support automatic scaling and similar use cases. This API provides information about node and pod resource usage, including CPU and memory metrics. Suppose you deploy the [Metrics API](https://github.com/kubernetes/metrics) into your cluster. In that case, clients of the Kubernetes API can then query for this information, and you can use Kubernetes' access control mechanisms to manage permissions to do so.[^2]

## Custom Metrics

- <https://github.com/kubernetes-sigs/custom-metrics-apiserver>

[^1]: <https://github.com/kubernetes-sigs/metrics-server>
[^2]: <https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/>
