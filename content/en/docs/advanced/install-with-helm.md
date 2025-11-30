---
title: Install with Helm
description: Deploy the Klustre CSI plugin using the OCI-distributed Helm chart.
weight: 40
---

The Helm chart is published under `oci://ghcr.io/klustrefs/charts/klustre-csi-plugin`.

## 1. Authenticate (optional)

If you use a GitHub personal access token for GHCR:

```bash
helm registry login ghcr.io -u <github-user>
```

Skip this step if anonymous pulls are permitted in your environment.

## 2. Install or upgrade

```bash
helm upgrade --install klustre-csi \
  oci://ghcr.io/klustrefs/charts/klustre-csi-plugin \
  --version 0.1.1 \
  --namespace klustre-system \
  --create-namespace \
  --set imagePullSecrets[0].name=ghcr-secret
```

Adjust the release name, namespace, and `imagePullSecrets` as needed. You can omit the secret if GHCR is reachable without credentials.

## 3. Override values

Common overrides:

- `nodePlugin.logLevel` – adjust verbosity (`debug`, `info`, etc.).
- `nodePlugin.pluginDir`, `nodePlugin.kubeletRegistrationPath` – change if `/var/lib/kubelet` differs on your hosts.
- `storageClass.mountOptions` – add Lustre mount flags such as `flock` or `user_xattr`.

View the full schema:

```bash
helm show values oci://ghcr.io/klustrefs/charts/klustre-csi-plugin --version 0.1.1
```

## 4. Check status

```bash
kubectl get pods -n klustre-system
helm status klustre-csi -n klustre-system
```

When pods are ready, continue with the [validation instructions](/docs/quickstart/#step-2--verify-the-daemonset) or deploy a workload that uses the Lustre-backed storage class.
