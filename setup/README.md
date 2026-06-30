# incus-zabbly-actions/setup

A GitHub Action that installs and configures [Incus](https://linuxcontainers.org/incus/) from the [Zabbly](https://github.com/zabbly/incus) packages on a GitHub-hosted Ubuntu runner, ready for both system containers and virtual machines (if requirements are met - see [Notes](#notes) for details). It also makes the incus bridge coexist with the runner's preinstalled Docker.

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/setup@v1
    with:
      channel: lts-7.0
  # incus is now installed, initialized, and ready
  - run: incus launch images:debian/trixie c1
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `channel` | `stable` | Zabbly incus line: `stable`, `daily`, or an `lts-X.Y` line (e.g. `lts-7.0`). |
| `group` | `adm` | Socket group for non-sudo `incus` access. |
| `preseed` | `''` | `incus admin init` preseed YAML; empty → `--minimal`. |
| `bridges` | `incusbr0` | Comma-separated bridges to allow through the Docker firewall. |
| `firewall` | `docker-user` | Docker coexistence strategy: `docker-user` \| `flush` \| `none`. |

## Outputs

| Output | Purpose |
| --- | --- |
| `incus-version` | Installed incus version. |

## Notes

VMs need `/dev/kvm` on the runner. GitHub's public **amd64** hosted runners provide it; public **arm64** runners don't. Self-hosted runners should work if they expose `/dev/kvm`.
