---
title: Overview
description: Klustre CSI Plugin brings Lustre’s high-throughput storage into Kubernetes clusters.
weight: 1
---

Klustre CSI Plugin is an open-source [Container Storage Interface (CSI)](https://kubernetes-csi.github.io/docs/) node driver that lets Kubernetes workloads mount existing Lustre file systems. The project focuses on high-performance computing (HPC), AI/ML training, and media workloads that need shared `ReadWriteMany` semantics with the bandwidth of Lustre.

## What does it provide?

- **Kubernetes-native storage** – Exposes Lustre exports via CSI objects such as `CSIDriver`, `PersistentVolume`, and `PersistentVolumeClaim`.
- **Node daemonset** – Runs a privileged pod on every Lustre-capable worker node to perform mounts and unmounts using the host’s Lustre client.
- **Static provisioning** – Administrators define PersistentVolumes that point at existing Lustre paths (for example `10.0.0.1@tcp0:/lustre-fs`) and bind them to workloads.
- **Helm and raw manifests** – Install using the published manifests in `manifests/` or the OCI Helm chart `oci://ghcr.io/klustrefs/charts/klustre-csi-plugin`.
- **Cluster policy alignment** – Default RBAC, topology labels, and resource requests are tuned so scheduling is constrained to nodes that actually have the Lustre client installed.

## Why would I use it?

Use the Klustre CSI plugin when you:

- Operate Lustre today (on-prem or cloud) and want to reuse those file systems inside Kubernetes without rearchitecting your storage layer.
- Need shared volume access (`ReadWriteMany`) with low latency and high throughput, such as MPI workloads, model training jobs, or render farms.
- Prefer Kubernetes-native declarative workflows for provisioning, auditing, and cleaning up Lustre mounts.
- Want a lightweight component that focuses on node-side responsibilities without managing Lustre servers themselves.

## Current scope and limitations

The project intentionally ships a minimal surface area:

- ✅ Mounts and unmounts existing Lustre exports on demand.
- ✅ Supports `ReadWriteMany` access modes and Lustre mount options (`flock`, `user_xattr`, etc.).
- ⚠️ Does **not** implement controller-side operations such as `CreateVolume`, snapshots, expansion, or metrics, so dynamic provisioning and quotas remain outside the plugin.
- ⚠️ Assumes the Lustre client stack is pre-installed on each worker node—images do not bundle client packages.
- ⚠️ Requires privileged pods with `SYS_ADMIN`, `hostPID`, and `hostNetwork`. Clusters that forbid privileged workloads cannot run the driver.

These boundaries keep the plugin predictable while the community converges on the right APIs. Contributions that expand support (e.g., NodeGetVolumeStats) are welcomed through GitHub issues and pull requests.

## Architecture at a glance

1. **DaemonSet**: `klustre-csi-node` schedules one pod per eligible node. It mounts `/sbin`, `/usr/sbin`, `/lib`, `/lib64`, `/dev`, and kubelet directories from the host so the Lustre kernel modules and socket paths remain consistent.
2. **Node Driver Registrar**: A sidecar container handles kubelet registration and lifecycle hooks by connecting to the CSI UNIX socket.
3. **ConfigMap-driven settings**: Runtime configuration such as log level, image tags, and kubelet plugin directory live in `ConfigMap klustre-csi-settings`, making updates declarative.
4. **StorageClass controls placement**: The default `klustre-csi-static` storage class enforces the `lustre.csi.klustrefs.io/lustre-client=true` node label to avoid scheduling onto nodes without Lustre support.

## Where should I go next?

- [Introduction](/docs/): Install the plugin and mount your first Lustre share.
- [GitHub Repository](https://github.com/klustrefs/klustre-csi-plugin): Review source, open issues, or clone manifests.
- [Helm Charts](https://klustrefs.io/charts/): Browse the Helm catalog and values for the CSI plugin chart.
- [Community Discussion](https://github.com/klustrefs/klustre-csi-plugin/discussions): Ask questions, propose features, or share deployment feedback.
