# incus-zabbly-actions/launch

A GitHub Action that launches an [Incus](https://linuxcontainers.org/incus/) instance — a system container or a virtual machine.

Requires Incus to be installed first (see [`setup`](../setup/README.md)).

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/setup@v1

  - name: Launch my instance
    id: launch
    uses: dionysius/incus-zabbly-actions/launch@v1
    with:
      image: images:debian/trixie
      # vm: true # for a virtual machine instead of a system container (if the same image alias offers both)

  - uses: dionysius/incus-zabbly-actions/exec@v1
    with:
      instance: ${{ steps.launch.outputs.name }}   # Incus generated the name; use the output
      run: echo hello
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `image` | — (required) | Image to launch, e.g. `images:debian/trixie`. |
| `name` | `''` | Name for the new instance. Empty lets Incus generate one, returned as the `name` output. |
| `vm` | `false` | Launch as a virtual machine instead of a system container. |
| `ephemeral` | `true` | Make the instance ephemeral — Incus deletes it when it stops. Set `false` to keep it. |
| `config` | `''` | Newline-separated `KEY=VALUE` instance config (each → `-c`). |
| `profiles` | `''` | Comma-separated profiles to apply (each → `-p`). |
| `workspace` | `true` | Mount `$GITHUB_WORKSPACE` into the instance at the same path. |
| `idmap` | `auto` | Root UID/GID mapping on the shared mounts: `auto` (map instance root to the runner → runner-owned files), `none` (no `raw.idmap`, stock Incus ownership), or a verbatim `raw.idmap` payload (e.g. `both 1000 1000`). |
| `wait` | `exec` | `none`, `exec` (until `incus exec` works), or `cloud-init` (`exec`, then `cloud-init status --wait`). |
| `timeout` | `120` | Seconds to wait for exec readiness when `wait` is `exec`/`cloud-init`. |

## Outputs

| Output | Purpose |
| --- | --- |
| `name` | Name of the launched instance (the given name, or the one Incus generated). |

## Notes

The `images:` remote serves the images published at <https://images.linuxcontainers.org/> — browse there for the available distributions, releases, and variants. You can use your own remote by adding it beforehand (`incus remote add ...`).

Available `config:` options can be found at <https://linuxcontainers.org/incus/docs/main/reference/instance_options/>.

With `workspace` on (default), the checkout is at `$GITHUB_WORKSPACE` inside the instance too, and [`exec`](../exec/README.md) runs there by default — see [`exec`](../exec/README.md) for details.

By default (`idmap: auto`) the instance's root is mapped to the runner's uid/gid (`raw.idmap`), so files it creates on the shared mounts come out runner-owned on the host — readable and writable by later runner steps (e.g. `upload-artifact`). Set `idmap: none` for stock Incus ownership, or pass a custom `raw.idmap` payload.

For VMs (`vm: true`) this action sizes the instance by default to the whole runner — `limits.cpu` to the host's CPU count and `limits.memory` to its total RAM. Containers are unaffected (they are not limited, but limits can be set there too).

`wait: cloud-init` only works on images that ship cloud-init (the `.../cloud` variants).
