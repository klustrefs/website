---
title: Install with kubectl/manifests
description: Apply the published Klustre CSI manifests with kubectl.
weight: 50
---

## 1. Install directly with Kustomize (no clone)

If you just want a default install, you don’t need to clone the repository. You can apply the published manifests directly from GitHub:

```bash
export KLUSTREFS_VERSION=main
kubectl apply -k "github.com/klustrefs/klustre-csi-plugin//manifests?ref=$KLUSTREFS_VERSION"
```

The `manifests/` directory includes the namespace, RBAC, `CSIDriver`, daemonset, node service account, default `StorageClass` (`klustre-csi-static`), and settings config map.

## 2. Work from a local clone (recommended for customization)

If you plan to inspect or customize the manifests, clone the repo and work from a local checkout:

```bash
git clone https://github.com/klustrefs/klustre-csi-plugin.git
cd klustre-csi-plugin
```

You can perform the same default install from the local checkout:

```bash
kubectl apply -k manifests
```

## 3. Customize with a Kustomize overlay (optional)

To change defaults such as `logLevel`, `nodeImage`, or the CSI endpoint path without editing the base files, create a small overlay that patches the settings config map.

Create `overlays/my-cluster/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../manifests

patchesStrategicMerge:
  - configmap-klustre-csi-settings-patch.yaml
```

Create `overlays/my-cluster/configmap-klustre-csi-settings-patch.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: klustre-csi-settings
  namespace: klustre-system
data:
  logLevel: debug
  nodeImage: ghcr.io/klustrefs/klustre-csi-plugin:0.1.1
```

Then apply your overlay instead of the base:

```bash
kubectl apply -k overlays/my-cluster
```

You can add additional patches in the overlay (for example, to tweak the daemonset or StorageClass) as your cluster needs grow.

## 4. Verify rollout

```bash
kubectl get pods -n klustre-system -o wide
kubectl describe daemonset klustre-csi-node -n klustre-system
kubectl logs daemonset/klustre-csi-node -n klustre-system -c klustre-csi
```

After the daemonset is healthy on all Lustre-capable nodes, continue with the [validation steps](/docs/quickstart/#step-2--verify-the-daemonset) or jump to the sample workload.
