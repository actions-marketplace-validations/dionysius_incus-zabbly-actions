# incus-zabbly-actions/exec

A GitHub Action that runs a shell script **inside an existing Incus instance** — like a step's `run:` block, but executed in the instance instead of on the runner. It saves you from wrapping everything in `incus exec <instance> -- ...` by hand.

Assumes the instance already exists (create it with [`setup`](../setup/README.md) + `incus launch`).

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/setup@v1
  - id: launch
    uses: dionysius/incus-zabbly-actions/launch@v1
    with:
      image: images:debian/trixie
  - name: Provision inside the instance
    uses: dionysius/incus-zabbly-actions/exec@v1
    with:
      instance: ${{ steps.launch.outputs.name }}
      run: |
        apt-get update
        apt-get install -y cowsay
        cowsay 'hello from inside the instance'
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `instance` | — (required) | Name of the existing Incus instance to run in. |
| `run` | — (required) | Shell script body to execute inside the instance. |
| `shell` | `sh` | Interpreter inside the instance (`sh`-compatible: `sh`, `bash`, `dash`), invoked as `<shell> -e -s`. |
| `cwd` | `''` | Working directory. If set, it's used; otherwise `$GITHUB_WORKSPACE` when shared into the instance (see [`launch`](../launch/README.md)'s `workspace`), else the instance default. |

## Outputs

| Output | Purpose |
| --- | --- |
| `result` | Whatever the body wrote to `$INCUS_RESULT`. |

## Notes

**Environment in.** The runner's environment is forwarded into the instance automatically (minus shell vars like `PATH`/`HOME`), so `$GITHUB_SHA`, workflow `env:`, etc. are available inside the body. Anything exported inside (without `$GITHUB_ENV`) is not available in later steps.

**Env out, to later steps — `$GITHUB_ENV`.** Write to `$GITHUB_ENV` in the body exactly as in a normal step; those vars are exported to subsequent job steps.

**Job summary — `$GITHUB_STEP_SUMMARY`.** Append markdown to `$GITHUB_STEP_SUMMARY` in the body; it lands in the job summary, same as a normal step.

**Data out, as this step's output — `$INCUS_RESULT`.** Write to `$INCUS_RESULT` in any format you like; its contents become the `result` output for a following step to parse:

```yaml
- id: build
  uses: dionysius/incus-zabbly-actions/exec@v1
  with:
    instance: ${{ steps.launch.outputs.name }}
    run: |
      echo "version=1.2.3" >> "$INCUS_RESULT"
      echo "arch=$(uname -m)" >> "$INCUS_RESULT"
- run: echo "${{ steps.build.outputs.result }}"   # parse the lines / KV / JSON / YAML / ... as you like
```

**Whole stdout in `result`.** Redirect the body's output into the file: `run: my-command > "$INCUS_RESULT"`. It then won't appear in the job log — pipe through `tee` (`my-command | tee "$INCUS_RESULT"`) to keep both.
