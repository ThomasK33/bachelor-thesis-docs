---
tags:
  - Kubernetes
  - extensible-admission-controllers
---

# Network Bandwidth Annotation Manager

## Setup

- Setup a local Kubernetes cluster using [[k3d]]
- Install [[cert-manager]].
- Build and push an OCI image using the following:

  ```bash
  # Build the OCI image
  docker build -t networkbandwidthannotator:0.1.0 "."
  # Tag it to use the local k3d managed registry
  docker tag networkbandwidthannotator:0.1.0 k3d-default-registry.localhost:9090/networkbandwidthannotator:0.1.0
  # Push it to the local registry
  docker push k3d-default-registry.localhost:9090/networkbandwidthannotator:0.1.0
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
      nbam-enabled: "true"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    namespace: nbam-test
  spec:
    containers:
    - name: my-container
      image: k3d-default-registry.localhost:9090/networkbandwidthannotator:0.1.0
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
          networking.k8s.io/ingress-bandwidth: 1M
          # Limits the egress bandwidth to 1Mbit/s
          networking.k8s.io/egress-bandwidth: 1M
  ```
