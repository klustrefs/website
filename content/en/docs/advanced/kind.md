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
- A GitHub personal access token with `read:packages` if you plan to pull images from GitHub Container Registry via an image pull secret (optional but recommended).

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

## 3. Prepare node labels

Label the Kind worker node so it is eligible to run Lustre workloads:

```bash
kubectl label node klustre-kind-worker lustre.csi.klustrefs.io/lustre-client=true
```

> The default `klustre-csi-static` storage class uses the label above inside `allowedTopologies`. Label any node that will run workloads needing Lustre.

## 4. Deploy Klustre CSI Plugin

Install the driver into the Kind cluster using the published Kustomize manifests:

```bash
export KLUSTREFS_VERSION=main
kubectl apply -k "github.com/klustrefs/klustre-csi-plugin//manifests?ref=$KLUSTREFS_VERSION"
```

Then watch the pods come up:

```bash
kubectl get pods -n klustre-system -o wide
```

Then wait for the daemonset rollout to complete:

```bash
kubectl rollout status daemonset/klustre-csi-node -n klustre-system --timeout=120s
```

Wait until the `klustre-csi-node` daemonset shows `READY` pods on the control-plane and worker nodes.

## 5. Mount the simulated Lustre share

Create a demo manifest that provisions a static PersistentVolume and a BusyBox deployment. Because the `mount.lustre` shim mounts `tmpfs`, data is confined to the worker node memory and disappears when the pod restarts. Replace the `source` string with the Lustre target you plan to use later—here it is only metadata.

Create the demo manifest with a heredoc:

```bash
cat <<'EOF' > lustre-demo.yaml
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
      # This is only metadata in the Kind lab; replace with a real target for production clusters.
      source: 10.0.0.1@tcp0:/lustre-fs
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
            - name: lustre
              mountPath: /mnt/lustre
      volumes:
        - name: lustre
          persistentVolumeClaim:
            claimName: lustre-demo-pvc
EOF
```

Apply the demo manifest:

```bash
kubectl apply -f lustre-demo.yaml
```

Wait for the demo deployment to become available:

```bash
kubectl wait --for=condition=available deployment/lustre-demo
```

Confirm the Lustre (tmpfs) mount is visible in the pod:

```bash
kubectl exec deploy/lustre-demo -- df -h /mnt/lustre
```

Write and read back a test file:

```bash
kubectl exec deploy/lustre-demo -- sh -c 'echo "hello from $(hostname)" > /mnt/lustre/hello.txt'
```

```bash
kubectl exec deploy/lustre-demo -- cat /mnt/lustre/hello.txt
```

You should see the `tmpfs` mount reported by `df` and be able to write temporary files.

## 6. Clean up (optional)

Remove the demo PV, PVC, and Deployment:

```bash
kubectl delete -f lustre-demo.yaml
```

If you want to tear down the Kind environment as well:

```bash
kubectl delete namespace klustre-system
kind delete cluster --name klustre-kind
rm kind-klustre.yaml lustre-shim.sh lustre-unmount.sh lustre-demo.yaml
```

## Troubleshooting

- If the daemonset pods crash with `ImagePullBackOff`, use `kubectl describe daemonset/klustre-csi-node -n klustre-system` and `kubectl logs daemonset/klustre-csi-node -n klustre-system -c klustre-csi` to inspect the error. The image is public on `ghcr.io`, so no image pull secret is required; ensure your nodes can reach `ghcr.io` (or your proxy) from inside the cluster.
- If the demo pod fails to mount `/mnt/lustre`, make sure the shim scripts were copied to every Kind node and are executable. You can rerun the `docker cp ... mount.lustre` / `umount.lustre` loop from step 2 after adding or recreating nodes.
- Remember that `tmpfs` lives in RAM. Large writes in the demo workload consume memory inside the Kind worker container and disappear after pod restarts. Move to a real Lustre environment for persistent data testing.

Use this local experience to get familiar with the manifests and volume lifecycle, then follow the [main Introduction guide](/docs/) when you are ready to operate against real Lustre backends.
