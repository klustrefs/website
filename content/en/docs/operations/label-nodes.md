---
title: Label Lustre-capable nodes
description: Apply and verify the topology label used by the Klustre storage class.
weight: 10
---

The default `klustre-csi-static` storage class restricts scheduling to nodes labeled `lustre.csi.klustrefs.io/lustre-client=true`. Use this runbook whenever you add or remove nodes from the Lustre client pool.

## Requirements

- Cluster-admin access with `kubectl`.
- Nodes already have the Lustre client packages installed and can reach your Lustre servers.

## Steps

1. **Identify nodes that can mount Lustre**

   ```bash
   kubectl get nodes -o wide
   ```

   Cross-reference with your infrastructure inventory or automation outputs to find the node names that have Lustre connectivity.

2. **Apply the label**

   ```bash
   kubectl label nodes <node-name> lustre.csi.klustrefs.io/lustre-client=true
   ```

   Repeat for each eligible node. Use `--overwrite` if the label already exists but the value should change.

3. **Verify**

   ```bash
   kubectl get nodes -L lustre.csi.klustrefs.io/lustre-client
   ```

   Ensure only the nodes with Lustre access show `true`. Remove the label from nodes that lose access:

   ```bash
   kubectl label nodes <node-name> lustre.csi.klustrefs.io/lustre-client-
   ```

4. **Confirm DaemonSet placement**

   ```bash
   kubectl get pods -n klustre-system -o wide \
     -l app.kubernetes.io/name=klustre-csi
   ```

   Pods from the `klustre-csi-node` daemonset should exist only on labeled nodes. If you see pods on unlabeled nodes, check the `nodeSelector` and tolerations in the daemonset spec.

## Related topics

- [Installation requirements](/docs/requirements/)
- [Troubleshooting tips](/docs/#troubleshooting-tips)
