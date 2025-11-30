---
title: Update the Klustre CSI image
description: Roll out a new plugin container image across the daemonset.
weight: 20
---

Use this guide to bump the Klustre CSI image version (for example, when adopting a new release).

## Requirements

- Cluster-admin access.
- The new image is pushed to a registry reachable by your cluster (GHCR or a mirror).
- The `ghcr-secret` or equivalent image pull secret already contains credentials for the registry.

## Steps

1. **Edit the settings ConfigMap**

   The manifests and Helm chart both reference `ConfigMap/klustre-csi-settings`. Update the `nodeImage` key with the new tag:

   ```bash
   kubectl -n klustre-system edit configmap klustre-csi-settings
   ```

   Example snippet:

   ```yaml
   data:
     nodeImage: ghcr.io/klustrefs/klustre-csi-plugin:0.1.2
   ```

   Save and exit.

2. **Restart the daemonset pods**

   ```bash
   kubectl rollout restart daemonset/klustre-csi-node -n klustre-system
   ```

3. **Watch the rollout**

   ```bash
   kubectl rollout status daemonset/klustre-csi-node -n klustre-system
   kubectl get pods -n klustre-system -o wide
   ```

4. **Verify the running image**

   ```bash
   kubectl get pods -n klustre-system -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[0].image}{"\n"}{end}'
   ```

   Confirm all pods now report the new tag.

5. **Optional: clean up old images**

   If you mirror images, remove unused tags from your registry or automation as needed.

## Related topics

- [Install with manifests](/docs/advanced/install-with-manifests/)
- [Install with Helm](/docs/advanced/install-with-helm/)
