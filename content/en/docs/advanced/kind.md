---
title: Kind Quickstart
description: Stand up a local Kind cluster, simulate a Lustre client, and exercise Klustre CSI Plugin without touching production clusters.
weight: 10
aliases:
  - /docs/getting-started/kind/
---

{{% pageinfo %}}
This walkthrough targets Linux hosts with Docker/Podman because Kind worker nodes run as containers. macOS and Windows hosts cannot load kernel modules required by Lustre, but you can still observe the driver boot sequence. The shim below fakes `mount.lustre` with `tmpfs` so you can run the end-to-end demo locally.
{{% /pageinfo %}}

## Requirements

- Docker 20.10+ (or a compatible container runtime supported by Kind).
- [Kind](https://kind.sigs.k8s.io/) v0.20+.
- `kubectl` v1.27+ pointed at your Kind context.
- `git` (used to fetch the manifests) and a GitHub personal access token with `read:packages` if you plan to pull images from GitHub Container Registry via an image pull secret.

## 1. Create a Kind cluster

Save the following Kind configuration and create the cluster:

```bash
cat <<'EOF' > kind-klustre.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.29.2
- role: worker
  image: kindest/node:v1.29.2
EOF

kind create cluster --name klustre-kind --config kind-klustre.yaml
kubectl cluster-info --context kind-klustre-kind
```

## 2. Install a Lustre shim inside the nodes

The CSI plugin shells out to `mount.lustre` and `umount.lustre`. Kind nodes do not ship with the Lustre client, so we create lightweight shims that mount a `tmpfs` and behave like a Lustre mount. This allows the volume lifecycle to complete even though no real Lustre server exists.

```bash
cat <<'EOF' > lustre-shim.sh
#!/bin/bash
set -euo pipefail
SOURCE="${1:-tmpfs}"
TARGET="${2:-/mnt/lustre}"
shift 2 || true
mkdir -p "$TARGET"
if mountpoint -q "$TARGET"; then
  exit 0
fi
mount -t tmpfs -o size=512m tmpfs "$TARGET"
EOF

cat <<'EOF' > lustre-unmount.sh
#!/bin/bash
set -euo pipefail
TARGET="${1:?target path required}"
umount "$TARGET"
EOF
chmod +x lustre-shim.sh lustre-unmount.sh

for node in $(kind get nodes --name klustre-kind); do
  docker cp lustre-shim.sh "$node":/usr/sbin/mount.lustre
  docker cp lustre-unmount.sh "$node":/usr/sbin/umount.lustre
  docker exec "$node" chmod +x /usr/sbin/mount.lustre /usr/sbin/umount.lustre
done
```

## 3. Prepare the cluster namespace, labels, and image pull secret

```bash
kubectl create namespace klustre-system
kubectl label node klustre-kind-worker lustre.csi.klustrefs.io/lustre-client=true
```

> The default `klustre-csi-static` storage class uses the label above inside `allowedTopologies`. Label any node that will run workloads needing Lustre.

If you have a GitHub token, log in to GHCR and convert your local Docker config into the `ghcr-secret` referenced by the manifests:

```bash
echo '<github-token>' | docker login ghcr.io -u <github-username> --password-stdin
kubectl create secret generic ghcr-secret \
  --namespace klustre-system \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

Alternatively, edit `manifests/daemonset-klustre-csi-node.yaml` and remove the `imagePullSecrets` stanza if you prefer to pull anonymously (the container image is public).

## 4. Deploy Klustre CSI Plugin

```bash
git clone https://github.com/klustrefs/klustre-csi-plugin.git
cd klustre-csi-plugin

kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/

kubectl get pods -n klustre-system -o wide
```

Wait until the `klustre-csi-node` daemonset shows `READY` pods on the control-plane and worker nodes.

## 5. Mount the simulated Lustre share

Create a demo manifest that provisions a static PersistentVolume and a busybox deployment. Because the `mount.lustre` shim mounts `tmpfs`, data is confined to the worker node memory and disappears when the pod restarts. Replace the `source` string with the Lustre target you plan to use later—here it is only metadata.

```bash
cat <<'EOF' > lustre-demo.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lustre-static-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: klustre-csi-static
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
EOF

kubectl apply -f lustre-demo.yaml
kubectl wait --for=condition=available deployment/lustre-demo
kubectl exec deploy/lustre-demo -- df -h /mnt/lustre
kubectl exec deploy/lustre-demo -- sh -c 'echo "hello from $(hostname)" > /mnt/lustre/hello.txt'
kubectl exec deploy/lustre-demo -- cat /mnt/lustre/hello.txt
```

You should see the `tmpfs` mount reported by `df` and be able to write temporary files.

## 6. Clean up

```bash
kubectl delete -f lustre-demo.yaml
kubectl delete namespace klustre-system
kind delete cluster --name klustre-kind
rm kind-klustre.yaml lustre-shim.sh lustre-unmount.sh lustre-demo.yaml
```

## Troubleshooting

- If the daemonset pods crash with `ImagePullBackOff`, double-check that `ghcr-secret` exists in `klustre-system` or remove the `imagePullSecrets` entry from the daemonset manifest.
- The shim script must be present on every node that runs the plugin; rerun the installation loop if you add more nodes.
- Remember that `tmpfs` lives in RAM. Large writes in the demo workload consume memory inside the Kind worker container and disappear after pod restarts. Move to a real Lustre environment for persistent data testing.

Use this local experience to get familiar with the manifests and volume lifecycle, then follow the [main Introduction guide](/docs/) when you are ready to operate against real Lustre backends.
