---
title: Static Lustre volume demo
description: Provision a Lustre-backed scratch space, populate it, and consume it from a training deployment.
weight: 10
---


{{% pageinfo %}}
Use these snippets as starting points for demos, CI smoke tests, or reproduction cases when you report issues.
{{% /pageinfo %}}

## Static Lustre volume demo

Creates a static PV/PVC pointing at `10.0.0.1@tcp0:/lustre-fs` and mounts it in a BusyBox deployment.

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
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lustre-static-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: klustre-csi-static
  resources:
    requests:
      storage: 10Gi
  volumeName: lustre-static-pv
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
        - name: app
          image: busybox
          command: ["sleep", "infinity"]
          volumeMounts:
            - name: lustre-share
              mountPath: /mnt/lustre
      volumes:
        - name: lustre-share
          persistentVolumeClaim:
            claimName: lustre-static-pvc
```

### Validate

```bash
kubectl apply -f lustre-demo.yaml
kubectl exec deploy/lustre-demo -- df -h /mnt/lustre
kubectl exec deploy/lustre-demo -- sh -c 'echo "hello $(date)" > /mnt/lustre/hello.txt'
```

Delete when finished:

```bash
kubectl delete -f lustre-demo.yaml
```

## Pod-level health probe

Use a simple read/write loop to verify Lustre connectivity inside a pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lustre-probe
spec:
  containers:
  - name: probe
    image: busybox
    command: ["sh", "-c", "while true; do date >> /mnt/lustre/probe.log && tail -n1 /mnt/lustre/probe.log; sleep 30; done"]
    volumeMounts:
    - name: lustre-share
      mountPath: /mnt/lustre
  volumes:
  - name: lustre-share
    persistentVolumeClaim:
      claimName: lustre-static-pvc
```

Run `kubectl logs pod/lustre-probe -f` to inspect the periodic writes.

## Where to find more

- `manifests/` directory in the [GitHub repo](https://github.com/klustrefs/klustre-csi-plugin/tree/main/manifests) for installation YAML.
- [Kind quickstart](/docs/advanced/kind/) for a self-contained lab, including the shim scripts used to emulate Lustre mounts.
