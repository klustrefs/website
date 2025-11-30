---
title: Upgrade guide
description: Plan Klustre CSI version upgrades alongside Kubernetes changes.
weight: 20
---

## 1. Review release notes

Check the [klustre-csi-plugin GitHub releases](https://github.com/klustrefs/klustre-csi-plugin/releases) for breaking changes, minimum Kubernetes versions, and image tags.

## 2. Update the image reference

- Helm users: bump `image.tag` and `nodePlugin.registrar.image.tag` in your values file, then run `helm upgrade`.
- Manifest users: edit `manifests/configmap-klustre-csi-settings.yaml` (`nodeImage`, `registrarImage`) and reapply the manifests.

See [Update the node daemonset image](/docs/operations/update-daemonset-image/) for detailed steps.

## 3. Roll out sequentially

```bash
kubectl rollout restart daemonset/klustre-csi-node -n klustre-system
kubectl rollout status daemonset/klustre-csi-node -n klustre-system
```

The daemonset restarts one node at a time, keeping existing mounts available.

## 4. Coordinate with Kubernetes upgrades

When upgrading kubelet:

1. Follow the [node maintenance checklist](/docs/operations/maintenance/node-maintenance/) for each node.
2. Upgrade the node OS/kubelet.
3. Verify the daemonset pod recreates successfully before moving to the next node.

## 5. Validate workloads

- Spot-check pods that rely on Lustre PVCs (`kubectl exec` into them and run `df -h /mnt/lustre`).
- Ensure no stale `FailedMount` events exist.

## Rollback

If the new version misbehaves:

1. Revert `nodeImage` and related settings to the previous tag.
2. Run `kubectl rollout restart daemonset/klustre-csi-node -n klustre-system`.
3. Inspect logs to confirm the old version is running.
