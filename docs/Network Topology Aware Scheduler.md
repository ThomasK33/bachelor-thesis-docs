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

## Adding Extended Resources To Nodes

Create a new shell session, then proxy the api-server to a local port, with authentication already in place:

```bash
kubectl proxy
```

Then set a nodes status capacity information, by e.g.:

```bash
curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/ingress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/docker-desktop/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/capacity/egress-bandwidth", "value": "1.25e+9"}]' \
  http://localhost:8001/api/v1/nodes/docker-desktop/status
```

(Note: `1.25e+9` is equivalent to `1.25Gbps`.)

If needed, one could also set a lower allocatable amount:

```bash
curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/allocatable/ingress-bandwidth", "value": "1e+9"}]' \
  http://localhost:8001/api/v1/nodes/docker-desktop/status

curl --header "Content-Type: application/json-patch+json" \
  --request PATCH \
  --data '[{"op": "add", "path": "/status/allocatable/egress-bandwidth", "value": "1e+9"}]' \
  http://localhost:8001/api/v1/nodes/docker-desktop/status
```

## Pod Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  annotations:
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
        ingress-bandwidth: 1M
        egress-bandwidth: 1M
      limits:
        cpu: 4
        # if the pod contains ingress and egress bandwidth annotations
        # the limits will be automatically set to the annotations values
        ingress-bandwidth: 1M
        egress-bandwidth: 1M
```
