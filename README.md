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

`check-containerd-pod` — a disposable pod running this layer's own image, asserting that containerd starts
with the rendered config and that `ctr plugins ls` reports the declared snapshotter `ok`. A pod has no
spare block device, which is exactly the device-less case: the ledger requires it to be **visible** rather
than silently missing.

## Layout

- `candy/layer-containerd/charly.yml` — the layer: the package, the rendered config, and the checks.
- `box/check-containerd-pod/charly.yml` — the bed's image: a Fedora base composing the layer under test.
- `charly.yml` — the project manifest and the `check-containerd-pod` witness.
