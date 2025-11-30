---
title: Static PV workflow
description: Define PersistentVolumes and PersistentVolumeClaims that reference Lustre exports.
weight: 10
---

## 1. Create the PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lustre-static-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  storageClassName: klustre-csi-static
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: lustre.csi.klustrefs.io
    volumeHandle: lustre-static-pv
    volumeAttributes:
      source: 10.0.0.1@tcp0:/lustre-fs
      mountOptions: flock,user_xattr
```

- `volumeHandle` just needs to be unique within the cluster; it is not used by the Lustre backend.
- `volumeAttributes.source` carries the Lustre management target and filesystem path.

## 2. Bind with a PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lustre-static-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: klustre-csi-static
  volumeName: lustre-static-pv
  resources:
    requests:
      storage: 10Gi
```

Even though Lustre capacity is managed outside Kubernetes, the `storage` field should match the PV so the binder succeeds.

## 3. Mount from workloads

```yaml
volumes:
  - name: lustre
    persistentVolumeClaim:
      claimName: lustre-static-pvc
containers:
  - name: app
    image: busybox
    volumeMounts:
      - name: lustre
        mountPath: /mnt/lustre
```

Multiple pods can reference the same PVC because Lustre supports `ReadWriteMany`. Pods must schedule on labeled nodes (`lustre.csi.klustrefs.io/lustre-client=true`).

## 4. Cleanup

Deleting the PVC detaches pods but the PV remains because the reclaim policy is `Retain`. Manually delete the PV when you no longer need it.
