---
title: Storage class parameters
description: Details of the default static storage class shipped with Klustre CSI.
weight: 20
---

The manifests bundle a `StorageClass` named `klustre-csi-static`. It targets pre-provisioned Lustre exports and enforces node placement.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: klustre-csi-static
allowedTopologies:
- matchLabelExpressions:
  - key: lustre.csi.klustrefs.io/lustre-client
    values:
    - "true"
mountOptions:
- flock
- user_xattr
provisioner: lustre.csi.klustrefs.io
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

### Field summary

- **`provisioner`** – Must stay `lustre.csi.klustrefs.io` so PVCs bind to the Klustre driver.
- **`allowedTopologies`** – Uses the `lustre.csi.klustrefs.io/lustre-client=true` label to ensure only Lustre-capable nodes run workloads. Update the label key/value if you customize node labels.
- **`mountOptions`** – Defaults to `flock` and `user_xattr`. Add or remove Lustre options as needed (e.g., `nolock`, `noatime`).
- **`reclaimPolicy`** – `Retain` keeps the PV around when a PVC is deleted, which is typical for statically provisioned Lustre shares.
- **`volumeBindingMode`** – `WaitForFirstConsumer` defers binding until a pod is scheduled, ensuring topology constraints match the consuming workload.

### Customization tips

- Create additional storage classes for different mount flag sets or topology labels. Ensure each class references the same provisioner.
- If you disable topology constraints, remove `allowedTopologies`, but be aware that pods might schedule onto nodes without Lustre access.
- For multi-cluster environments, consider namespacing storage class names (e.g., `klustre-csi-static-prod`).
