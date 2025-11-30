---
title: Helm values
description: Commonly overridden values for the Klustre CSI Helm chart.
weight: 30
---

The chart is published as `oci://ghcr.io/klustrefs/charts/klustre-csi-plugin`. Run `helm show values oci://ghcr.io/klustrefs/charts/klustre-csi-plugin --version 0.1.1` for the full schema. This page summarizes frequently tuned fields.

| Path | Default | Purpose |
| ---- | ------- | ------- |
| `image.repository` | `ghcr.io/klustrefs/klustre-csi-plugin` | Node plugin image repository. |
| `image.tag` | `0.1.1` | Node plugin tag. |
| `imagePullSecrets` | `[]` | Global image pull secrets applied to all pods. |
| `nodePlugin.pluginDir` | `/var/lib/kubelet/plugins/lustre.csi.klustrefs.io` | Host path for CSI sockets. |
| `nodePlugin.kubeletRegistrationPath` | `/var/lib/kubelet/plugins/lustre.csi.klustrefs.io/csi.sock` | Path passed to kubelet registrar. |
| `nodePlugin.logLevel` | `info` | Verbosity for the node binary. |
| `nodePlugin.resources` | `requests: 50m/50Mi`, `limits: 200m/200Mi` | Container resource settings. |
| `nodePlugin.registrar.image.repository` | `registry.k8s.io/sig-storage/csi-node-driver-registrar` | Sidecar repository. |
| `nodePlugin.registrar.image.tag` | `v2.10.1` | Sidecar tag. |
| `nodePlugin.extraVolumes` / `extraVolumeMounts` | `[]` | Inject custom host paths (e.g., additional libraries). |
| `storageClass.create` | `true` | Toggle creation of `klustre-csi-static`. |
| `storageClass.allowedTopologies[0].matchLabelExpressions[0].key` | `lustre.csi.klustrefs.io/lustre-client` | Node label key for placement. |
| `storageClass.mountOptions` | `["flock","user_xattr"]` | Default Lustre mount flags. |
| `settingsConfigMap.create` | `true` | Controls whether the chart provisions `klustre-csi-settings`. |
| `serviceAccount.create` | `true` | Create the node service account automatically. |
| `rbac.create` | `true` | Provision RBAC resources (ClusterRole/Binding). |

### Example override file

```yaml
image:
  tag: 0.1.2
nodePlugin:
  logLevel: debug
  extraVolumeMounts:
    - name: host-etc
      mountPath: /host/etc
  extraVolumes:
    - name: host-etc
      hostPath:
        path: /etc
storageClass:
  mountOptions:
    - flock
    - user_xattr
    - noatime
```

Install with:

```bash
helm upgrade --install klustre-csi \
  oci://ghcr.io/klustrefs/charts/klustre-csi-plugin \
  --version 0.1.1 \
  --namespace klustre-system \
  --create-namespace \
  -f overrides.yaml
```
