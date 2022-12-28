---
tags:
  - Kubernetes
  - extensible-admission-controllers
---

# Network Bandwidth Annotator

## Setup

- Setup a local Kubernetes cluster using [[k3d]]
- Install [[cert-manager]].
- Build and push an OCI image using the following:

  ```bash
  # Build the OCI image
  docker build -t networkbandwidthannotator:0.1.0 "."
  # Tag it to use the local k3d managed registry
  docker tag networkbandwidthannotator:0.1.0 default-registry.localhost:61940/networkbandwidthannotator:0.1.0
  # Push it to the local registry
  docker push default-registry.localhost:61940/networkbandwidthannotator:0.1.0
  ```

  Note: The local registry port will change and will require adjustment.

- Deploy annotator using:

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: nba
  ---
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: selfsigned-issuer
  spec:
    selfSigned: {}
  ---
  apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    name: network-bandwidth-annotator
    namespace: nba
  spec:
    dnsNames:
      - network-bandwidth-annotator.nba.svc
      - network-bandwidth-annotator.nba.svc.cluster.local
    duration: 2160h
    isCA: false
    issuerRef:
      kind: ClusterIssuer
      name: selfsigned-issuer
    privateKey:
      algorithm: RSA
      encoding: PKCS1
      size: 2048
    renewBefore: 360h
    secretName: tls-network-bandwidth-annotator
    subject:
      organizations:
        - Thomas Kosiewski
    usages:
      - server auth
      - client auth
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: network-bandwidth-annotator
    namespace: nba
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: network-bandwidth-annotator
    template:
      metadata:
        labels:
          app: network-bandwidth-annotator
      spec:
        containers:
          - command:
              - ./network-bandwidth-annotator
              - -v
            env:
              - name: ADDR
                value: 0.0.0.0:8443
              - name: TLS_CERT_FILE
                value: /certs/tls.crt
              - name: TLS_KEY_FILE
                value: /certs/tls.key
            image: default-registry:61940/networkbandwidthannotator:0.1.0
            name: network-bandwidth-annotator
            ports:
              - containerPort: 8443
                name: https
            volumeMounts:
              - mountPath: /certs
                name: tls-certs
                readOnly: true
        volumes:
          - name: tls-certs
            secret:
              secretName: tls-network-bandwidth-annotator
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: network-bandwidth-annotator
    namespace: nba
  spec:
    ports:
      - name: https
        port: 8443
    selector:
      app: network-bandwidth-annotator
  ---
  apiVersion: admissionregistration.k8s.io/v1
  kind: MutatingWebhookConfiguration
  metadata:
    annotations:
      cert-manager.io/inject-ca-from: nba/network-bandwidth-annotator
    name: network-bandwidth-annotator
    namespace: nba
  webhooks:
    - admissionReviewVersions:
        - v1
        - v1beta1
      clientConfig:
        service:
          name: network-bandwidth-annotator
          namespace: nba
          path: /mutate
          port: 8443
      failurePolicy: Ignore
      name: network-bandwidth-annotator.nba.svc
      namespaceSelector:
        matchLabels:
          nba-enabled: "true"
      rules:
        - apiGroups:
            - ""
          apiVersions:
            - v1
          operations:
            - CREATE
            - UPDATE
          resources:
            - pods
          scope: Namespaced
      sideEffects: None
      timeoutSeconds: 5
  ```

- Deploy a test pod using the following:

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: nba-test
    labels:
      nba-enabled: "true"
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    namespace: nba-test
    annotations:
      # This is necessary because of the way CNI traffic shaping support
      # currently implements its limits
      kubernetes.io/ingress-bandwidth: 1M
      kubernetes.io/egress-bandwidth: 1M
  spec:
    containers:
    - name: my-container
      image: default-registry:61940/networkbandwidthannotator:0.1.0
      resources:
        requests:
          cpu: 2
          # if the pod contains ingress and egress bandwidth annotations
          # the requests will be automatically set to the annotations values
          ingress-bandwidth: 1M
          egress-bandwidth: 1M
        limits:
          cpu: 4
          # Not used by neither the kube-scheduler, nor the CNI
          ingress-bandwidth: 1M
          egress-bandwidth: 1M
  ```
