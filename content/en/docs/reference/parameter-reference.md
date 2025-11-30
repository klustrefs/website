---
title: Parameter reference
description: CLI flags and environment variables for the Klustre CSI node daemonset.
weight: 40
---

The node daemonset containers accept a small set of flags and environment variables. Most values are sourced from `ConfigMap/klustre-csi-settings`. Use this table as a quick lookup when you need to override behavior.

| Component / flag | Env var | Purpose | Default source |
| ---------------- | ------- | ------- | -------------- |
| `klustre-csi --node-id` | `KUBE_NODE_NAME` | Unique identifier sent to the CSI sidecars and kubelet. Normally the Kubernetes node name. | Downward API (`spec.nodeName`). |
| `klustre-csi --endpoint` | `CSI_ENDPOINT` | Path to the CSI UNIX socket served by the node plugin. Must match kubelet registration path. | `csiEndpoint` in the settings ConfigMap. |
| `klustre-csi --log-level` | `LOG_LEVEL` | Driver verbosity (`error`, `warn`, `info`, `debug`, `trace`). | `logLevel` in the settings ConfigMap. |
| `PATH` | `PATH` | Ensures `mount.lustre`, `umount.lustre`, and related tools are found inside the container. | `/host/usr/sbin:/host/sbin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`. |
| `LD_LIBRARY_PATH` | `LD_LIBRARY_PATH` | Points to host library directories required by the Lustre client binaries. | `/host/lib:/host/lib64:/host/usr/lib:/host/usr/lib64`. |
| `node-driver-registrar --csi-address` | n/a | Location of the CSI socket inside the pod. | `/csi/csi.sock`. |
| `node-driver-registrar --kubelet-registration-path` | n/a (derived from ConfigMap) | Host path where kubelet looks for CSI drivers. | `driverRegistrationArg` from the settings ConfigMap. |

### How to override

1. Edit the settings ConfigMap:

   ```bash
   kubectl -n klustre-system edit configmap klustre-csi-settings
   ```

   Change `csiEndpoint`, `driverRegistrationArg`, or `logLevel` as needed.

2. If you must customize `PATH` or `LD_LIBRARY_PATH`, edit the daemonset directly (Helm users can override `nodePlugin.pathEnv` or `nodePlugin.ldLibraryPath` values).

3. Restart the daemonset pods:

   ```bash
   kubectl rollout restart daemonset/klustre-csi-node -n klustre-system
   ```

### Notes

- `--node-id` should stay aligned with the Kubernetes node name unless you have a strong reason to deviate (CSI treats it as the authoritative identifier).
- Changing `CSI_ENDPOINT` or `driverRegistrationArg` requires matching host path mounts in the daemonset (`pluginDir`, `registrationDir`).
- Increasing `LOG_LEVEL` to `debug` or `trace` is useful for troubleshooting but may emit sensitive information—reset it after collecting logs.
