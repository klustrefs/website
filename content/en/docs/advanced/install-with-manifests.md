---
title: Install with kubectl/manifests
description: Apply the published Klustre CSI manifests with kubectl.
weight: 50
---

## 1. Clone the repository

```bash
git clone https://github.com/klustrefs/klustre-csi-plugin.git
cd klustre-csi-plugin
```

## 2. Review settings

Inspect `manifests/configmap-klustre-csi-settings.yaml` if you need to override defaults such as `logLevel`, `nodeImage`, or the CSI endpoint path. Adjust the config map before applying.

## 3. Apply manifests

```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/
```

The directory includes the namespace, RBAC, `CSIDriver`, daemonset, node service account, default `StorageClass` (`klustre-csi-static`), and settings config map.

## 4. Verify rollout

```bash
kubectl get pods -n klustre-system -o wide
kubectl describe daemonset klustre-csi-node -n klustre-system
kubectl logs daemonset/klustre-csi-node -n klustre-system -c klustre-csi
```

After the daemonset is healthy on all Lustre-capable nodes, continue with the [validation steps](/docs/quickstart/#step-2--verify-the-daemonset) or jump to the sample workload.
