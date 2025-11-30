---
---
title: Concepts
description: Understand the Klustre CSI architecture, components, and data flow.
weight: 2
---

{{% pageinfo %}}
This page explains how the Klustre CSI Plugin is structured so you can reason about what happens when volumes are created, mounted, or removed.
{{% /pageinfo %}}

## High-level architecture

Klustre CSI Plugin implements the [CSI node service](https://kubernetes-csi.github.io/docs/) for Lustre file systems. It focuses on node-side operations only:

```
Kubernetes API           Kubelet on each node
--------------           --------------------
PersistentVolume   ->    NodePublishVolume -> mount.lustre -> Pod mount
PersistentVolumeClaim    NodeUnpublishVolume -> umount.lustre
```

There is no controller deployment or dynamic provisioning. Instead, administrators define static `PersistentVolume` objects that reference existing Lustre exports. The plugin mounts those exports inside pods when a `PersistentVolumeClaim` is bound to a workload.

## Components

- **DaemonSet (`klustre-csi-node`)** – Runs on every node that has the Lustre client installed. It includes:
  - `klustre-csi` container: the Rust-based CSI node driver.
  - `node-driver-registrar` sidecar: registers the driver socket with kubelet and handles node-level CSIDriver metadata.
- **Settings ConfigMap** – Injects runtime configuration (socket paths, image tags, log level).
- **StorageClass (`klustre-csi-static`)** – Encapsulates mount options, retain policy, and topology constraints.
- **CSIDriver object** – Advertises the driver name `lustre.csi.klustrefs.io`, declares that controller operations aren’t required, and enables `podInfoOnMount`.

## Data flow

1. **Cluster admin creates a PV** that sets `csi.driver=lustre.csi.klustrefs.io` and `volumeAttributes.source=<HOST>@tcp:/fs`.
2. **PVC binds to the PV**, usually via the `klustre-csi-static` storage class.
3. **Pod scheduled on labeled node** (`lustre.csi.klustrefs.io/lustre-client=true`).
4. **kubelet invokes NodePublishVolume**:
   - CSI driver reads the `source` attribute.
   - It spawns `mount.lustre` inside the container with the host’s `/sbin` and `/usr/sbin` bind-mounted.
   - The Lustre client mounts into the pod’s `volumeMount`.
5. **Pod uses the mounted path** (read/write simultaneously across multiple pods).
6. **When the pod terminates** and the PVC is detached, kubelet calls `NodeUnpublishVolume`, which triggers `umount.lustre`.

## Labels and topology

The driver relies on a node label to ensure only valid hosts run Lustre workloads:

```
lustre.csi.klustrefs.io/lustre-client=true
```

`allowedTopologies` in the storage class references this label so the scheduler only considers labeled nodes. Make sure to label/clear nodes as you add or remove Lustre support (see [Label Lustre-capable nodes](/docs/operations/label-nodes/)).

## Security and privileges

- Pods run `privileged` with `SYS_ADMIN` capability to execute `mount.lustre`.
- Host namespaces: `hostNetwork: true` and `hostPID: true` so the driver can interact with kubelet paths and kernel modules.
- HostPath mounts expose `/var/lib/kubelet`, `/dev`, `/sbin`, `/usr/sbin`, `/lib`, and `/lib64`. Verify your cluster policy (PSA/PSP) allows this in the `klustre-system` namespace.

## Limitations (by design)

- ❌ No dynamic provisioning (no `CreateVolume`/`DeleteVolume`): you must create static PVs.
- ❌ No CSI snapshots or expansion.
- ❌ No node metrics endpoints.
- ✅ Designed for `ReadWriteMany` workloads with preexisting Lustre infrastructure.

Understanding these guardrails helps you decide whether Klustre CSI fits your cluster and what automation you may need to layer on top (for example, scripts that generate PVs from a Lustre inventory). Now that you know the architecture, head back to the [Introduction](/docs/) or browse the [Reference](/docs/reference/) for tunable parameters.
