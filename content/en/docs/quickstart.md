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
git clone https://github.com/klustrefs/klustre-csi-plugin.git
cd klustre-csi-plugin
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/
```

## Step 2 — Verify the daemonset

```bash
kubectl get pods -n klustre-system -o wide
kubectl describe daemonset klustre-csi-node -n klustre-system
```

Pods should schedule only on nodes labeled `lustre.csi.klustrefs.io/lustre-client=true`.

## Step 3 — Mount a Lustre share

1. Copy the static PV/PVC demo from [Operations → Volumes → Static PV workflow](/docs/operations/nodes-and-volumes/volumes/static-pv-workflow/). Update `volumeAttributes.source` to your Lustre target.
2. `kubectl apply -f lustre-demo.yaml`
3. `kubectl exec deploy/lustre-demo -- df -h /mnt/lustre`
4. `kubectl exec deploy/lustre-demo -- sh -c 'date > /mnt/lustre/hello.txt'`

## Step 4 — Clean up (optional)

```bash
kubectl delete -f lustre-demo.yaml
```

## What’s next?

- Need provider-specific instructions? Check the [advanced notes](/docs/advanced/) for Kind labs, bare metal clusters, or Amazon EKS.
- Want to understand every knob? Dive into the [Advanced installation](/docs/advanced/) section and the [Operations → Nodes & Volumes](/docs/operations/nodes-and-volumes/) pages.
