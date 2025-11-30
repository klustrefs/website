---
title: Node maintenance checklist
description: Drain nodes safely and ensure Klustre CSI pods return to service.
weight: 10
---

## 1. Cordon and drain

```bash
kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```

Because the Klustre CSI daemonset is a DaemonSet, it is unaffected by `--ignore-daemonsets`, but draining ensures your workloads move off the node before reboot.

## 2. Verify daemonset status

```bash
kubectl get pods -n klustre-system -o wide | grep <node>
```

Expect the daemonset pod to terminate when the node drains and recreate once the node returns.

## 3. Patch or reboot the node

- Apply OS updates, reboot, or swap hardware as needed.
- Ensure the Lustre client packages remain installed (validate with `mount.lustre --version`).

## 4. Uncordon and relabel if necessary

```bash
kubectl uncordon <node>
```

If the node lost the `lustre.csi.klustrefs.io/lustre-client=true` label, reapply it after verifying Lustre connectivity.

## 5. Watch for daemonset rollout

```bash
kubectl rollout status daemonset/klustre-csi-node -n klustre-system
```

## 6. Confirm workloads recover

Use `kubectl get pods` for namespaces that rely on Lustre PVCs to ensure pods are running and mounts succeeded.

## Tips

- For large clusters, drain one Lustre node at a time to keep mounts available.
- If `kubectl drain` hangs due to pods using Lustre PVCs, identify them with `kubectl get pods --all-namespaces -o wide | grep <node>` and evict manually.
