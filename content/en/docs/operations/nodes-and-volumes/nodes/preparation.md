---
title: Node preparation
description: Install the Lustre client, label nodes, and grant the privileges required by Klustre CSI.
weight: 10
---

## Install the Lustre client stack

Every node that runs Lustre-backed pods must have:

- `mount.lustre` and `umount.lustre` binaries (via `lustre-client` RPM/DEB).
- Kernel modules compatible with your Lustre servers.
- Network reachability to the Lustre MGS/MDS/OSS endpoints.

Verify installation:

```bash
mount.lustre --version
lsmod | grep lustre
```

## Label nodes

The default storage class and daemonset use the label `lustre.csi.klustrefs.io/lustre-client=true`.

```bash
kubectl label nodes <node-name> lustre.csi.klustrefs.io/lustre-client=true
```

Remove the label when a node no longer has Lustre access:

```bash
kubectl label nodes <node-name> lustre.csi.klustrefs.io/lustre-client-
```

## Allow privileged workloads

Klustre CSI pods require:

- `privileged: true`, `allowPrivilegeEscalation: true`
- `hostPID: true`, `hostNetwork: true`
- HostPath mounts for `/var/lib/kubelet`, `/dev`, `/sbin`, `/usr/sbin`, `/lib`, and `/lib64`

Label the namespace with Pod Security Admission overrides:

```bash
kubectl create namespace klustre-system
kubectl label namespace klustre-system \
  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/audit=privileged \
  pod-security.kubernetes.io/warn=privileged
```

## Maintain consistency

- Keep AMIs or OS images in sync so every node has the same Lustre client version.
- If you use autoscaling groups, bake the client packages into your node image or run a bootstrap script before kubelet starts.
- Automate label management with infrastructure-as-code (e.g., Cluster API, Ansible) so the right nodes receive the `lustre-client=true` label on join/leave events.
