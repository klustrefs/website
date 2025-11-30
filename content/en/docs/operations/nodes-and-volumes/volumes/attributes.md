---
title: Volume attributes and mount options
description: Map Kubernetes fields to Lustre mount flags and behaviors.
weight: 20
---

## `volumeAttributes`

| Key | Example | Purpose |
| --- | --- | --- |
| `source` | `10.0.0.1@tcp0:/lustre-fs` | Host(s) and filesystem path given to `mount.lustre`. |
| `mountOptions` | `flock,user_xattr` | Comma-separated Lustre mount flags. |

Additional keys (e.g., `subdir`) can be added in the future; the driver simply passes the map to the Lustre helper script.

## Storage class tuning

See the [storage class reference](/docs/reference/storage-class/) for details on:

- `allowedTopologies` – keep workloads on nodes with the Lustre label.
- `reclaimPolicy` – typically `Retain` for static PVs.
- `mountOptions` – defaults to `flock` and `user_xattr`, but you can add `noatime`, `flock`, `user_xattr`, etc.

Override mount options per volume by setting `volumeAttributes.mountOptions`. This is useful when a subset of workloads needs different locking semantics.

## Access modes

- Use `ReadWriteMany` for shared Lustre volumes.
- `ReadOnlyMany` is supported when you only need read access.
- `ReadWriteOnce` offers no benefit with Lustre; prefer RWX.

## Lifecycle reminders

- Klustre CSI does not provision or delete Lustre exports. Ensure the server-side directory exists and has the correct permissions.
- Kubernetes capacity values are advisory. Quotas should be enforced on the Lustre server.
- `PersistentVolumeReclaimPolicy=Retain` keeps PVs around after PVC deletion; clean them up manually to avoid dangling objects.
