---
title: Requirements
description: Detailed requirements for running the Klustre CSI Plugin.
weight: 2
---

Use this page when you need the full checklist (versions, node prep, and registry access) before installing Klustre CSI. The Quickstart and Introduction pages summarize this information, but this serves as a canonical reference.

## Kubernetes cluster

- Kubernetes v1.20 or newer with CSI v1.5 enabled.
- Control plane and kubelets must allow privileged pods (`hostPID`, `hostNetwork`, `SYS_ADMIN` capability).
- Cluster-admin `kubectl` access.

## Lustre-capable worker nodes

- Install the Lustre client packages (`mount.lustre`, kernel modules, user-space tools) on every node that will host Lustre-backed workloads.
- Ensure network connectivity (TCP, RDMA, etc.) from those nodes to your Lustre servers.
- Label the nodes:

  ```bash
  kubectl label nodes <node-name> lustre.csi.klustrefs.io/lustre-client=true
  ```

## Namespace, security, and registry access

- Create the namespace and optional GHCR image pull secret:

  ```bash
  kubectl create namespace klustre-system
  kubectl create secret docker-registry ghcr-secret \
    --namespace klustre-system \
    --docker-server=ghcr.io \
    --docker-username=<github-user> \
    --docker-password=<github-token>
  ```

- Label the namespace for Pod Security Admission if your cluster enforces it:

  ```bash
  kubectl label namespace klustre-system \
    pod-security.kubernetes.io/enforce=privileged \
    pod-security.kubernetes.io/audit=privileged \
    pod-security.kubernetes.io/warn=privileged
  ```

## Tooling

- `kubectl`, `git`, and optionally `helm`.
- Hugo/npm/Go are only needed if you plan to contribute to this documentation site.
