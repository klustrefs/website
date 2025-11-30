---
title: Settings ConfigMap
description: Values consumed by the Klustre CSI node daemonset via ConfigMap.
weight: 10
---

`ConfigMap/klustre-csi-settings` provides runtime configuration to the node daemonset. Each key maps to either an environment variable or a command-line argument.

| Key | Description | Default |
| --- | --- | --- |
| `csiEndpoint` | UNIX socket path used by the node plugin. Must align with kubelet’s plugin directory. | `unix:///var/lib/kubelet/plugins/lustre.csi.klustrefs.io/csi.sock` |
| `driverRegistrationArg` | Argument passed to the node-driver-registrar sidecar. | `--kubelet-registration-path=/var/lib/kubelet/plugins/lustre.csi.klustrefs.io/csi.sock` |
| `logLevel` | Verbosity for the Klustre CSI binary (`info`, `debug`, etc.). | `info` |
| `nodeImage` | Container image for the Klustre CSI node plugin. | `ghcr.io/klustrefs/klustre-csi-plugin:0.0.1` |
| `pluginDir` | HostPath where CSI sockets live. | `/var/lib/kubelet/plugins/lustre.csi.klustrefs.io` |
| `priorityClassName` | Priority class applied to the daemonset pods. | `system-node-critical` |
| `registrarImage` | Container image for the node-driver-registrar sidecar. | `registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.10.1` |
| `registrationDir` | HostPath where kubelet expects CSI driver registration files. | `/var/lib/kubelet/plugins_registry` |

To update any field:

```bash
kubectl -n klustre-system edit configmap klustre-csi-settings
kubectl rollout restart daemonset/klustre-csi-node -n klustre-system
```

Ensure any customized paths (e.g., `pluginDir`) match the volumes mounted in the daemonset spec.
