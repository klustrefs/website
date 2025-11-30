---
title: Bare Metal Notes
description: Notes for operators preparing self-managed clusters before following the main introduction flow.
weight: 20
aliases:
  - /docs/getting-started/baremetal/
---

This guide will describe how to prepare on-prem or colocation clusters where you manage the operating systems directly (kernel modules, Lustre packages, kubelet paths, etc.). While the detailed walkthrough is in progress, you can already follow the general [Introduction](/docs/) page and keep the following considerations in mind:

- Ensure every node that should host Lustre-backed pods has the Lustre client packages installed via your distribution’s package manager (for example, `lustre-client` RPM/DEB).
- Label those nodes with `lustre.csi.klustrefs.io/lustre-client=true`.
- Grant the `klustre-system` namespace Pod Security admission exemptions (e.g., `pod-security.kubernetes.io/enforce=privileged`) because the daemonset requires `hostPID`, `hostNetwork`, and `SYS_ADMIN`.

If you are interested in helping us document more advanced configurations (multiple interfaces, bonded networks, RDMA, etc.), please open an issue or discussion in the [GitHub repository](https://github.com/klustrefs/klustre-csi-plugin).
