# incus-zabbly-actions/launch

A GitHub Action that launches an [Incus](https://linuxcontainers.org/incus/) instance — a system container or a virtual machine.

Requires Incus to be installed first (see [`setup`](../setup/README.md)).

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/setup@v1

  - name: Launch my instance
    uses: dionysius/incus-zabbly-actions/launch@v1
    with:
      image: images:debian/trixie
      name: myinstance
      # vm: true # for virtual machine instead of a system container (if the same image alias offers both)
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `image` | — (required) | Image to launch, e.g. `images:debian/trixie`. |
| `name` | — (required) | Name to give the new instance. |
| `vm` | `false` | Launch as a virtual machine instead of a system container. |
| `config` | `''` | Newline-separated `KEY=VALUE` instance config (each → `-c`), e.g. `limits.cpu=2`. |
| `profiles` | `''` | Comma-separated profiles to apply (each → `-p`). |
| `workspace` | `true` | Mount `$GITHUB_WORKSPACE` into the instance at the same path. |
| `wait` | `exec` | `none`, `exec` (until `incus exec` works), or `cloud-init` (`exec`, then `cloud-init status --wait`). |
| `timeout` | `120` | Seconds to wait for exec readiness when `wait` is `exec`/`cloud-init`. |

## Outputs

| Output | Purpose |
| --- | --- |
| `name` | Name of the launched instance. |

## Notes

With `workspace` on (default), the checkout is at `$GITHUB_WORKSPACE` inside the instance too — set an [`exec`](../exec/README.md) step's `cwd` to it to work there directly.

`wait: cloud-init` only works on images that ship cloud-init (the `.../cloud` variants).
