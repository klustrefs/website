---
title: Node integration flow
description: Understand how Klustre CSI interacts with kubelet and the host filesystem.
weight: 20
---

## Daemonset host mounts

`DaemonSet/klustre-csi-node` mounts the following host paths:

- `/var/lib/kubelet/plugins` and `/var/lib/kubelet/pods` – required for CSI socket registration and mount propagation.
- `/dev` – ensures device files (if any) are accessible when mounting Lustre.
- `/sbin`, `/usr/sbin`, `/lib`, `/lib64` – expose the host’s Lustre client binaries and libraries to the container.

If your kubelet uses custom directories, update `pluginDir` and `registrationDir` in the [settings ConfigMap](/docs/reference/settings-configmap/).

## CSI socket lifecycle

1. The node plugin listens on `csiEndpoint` (defaults to `/var/lib/kubelet/plugins/lustre.csi.klustrefs.io/csi.sock`).
2. The node-driver-registrar sidecar registers that socket with kubelet via `registrationDir`.
3. Kubelet uses the UNIX socket to call `NodePublishVolume` and `NodeUnpublishVolume` when pods mount or unmount PVCs.

If the daemonset does not come up or kubelet cannot reach the socket, run:

```bash
kubectl describe daemonset klustre-csi-node -n klustre-system
kubectl logs -n klustre-system daemonset/klustre-csi-node -c klustre-csi
```

## PATH and library overrides

The containers inherit PATH and LD_LIBRARY_PATH values that point at the host bind mounts. If your Lustre client lives elsewhere, override:

- `nodePlugin.pathEnv`
- `nodePlugin.ldLibraryPath`

via Helm values or by editing the daemonset manifest.

## Health signals

- Kubernetes events referencing `lustre.csi.klustrefs.io` indicate mount/unmount activity.
- `kubectl get pods -n klustre-system -o wide` should show one pod per labeled node.
- A missing pod usually means the node label is absent or taints/tolerations are mismatched.
