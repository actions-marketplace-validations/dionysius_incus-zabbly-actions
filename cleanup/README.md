# incus-zabbly-actions/cleanup

A GitHub Action that removes [Incus](https://linuxcontainers.org/incus/) and the changes [`setup`](../setup/README.md) made to the runner — the opposite of `setup`. Mostly useful on **self-hosted / persistent runners**; GitHub-hosted runners are discarded after the job, so a cleanup step there is optional.

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/setup@v1
  # ... launch, exec, etc. ...
  - name: Remove Incus
    if: always()
    uses: dionysius/incus-zabbly-actions/cleanup@v1
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `instances` | `true` | Delete all Incus instances (forced) before removing incus. |
| `purge` | `true` | `apt-get purge` the incus package and `autoremove` its dependencies. |

## What it reverses

| `setup` did | `cleanup` undoes |
| --- | --- |
| `apt-get install incus` | `apt-get purge incus` + `autoremove` (also removes `/var/lib/incus` and the setup-edited systemd unit files) |
| Wrote the Zabbly apt repo + keyring | removes `/etc/apt/sources.list.d/zabbly-incus.sources` and `/etc/apt/keyrings/zabbly.asc` |
| Appended `root:<runner uid/gid>:1` to `/etc/subuid`/`/etc/subgid` | deletes exactly those lines |

Not reversed: the `DOCKER-USER` firewall ACCEPT rules `setup` inserts reference the incus bridge by name, so they become inert once incus (and its bridge) are removed.
