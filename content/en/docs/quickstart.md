---
title: Quickstart
description: Opinionated steps to install Klustre CSI and mount a Lustre share in under 10 minutes.
weight: 3
---

Looking for the fastest path from zero to a mounted Lustre volume? Follow this TL;DR workflow, then explore the detailed installation pages if you need customization.

## Requirements

Before you sprint through the commands below, complete the [requirements checklist](/docs/requirements/). You’ll need:

- `kubectl`, `git`, and optionally `helm`.
- Worker nodes with the Lustre client installed and reachable MGS/MDS/OSS endpoints.
- Nodes labeled `lustre.csi.klustrefs.io/lustre-client=true`.
- The `klustre-system` namespace plus optional GHCR image pull secret (see the requirements page for the canonical commands).

## Step 1 — Install Klustre CSI

```bash
export KLUSTREFS_VERSION=main
kubectl apply -k "github.com/klustrefs/klustre-csi-plugin//manifests?ref=$KLUSTREFS_VERSION"
```

## Step 2 — Verify the daemonset

Wait for the DaemonSet rollout to complete:

```bash
kubectl rollout status daemonset/klustre-csi-node -n klustre-system --timeout=120s
```

Then wait for all node pods to report `Ready`:

```bash
kubectl wait --for=condition=Ready pod -l app=klustre-csi-node -n klustre-system --timeout=120s
```

Pods should schedule only on nodes labeled `lustre.csi.klustrefs.io/lustre-client=true`.

## Step 3 — Mount a Lustre share

Save the following manifest as `lustre-demo.yaml`, updating `volumeAttributes.source` to your Lustre target:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lustre-demo-pv
spec:
  storageClassName: klustre-csi-static
  capacity:
    storage: 1Ti
  accessModes:
    - ReadWriteMany
  csi:
    driver: lustre.csi.klustrefs.io
    volumeHandle: lustre-demo
    volumeAttributes:
      source: 10.0.0.1@tcp0:/lustre-fs # TODO: replace with your Lustre target
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lustre-demo-pvc
spec:
  storageClassName: klustre-csi-static
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Ti
  volumeName: lustre-demo-pv
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lustre-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lustre-demo
  template:
    metadata:
      labels:
        app: lustre-demo
    spec:
      containers:
        - name: demo
          image: busybox:1.36
          command: ["sh", "-c", "sleep 3600"]
          volumeMounts:
            - mountPath: /mnt/lustre
              name: lustre
      volumes:
        - name: lustre
          persistentVolumeClaim:
            claimName: lustre-demo-pvc
```

Apply the demo manifest:

```bash
kubectl apply -f lustre-demo.yaml
```

Confirm the Lustre mount is visible in the pod:

```bash
kubectl exec deploy/lustre-demo -- df -h /mnt/lustre
```

Write a test file into the mounted Lustre share:

```bash
kubectl exec deploy/lustre-demo -- sh -c 'date > /mnt/lustre/hello.txt'
```

## Step 4 — Clean up (optional)

Remove the demo PV, PVC, and Deployment:

```bash
kubectl delete -f lustre-demo.yaml
```

If you only installed Klustre CSI for this quickstart and want to remove it as well, uninstall the driver:

```bash
kubectl delete -k "github.com/klustrefs/klustre-csi-plugin//manifests?ref=$KLUSTREFS_VERSION"
```

## What’s next?

- Need provider-specific instructions or more realistic workloads? Check the [advanced notes](/docs/advanced/) for Kind labs, bare metal clusters, or Amazon EKS.
- Want to understand every knob and see production-ready volume patterns? Dive into the [Advanced installation](/docs/advanced/) section and the [Operations → Nodes & Volumes](/docs/operations/nodes-and-volumes/) pages, including the [Static PV workflow](/docs/operations/nodes-and-volumes/volumes/static-pv-workflow/).
