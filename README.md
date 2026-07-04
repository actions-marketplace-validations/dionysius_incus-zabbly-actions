# incus-zabbly-actions

A collection of GitHub Actions for running [Incus](https://linuxcontainers.org/incus/) (from the [Zabbly](https://github.com/zabbly/incus) packages) in CI — set it up on a runner, launch instances (containers or VMs), and run work inside them.

Work inside an instance **feels like a native job**. The idea is *mount, don't copy*: [`launch`](launch/README.md) shares `$GITHUB_WORKSPACE` and `$RUNNER_TEMP` into the instance at their real paths and maps the instance's root to the runner, so anything run inside reads and writes the runner's actual files. A step running *inside* an instance therefore behaves like an ordinary `run:` step on the runner:

| Native `run:` step | Inside an instance |
| --- | --- |
| `working-directory:` | `cwd` (defaults to the shared `$GITHUB_WORKSPACE`) |
| `shell:`, fail-fast on error | `shell` (`sh`/`bash`/`dash`), invoked `-e` |
| Runner env, `env:`, `GITHUB_*`, `CI`, secrets you export | forwarded into the body automatically |
| Export to later steps via `$GITHUB_ENV` | write `$GITHUB_ENV` — it is the runner's real file |
| Job summary via `$GITHUB_STEP_SUMMARY` | write `$GITHUB_STEP_SUMMARY` — same file |
| Log commands / annotations (`::error::`, `::group::`, `::add-mask::`) | printed by the body, parsed by GitHub (its stdout is the step's stdout) |
| `actions/cache`, `actions/upload-artifact` over the workspace | just work — the files the body wrote are already on the runner, owned by the runner |
| Arbitrary step outputs via `$GITHUB_OUTPUT` | one `result` output via `$INCUS_RESULT` (composite actions can't declare dynamic output keys) |

The only thing that isn't a drop-in is arbitrary named step outputs; everything else is identical or a thin, documented equivalent.

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
      - id: launch
        uses: dionysius/incus-zabbly-actions/launch@v1
        with:
          image: images:debian/trixie/cloud
          vm: true
          config: |
            limits.cpu=4
            limits.memory=4GiB
          wait: cloud-init

      # Build inside the VM, in the shared checkout — like a normal run: block.
      - uses: dionysius/incus-zabbly-actions/exec@v1
        with:
          instance: ${{ steps.launch.outputs.name }}
          run: |
            apt-get update
            apt-get install -y build-essential
            make

      # Artifacts written under the workspace are already back on the runner.
      - run: ls dist
```

See each action's README for full inputs and details.
