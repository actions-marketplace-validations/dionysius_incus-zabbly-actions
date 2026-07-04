# incus-zabbly-actions

A collection of GitHub Actions for running [Incus](https://linuxcontainers.org/incus/) (from the [Zabbly](https://github.com/zabbly/incus) packages) in CI — setting it up on a runner, launching instances, and running work inside them.

## Actions

| Action | Reference | Purpose |
| --- | --- | --- |
| [`setup`](setup/README.md) | `dionysius/incus-zabbly-actions/setup@v1` | Install & configure Incus on a GitHub-hosted Ubuntu runner (containers + VMs), coexisting with the runner's Docker. |
| [`launch`](launch/README.md) | `dionysius/incus-zabbly-actions/launch@v1` | Launch an instance (container or VM), share the workspace in, and wait until it is usable. |
| [`exec`](exec/README.md) | `dionysius/incus-zabbly-actions/exec@v1` | Run a shell script body inside an instance — like a `run:` block, but in the instance. |
| [`delete`](delete/README.md) | `dionysius/incus-zabbly-actions/delete@v1` | Delete an instance (forced, stops it first) — the opposite of `launch`. |
| [`cleanup`](cleanup/README.md) | `dionysius/incus-zabbly-actions/cleanup@v1` | Remove Incus and everything `setup` installed/changed — the opposite of `setup`. |

## Example

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6

      # Install Incus.
      - uses: dionysius/incus-zabbly-actions/setup@v1

      # Launch a VM; the workspace is shared in at $GITHUB_WORKSPACE (default).
      - uses: dionysius/incus-zabbly-actions/launch@v1
        with:
          image: images:debian/trixie/cloud
          name: myinstance
          vm: true
          config: |
            limits.cpu=4
            limits.memory=4GiB
          wait: cloud-init

      # Build inside the VM, in the shared checkout — like a normal run: block.
      - uses: dionysius/incus-zabbly-actions/exec@v1
        with:
          instance: myinstance
          run: |
            apt-get update
            apt-get install -y build-essential
            make

      # Artifacts written under the workspace are already back on the runner.
      - run: ls dist
```

See each action's README for full inputs and details.
