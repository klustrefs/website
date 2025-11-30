---
title: Klustre
features:
  - title: Native Lustre mounts
    description: Point a PV at a pre-provisioned Lustre share and KlustreFS makes sure pods mount it using familiar `mount.lustre` options.
    icon: feature-insights.svg
    target_icon: fa-database
    target_label: Lustre share
  - title: Portable HPC pipelines
    description: Reuse the same Lustre-backed scratch space whether the job runs under SLURM or Kubernetes; PVCs keep the paths you already trust.
    icon: feature-ops.svg
    target_icon: fa-server
    target_label: HPC job
  - title: Straightforward PVC bindings
    description: Keep each PVC pointed at a single Lustre export so pods only mount the directories Lustre admins already own.
    icon: feature-insights-alt.svg
    target_icon: fa-folder-tree
    target_label: Namespaced PVC
---

{{< blocks/cover image_anchor="top" height="min" color="primary" >}}
<img src="klustre-logo.svg" class="site-logo" alt="KlustreFS logo">
<h2 class="mt-4">
  High-performance Lustre storage, scheduled natively inside Kubernetes.
</h2>
<div class="mt-5 mx-auto home-hero__cta d-flex flex-column flex-md-row justify-content-center">
  <a class="btn btn-lg btn-primary me-md-3 mb-4" href="/docs">
    Learn more <i class="fas fa-book-open ms-2"></i>
  </a>
  <a class="btn btn-lg btn-secondary mb-4" href="/docs/quickstart/">
    Quickstart <i class="fas fa-arrow-alt-circle-right ms-2"></i>
  </a>
</div>
{{< /blocks/cover >}}

{{% blocks/lead color="white" %}}
  <h1>What is KlustreFS?</h1>
  <p>
    <strong>KlustreFS</strong> is a CSI driver that mounts your existing Lustre shares straight into
    Kubernetes, bridging the gap between HPC and cloud-native infrastructure. Use the same
    PV/PVC workflow you already know while letting AI training workloads and other persistent pods
    access high-throughput (RWM) scratch space—no storage re-platforming required.
  </p>
{{% /blocks/lead %}}

<div class="container">
  {{< home/features >}}
</div>

<div class="home-footer-separator"></div>
