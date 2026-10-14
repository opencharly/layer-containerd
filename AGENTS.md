# AGENTS.md — layer-containerd

The containerd layer. The authoritative rulebook is the umbrella `AGENTS.md` in `opencharly/opencharly`;
this file adds only what is specific to this repo.

- **The snapshotter is the whole point.** `native` needs no block device; `devmapper` needs an LVM thin
  pool this layer does not create. Write the config in the distro's own file, never a hand-installed copy
  of the daemon, and never let a missing device fail obscurely — the device-less case must be *visible*.
- **The daemon is asserted where it runs.** containerd cannot start at image-build time, so the bed is a
  POD: `ctr plugins ls` is the assertion, and the config under test is the one this layer rendered.
- `charly box validate` at the repo root is the structural gate; the merge gate is the org-wide
  `charly/pr-validator` (required check `validate / validate`).
- Landing is PR-only, and `tag-on-merge` mints the CalVer tag.
