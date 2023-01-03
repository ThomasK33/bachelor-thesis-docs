---
tags:
  - Kubernetes
  - scheduler
  - kube-scheduler
---

# Scheduling Using Extended Resources

> Extended resources are fully-qualified resource names outside the `kubernetes.io` domain. They allow cluster operators to advertise and users to consume the non-Kubernetes-built-in resources.
> There are two steps required to use Extended Resources. First, the cluster operator must advertise an Extended Resource. Second, users must request the Extended Resource in Pods.[^1]

[^1]: <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#extended-resources>

One can add networking-related information to a node's ingress and egress capacities and allocatable amounts by utilizing Extended Resources.

Those can either be added manually, by a controller or using a device plugin.[^6]

## Shortcomings

> **Note:** Extended resources cannot be overcommitted, so request and limit must be equal if both are present in a container spec.[^5]

## Adding Network Related Resources To Nodes

Create a new shell session, then proxy the api-server to a local port, with authentication already in place:

```bash
kubectl proxy
```

### Capacity Information

Set a node's status capacity information by using, e.g.:

```bash
curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1ingress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1egress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status
```

(Note: `1.25e+9` is equivalent to `1.25Gbps`.)

Applied to a local as outlined in [[k3d#Cluster Creation]]:

```bash
curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1ingress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1egress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1ingress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-1/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1egress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-1/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1ingress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-2/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/networking.k8s.io~1egress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-2/status
```

### Allocatable Amount Information

Furthermore, if needed, one could also set a lower allocatable amount:

```bash
curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/allocatable/networking.k8s.io~1ingress-bandwidth", "value": "1e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/allocatable/networking.k8s.io~1egress-bandwidth", "value": "1e+9"}]' \
  http://localhost:8001/api/v1/nodes/k3d-default-server-0/status
```

## Using Resources In Pod Definition

For traffic shaping[^2], Pods require an annotation for ingress[^3] and egress[^4] bandwidth limits and do not utilize the resource specs.
The [[Kubernetes Scheduler|kube-scheduler]] only uses the resource request specification for scheduling.
Therefore, it ignores the annotations, while the [[CNI Bandwidth Limiting]] ignores the specified resource requests and only utilizes the annotations for limiting.

In that sense, setting container limits in `spec.containers[*].resources.limits.ingress-bandwidth` or `spec.containers[*].resources.limits.egress-bandwidth` is pointless unless a [[Mutating Webhooks|mutating webhook]] rewrites the pod definition and sets the required pod's bandwidth annotations based on the limits if no bandwidth related annotations are present.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  annotations:
    # This is necessary because of the way CNI traffic shaping support
    # currently implements its limits
    kubernetes.io/ingress-bandwidth: 1M
    kubernetes.io/egress-bandwidth: 1M
spec:
  containers:
  - name: my-container
    image: myimage
    resources:
      requests:
        cpu: 2
        # if the pod contains ingress and egress bandwidth annotations
        # the requests will be automatically set to the annotations values
        networking.k8s.io/ingress-bandwidth: 1M
        networking.k8s.io/egress-bandwidth: 1M
      limits:
        cpu: 4
        # Not used by neither the kube-scheduler, nor the CNI
        networking.k8s.io/ingress-bandwidth: 1M
        networking.k8s.io/egress-bandwidth: 1M
```

[^2]: <https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#support-traffic-shaping>
[^3]: <https://kubernetes.io/docs/reference/labels-annotations-taints/#kubernetes-io-ingress-bandwidth>
[^4]: <https://kubernetes.io/docs/reference/labels-annotations-taints/#kubernetes-io-egress-bandwidth>
[^5]: <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#consuming-extended-resources>
[^6]: <https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/>
