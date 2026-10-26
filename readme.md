# Kubernetes Category Test

## Overview

This project demonstrates how Kubernetes and `kubectl` handle CustomResourceDefinition (CRD) **categories**, particularly when category names collide with:
- Standard Kubernetes resource names or shortnames (such as `service`, `services`, or `svc`).
- Other CRD resource names (such as `podnetwork` and `podnetworks`).

It illustrates how `kubectl get <category>` resolves queries when categories match existing resource names or shortnames, resulting in custom resources being returned alongside or in place of expected types.

## What It Deploys

1. **Custom Resource Definitions (`./deployment`)**:
   - **`PodNetwork`** (`podnetworks.multinetwork.networking.x-k8s.io`): A cluster-scoped CRD defined without custom categories to serve as a baseline resource type.
   - **`ClusterNetwork`** (`clusternetworks.multinetwork.networking.x-k8s.io`): A cluster-scoped CRD specifying categories: `podnetwork`, `podnetworks`, `service`, `services`, `svc`, and `all`.
   - **`MyNetwork`** (`mynetworks.multinetwork.networking.x-k8s.io`): A namespaced CRD specifying the same categories: `podnetwork`, `podnetworks`, `service`, `services`, `svc`, and `all`.

2. **Sample Test Resources**:
   - Instances of `ClusterNetwork`, `MyNetwork`, and `PodNetwork`.
   - A standard Kubernetes `Service` to demonstrate how `kubectl get service` and `kubectl get svc` resolve when CRD categories collide with core resource names.

## Demo

Deploy the CRDs:
```yaml
# Locally
kubectl apply -f ./deployment

# Remotely
kubectl apply -f https://raw.githubusercontent.com/LionelJouin/k8s-category-test/refs/heads/main/deployment/multinetwork.networking.x-k8s.io_clusternetworks.yaml
kubectl apply -f https://raw.githubusercontent.com/LionelJouin/k8s-category-test/refs/heads/main/deployment/multinetwork.networking.x-k8s.io_mynetworks.yaml
kubectl apply -f https://raw.githubusercontent.com/LionelJouin/k8s-category-test/refs/heads/main/deployment/multinetwork.networking.x-k8s.io_podnetworks.yaml
```

Deploy `MyNetwork` and `ClusterNetwork` resources:
```yaml
kubectl apply -f - <<EOF
---
apiVersion: multinetwork.networking.x-k8s.io/v1alpha1
kind: MyNetwork
metadata:
  name: test
---
apiVersion: multinetwork.networking.x-k8s.io/v1alpha1
kind: ClusterNetwork
metadata:
  name: test
EOF
```

Deploy a `PodNetwork` resource:
```yaml
kubectl apply -f - <<EOF
---
apiVersion: multinetwork.networking.x-k8s.io/v1alpha1
kind: PodNetwork
metadata:
  name: test
EOF
```

Deploy a `Service` resource:
```yaml
kubectl apply -f - <<EOF
---
apiVersion: v1
kind: Service
metadata:
  name: service-a
spec:
  clusterIP: None
  selector:
    app: example
EOF
```

Get the resources with kubectl:
```sh
# try with PodNetwork:
$ kubectl get podnetwork
NAME                                                   AGE
clusternetwork.multinetwork.networking.x-k8s.io/test   7m30s

NAME                                              AGE
mynetwork.multinetwork.networking.x-k8s.io/test   7m30s

# Try with Service:
$ kubectl get service
NAME                                                   AGE
clusternetwork.multinetwork.networking.x-k8s.io/test   7m14s

NAME                                              AGE
mynetwork.multinetwork.networking.x-k8s.io/test   7m14s

# Try with Service shortname:
$ kubectl get svc
NAME                                                   AGE
clusternetwork.multinetwork.networking.x-k8s.io/test   7m27s

NAME                                              AGE
mynetwork.multinetwork.networking.x-k8s.io/test   7m27s

# Get the services in the cluster (note: the `.` at the end of the command):
$ kubectl get service.
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10m
service-a    ClusterIP   None         <none>        <none>    8m13s

# Get the PodNetwork resources:
$ kubectl get podnetwork..multinetwork.networking.x-k8s.io
NAME   AGE
test   9m32s
```

This has been tested with:
```sh
$ kubectl version
Client Version: v1.37.0
Kustomize Version: v5.8.1
Server Version: v1.37.0
```