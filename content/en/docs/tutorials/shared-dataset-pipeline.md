---
title: Share a dataset between prep and training jobs
description: Provision a Lustre-backed scratch space, populate it, and consume it from a training deployment.
weight: 20
---

This tutorial wires a simple data pipeline together:

1. Create a static Lustre `PersistentVolume` and bind it to a `PersistentVolumeClaim`.
2. Run a data-prep job that writes artifacts into the Lustre mount.
3. Start a training deployment that reads the prepared data.
4. Validate shared access and clean up.

## Requirements

- Klustre CSI Plugin installed and verified (see the [Introduction](/docs/)).
- An existing Lustre export, e.g., `10.0.0.1@tcp0:/lustre-fs`.
- `kubectl` access with cluster-admin privileges.

## 1. Define the storage objects

Save the following manifest as `lustre-pipeline.yaml`. Update `volumeAttributes.source` to match your Lustre target and tweak `mountOptions` if required.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lustre-scratch-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  storageClassName: klustre-csi-static
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: lustre.csi.klustrefs.io
    volumeHandle: lustre-scratch
    volumeAttributes:
      source: 10.0.0.1@tcp0:/lustre-fs
      mountOptions: flock,user_xattr
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lustre-scratch-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: klustre-csi-static
  resources:
    requests:
      storage: 100Gi
  volumeName: lustre-scratch-pv
```

Apply it:

```bash
kubectl apply -f lustre-pipeline.yaml
```

Confirm the PVC is bound:

```bash
kubectl get pvc lustre-scratch-pvc
```

## 2. Run the data-prep job

Append the job definition to `lustre-pipeline.yaml` or save it separately as `dataset-job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: dataset-prep
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: writer
          image: busybox
          command:
            - sh
            - -c
            - |
              echo "Generating synthetic dataset..."
              RUNDIR=/mnt/lustre/datasets/run-$(date +%s)
              mkdir -p "$RUNDIR"
              dd if=/dev/urandom of=$RUNDIR/dataset.bin bs=1M count=5
              echo "ready" > $RUNDIR/status.txt
              ln -sfn "$RUNDIR" /mnt/lustre/datasets/current
          volumeMounts:
            - name: lustre
              mountPath: /mnt/lustre
      volumes:
        - name: lustre
          persistentVolumeClaim:
            claimName: lustre-scratch-pvc
```

Apply and monitor (substitute the file name you used above):

```bash
kubectl apply -f dataset-job.yaml  # or lustre-pipeline.yaml
kubectl logs job/dataset-prep
```

Ensure the job completes successfully before moving on.

## 3. Launch the training deployment

This deployment tails the generated status file and lists artifacts to demonstrate read access.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trainer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: trainer
  template:
    metadata:
      labels:
        app: trainer
    spec:
      containers:
        - name: trainer
          image: busybox
          command:
            - sh
            - -c
            - |
              ls -lh /mnt/lustre/datasets/current
              tail -f /mnt/lustre/datasets/current/status.txt
          volumeMounts:
            - name: lustre
              mountPath: /mnt/lustre
      volumes:
        - name: lustre
          persistentVolumeClaim:
            claimName: lustre-scratch-pvc
```

Apply and inspect logs:

```bash
kubectl apply -f trainer-deployment.yaml
kubectl logs deploy/trainer
```

You should see the dataset files created by the job alongside the status text.

## 4. Cleanup

When finished, remove all resources. Because the PV uses `Retain`, data remains on the Lustre share; delete or archive it manually if desired.

```bash
kubectl delete deployment trainer
kubectl delete job dataset-prep
kubectl delete pvc lustre-scratch-pvc
kubectl delete pv lustre-scratch-pv
```

## Next steps

- Adapt the job and deployment containers to your actual preprocessing/training images.
- Add a [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) to refresh datasets on a schedule.
- Use the [Kind quickstart](/docs/advanced/kind/) if you need a disposable lab cluster to iterate on this flow.
