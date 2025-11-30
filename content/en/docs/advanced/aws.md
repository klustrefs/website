---
title: Amazon EKS Notes
description: Outline for deploying Klustre CSI Plugin on managed Amazon EKS clusters backed by Lustre (FSx or self-managed).
weight: 20
aliases:
  - /docs/getting-started/aws/
---

The AWS-oriented quickstart is under construction. It will cover:

- Preparing EKS worker nodes with the Lustre client (either Amazon Linux extras or the FSx-provided packages).
- Handling IAM roles for service accounts (IRSA) and pulling container images from GitHub Container Registry.
- Connecting to FSx for Lustre file systems (imported or linked to S3 buckets) and exposing them via static PersistentVolumes.

Until the full write-up lands, adapt the [Introduction](/docs/) flow by:

1. Installing the Lustre client on your managed node groups (e.g., with `yum install lustre-client` in your AMI or through user data).
2. Labeling the nodes that have Lustre access with `lustre.csi.klustrefs.io/lustre-client=true`.
3. Applying the Klustre CSI manifests or Helm chart in the `klustre-system` namespace.

Feedback on which AWS-specific topics matter most (FSx throughput tiers, PrivateLink, IAM policies, etc.) is welcome in the [community discussions](https://github.com/klustrefs/klustre-csi-plugin/discussions).
