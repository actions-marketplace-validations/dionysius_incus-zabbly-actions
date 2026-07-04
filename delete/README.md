# incus-zabbly-actions/delete

A GitHub Action that deletes an [Incus](https://linuxcontainers.org/incus/) instance — the opposite of [`launch`](../launch/README.md). It stops the instance first if it is running (always forced), so a single step tears down whatever `launch` created.

## Usage

```yaml
steps:
  - uses: dionysius/incus-zabbly-actions/delete@v1
    with:
      name: ${{ steps.launch.outputs.name }}
```

Typically as a teardown step:

```yaml
  - name: Remove the instance
    if: always()
    uses: dionysius/incus-zabbly-actions/delete@v1
    with:
      name: ${{ steps.launch.outputs.name }}
```

## Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `name` | — (required) | Name of the Incus instance to delete. |

## Notes

Always uses `--force`, so a running instance is stopped and deleted in one go. If the instance does not exist the step succeeds (nothing to delete), which is safe to run in an `if: always()` teardown even when `launch` failed before creating it.
