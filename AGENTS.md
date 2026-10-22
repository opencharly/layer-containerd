# AGENTS.md — layer-containerd

The containerd layer. The authoritative rulebook is the umbrella `AGENTS.md` in `opencharly/opencharly`;
this file adds only what is specific to this repo.

- **The snapshotter is the whole point.** `native` needs no block device; `devmapper` needs an LVM thin
  pool this layer does not create. Write the config in the distro's own file, never a hand-installed copy
  of the daemon, and never let a missing device fail obscurely — the device-less case must be *visible*.
- **The daemon is asserted where it runs, and that is a VM.** containerd cannot start at image-build time;
  it also cannot run as the init of a POD, which was measured the hard way and is why the pod bed is gone.
  A pod runs its init as uid 1000, where a service `user: root` is inert (a uid-1000 supervisord cannot
  setuid), and containerd then dies on its own requirements — `mkdir /etc/cni: permission denied`, then
  `chown /run/containerd/containerd.sock.ttrpc: operation not permitted`, then `FATAL Exited too quickly`
  under supervisor. The layer still prepares the daemon's own directories, owned by **root** — the identity
  the packaged unit runs as on a host; a store owned by uid 1000 would be writable by an unprivileged user.
  `check-containerd-vm` is the gate, and it passes: the guest's own systemd runs the packaged unit as root
  and `ctr plugins ls` reports the declared snapshotter `ok`.
- `charly box validate` at the repo root is the structural gate; the merge gate is the org-wide
  `charly/pr-validator` (required check `validate / validate`).
- Landing is PR-only, and `tag-on-merge` mints the CalVer tag.
