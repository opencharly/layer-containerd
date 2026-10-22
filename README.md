# layer-containerd

The containerd layer: the daemon installed and configured with the one choice that decides whether a
Firecracker/Fireactions host can run pools at all — the **snapshotter**.

A host with no devmapper device (no LVM thin pool, no spare block device) otherwise loses the whole leg.
This layer writes a version-2 `config.toml` on the **`native`** snapshotter, which needs no block device,
so the device-less case works and a host that *does* have the device can opt into devmapper deliberately
rather than by accident.

Making that choice a **declared value** — the typed `containerd:` surface the umbrella ledger asks for in
WI-8 — is filed as opencharly/spec#239 (the constant here is the device-less default that surface keeps).

## What proves it

`check-containerd-vm` — a disposable Arch guest with the layer applied by a nested `local` deploy. The
guest's own systemd runs the packaged unit as root, `ctr plugins ls` reports the declared snapshotter
`ok`, and the bed also proves the CNI `portmap` plugin can append a multiport rule inside a
rootless network namespace (the capability that is missing when the kernel module set has no
`xt_multiport`). A pod cannot run containerd as its init — uid 1000 cannot setuid — which is why this
bed is a VM.

## Layout

- `candy/layer-containerd/charly.yml` — the layer: the packages, the rendered config, the module step, and the checks.
- `charly.yml` — the project manifest, the `containerd-vm-base` template, and the `check-containerd-vm` witness.
