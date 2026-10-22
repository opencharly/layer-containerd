# layer-containerd

The containerd host layer: the daemon installed and configured as `CONTAINERD.md` §3.1 prescribes, with
the one remaining choice that decides whether a Firecracker/Fireactions host can run pools at all — the
**snapshotter**.

## What the layer writes

- a **version-4** `/etc/containerd/config.toml` with `imports = ["/etc/containerd/conf.d/*.toml"]`, its
  gRPC listener under **`plugins."io.containerd.server.v1.grpc"`** (a top-level `[grpc]` is ignored by a
  v4 config and the daemon then binds the default socket — MEASURED), the CRI snapshotter set to
  **`overlayfs`** (CONTAINERD.md §1 rule 3 / §3.3.2; `native` and `blockfile` stay served, so the
  device-less and Firecracker paths remain available), a `crun` default handler, and a `krun` handler
  with `pod_annotations = ["krun.*"]` — both with `SystemdCgroup = true`;
- **`/etc/containerd/seccomp/crabbox-userns.json`**, GENERATED on the host from the installed
  containerd's own `DefaultProfile()` plus an unconditional `{"names":["unshare","clone"],"action":
  "SCMP_ACT_ALLOW"}` rule, with `clone3` left `ENOSYS` and `keyctl`/`mount` still denied
  (CONTAINERD.md §3.1.1) — never a copied blob;
- `kernel.unprivileged_userns_clone=1` persisted, and applied on a live venue, where that sysctl exists;
- the blockfile snapshotter's formatted scratch image (the plugin loads as `error` without it), and the
  `xt_multiport` module declaration the CNI `portmap` plugin needs;
- `containerd.service` and, on the Arch family, `buildkit.service` enabled.

A host with no devmapper device (no LVM thin pool, no spare block device) otherwise loses the whole leg.
The CRI default needs no block device, and a host that *does* have the device can opt into devmapper
deliberately rather than by accident. Making the snapshotter a **declared value** — the typed
`containerd:` surface the umbrella ledger asks for in WI-8 — is filed as opencharly/spec#239.

## What proves it

`check-containerd-vm` — a disposable Arch guest with the layer applied by a nested `local` deploy. The
guest's own systemd runs the packaged units as root (a pod cannot: uid 1000 cannot setuid, which is why
this bed is a VM), and the bed's plan is the three-step shape:

1. **absence** — the distro's **stock** config, run as a private daemon on a private root, state and
   socket, serves no `krun` (`crictl info`), so the presence assertion cannot pass by coincidence. The
   step also reads `ss -lx` and the system socket's inode/mtime, so a test daemon can never touch the
   venue's system socket;
2. **perform** — the member deploy runs the layer's own `run:` steps;
3. **presence** — the running daemon's `crictl info` lists `crun` and `krun`, and the snapshotter the
   effective config declares (`overlayfs`), plus `native` and `blockfile`, report `ok` in
   `ctr plugins ls`.

The bed also proves the config's effective values (read by the daemon's own parser), the narrow seccomp
profile's three contract points, that the store/state roots and the blockfile directory the layer owns are
`root:root`, the user-namespace sysctl where the kernel exposes it, and that the CNI `portmap` plugin can
append a multiport rule inside a rootless network namespace.

## Layout

- `candy/layer-containerd/charly.yml` — the layer: the packages, the rendered config, the seccomp
  generator, the module and sysctl steps, and the preparation step that repairs the ownership of the
  store/state roots and the blockfile directory it owns.
- `charly.yml` — the project manifest, the `containerd-vm-base` template, and the `check-containerd-vm`
  witness. **Every assertion lives in the bed's plan** — the three-step shape, the config contract, the
  seccomp-profile contract, the ownership assertion and the portmap probe. The candy's plan carries the
  install timeline and its mandatory ADE `check:` only: a VM bed resolves no member plan at `check-live`
  and builds no image, so a candy-plan check would run in no phase here.
