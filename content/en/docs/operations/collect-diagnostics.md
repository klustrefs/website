---
title: Collect diagnostics
description: Gather logs and cluster state for troubleshooting or support requests.
weight: 30
---

When reporting an issue, provide the following artifacts so maintainers can reproduce the problem.

## 1. Capture pod logs

```bash
kubectl logs -n klustre-system daemonset/klustre-csi-node -c klustre-csi --tail=200 > klustre-csi.log
kubectl logs -n klustre-system daemonset/klustre-csi-node -c node-driver-registrar --tail=200 > node-driver-registrar.log
```

If a specific pod is failing, target it directly:

```bash
kubectl logs -n klustre-system <pod-name> -c klustre-csi --previous
```

## 2. Describe pods and daemonset

```bash
kubectl describe daemonset klustre-csi-node -n klustre-system > klustre-csi-daemonset.txt
kubectl describe pods -n klustre-system -l app.kubernetes.io/name=klustre-csi > klustre-csi-pods.txt
```

## 3. Export relevant resources

```bash
kubectl get csidriver lustre.csi.klustrefs.io -o yaml > csidriver.yaml
kubectl get storageclass klustre-csi-static -o yaml > storageclass.yaml
kubectl get configmap klustre-csi-settings -n klustre-system -o yaml > configmap.yaml
```

Remove sensitive data (e.g., registry credentials) before sharing.

## 4. Include node information

- Output of `uname -a`, `lsmod | grep lustre`, and the Lustre client version on affected nodes.
- Whether the node can reach your Lustre servers (share ping or `mount.lustre` command output if available).

## 5. Bundle and share

Package the files into an archive and attach it to your GitHub issue or support request:

```bash
tar czf klustre-diagnostics.tgz klustre-csi.log node-driver-registrar.log \
  klustre-csi-daemonset.txt klustre-csi-pods.txt csidriver.yaml storageclass.yaml configmap.yaml
```

## Related topics

- [Troubleshooting tips](/docs/#troubleshooting-tips)
- [Community support](/docs/)
