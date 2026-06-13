# Trajectory archive naming

Convention for prefixed (`_`-leading) directories under `trajectories/songwenzhao/`. Active runs live in unprefixed dirs (whose names are determined by SWE-agent's run config). Anything moved/copied aside follows the scheme below.

## Slot structure

```
_<purpose>__<date>[__<scope>][__<tag>][__<stage>][__<model>]
```

- Slots are separated by **double underscore** `__`.
- Empty optional slots are dropped (no placeholder `__`).
- `<purpose>` and `<date>` are required; the rest are optional.

## Slots

| slot | required | values |
|---|---|---|
| `<purpose>` | yes | `backup` / `trial` / `debug` / `general` |
| `<date>` | yes | `YYYYMMDD` (use mtime of the archived dir) |
| `<scope>` | no | instance count: `3inst`, `10inst`, `13inst`, ... |
| `<tag>` | no | dataset run_id (`exp_no_test_v1` / `exp_no_test_v2`), milestone (`v2` as a generation marker), or descriptive tag (`tiny_error`, `no_coverage_hint`, etc.) |
| `<stage>` | no | `mask` / `problem_gen` / `verifier` / `dev_tools` / `test_gen_secfix` |
| `<model>` | no | `sonnet45` (`claude-sonnet-4-5-20250929`) / `sonnet46` (`claude-sonnet-4-6`) / ... |

Stage + model identify a single-stage traj. Envelope dirs that contain multiple stage-instance subdirs omit `<stage>` and `<model>`.

## Purpose semantics

| purpose | meaning |
|---|---|
| `backup` | Preserved canonical/original state for restoration. Used when a known-good state is about to be overwritten and you want the option to roll back. |
| `trial` | One-off experiment, milestone snapshot, or experimental output kept for analysis. |
| `debug` | Output of a `--debug` invocation (single-iter, no `remove_results` cleanup). |
| `general` | Archived without specific purpose (e.g., a prior generation's runs moved aside before starting the next). |

When uncertain or no purpose fits cleanly, use `general`.

## Examples

```
_backup__20260522__exp_no_test_v1_run1                                # envelope: trial50 prompt-tuning iter1, contains problem_gen + verifier subdirs
_backup__20260525__10inst__exp_no_test_v2__problem_gen__sonnet45      # 10 v2 instances' original problem_gen traj kept for prompt-experiment rollback
_trial__20260605__13inst__exp_no_test_v2__problem_gen__sonnet45       # 13 problem_gen outputs from prompt-tuning experiment
_debug__20260413__run6__mask__sonnet45                                # 6th --debug invocation, mask stage
_general__20260605__v2__mask__sonnet45                                # v2-generation mask archive, moved aside before starting v3
_general__20260408__problem_gen__sonnet46                             # bare archive of an old sonnet46 problem_gen run (no specific tag)
```

## How to apply

- Use `mtime` of the original active dir for `<date>` when archiving. For mv'd contents the mtime is preserved; for new backups (`cp -r`) the date is the day of archival.
- For "next generation" cutover (e.g., starting v3 after a v2 run), `_general__<date>__v2__...` is the standard form.

### Determining the dataset `<tag>`

Prefer the full dataset run_id (e.g., `exp_no_test_v2`) as `<tag>` whenever the traj cleanly belongs to one. Match by instance IDs:

1. List the active traj dir's instance subdirs. Skip batch-level files: `preds.json`, `run_batch.config.yaml`, `run_batch.log`, `run_batch_exit_statuses.yaml`.
2. For each candidate `<run_id>` under `susvibes/datasets/`, load `processed_dataset.jsonl` and collect its `instance_id`s.
3. If one run_id covers (near-)all the instance subdirs with negligible extras → use that run_id as `<tag>` (e.g., `exp_no_test_v2`).
4. If multiple run_ids each contribute (e.g., `dev_tools` whose `run_name` is shared across all `run_id`s, so trajectories accumulate) → join all matching run_ids with `+` (e.g., `exp_no_test_v1+exp_no_test_v2`).
5. If no run_id matches at all (truly external/unknown source) → fall back to a generation marker (`v2`, `v3`, ...).

## Tooling

Bulk rename / archive operations should use a Python script with a preflight (assert every `src` exists and no `dst` collides) before issuing `os.rename`. Never bulk-rename with shell glob — slot boundaries are easy to mis-edit.
