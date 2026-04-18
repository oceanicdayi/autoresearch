---
name: autoresearch
description: >-
    Run autonomous 5-minute GPT training experiments in this repository by iterating only on train.py and optimizing for lowest val_bpb.
user-invocable: true
---

# autoresearch

Use this skill to run autonomous research loops in this repository.

## Setup

1. Propose a run tag from today (for example `mar5`) and confirm branch `autoresearch/<tag>` does not exist.
2. Create the branch from the current default branch (for example `main`): `git checkout -b autoresearch/<tag>`.
3. Read `README.md`, `prepare.py`, and `train.py`.
4. Check that `~/.cache/autoresearch/` has prepared data and tokenizer; if missing, ask the user to run `uv run prepare.py`.
5. Create `results.tsv` with header:

```bash
printf 'commit\tval_bpb\tmemory_gb\tstatus\tdescription\n' > results.tsv
```

## Constraints

- Only edit `train.py`.
- Do not modify `prepare.py`.
- Do not install new dependencies.
- Ground-truth metric is `evaluate_bpb` from `prepare.py`.

## Experiment loop

For each experiment:

1. Edit `train.py` with one clear idea.
2. Commit.
3. Run: `uv run train.py > run.log 2>&1`.
4. Read metrics: `grep "^val_bpb:\|^peak_vram_mb:" run.log`.
5. If grep is empty, inspect crash with `tail -n 50 run.log`.
6. Append to `results.tsv`:
   - `commit`: short hash
   - `val_bpb`: `0.000000` on crash
   - `memory_gb`: `peak_vram_mb / 1024` rounded to 1 decimal, `0.0` on crash
   - `status`: `keep` / `discard` / `crash`
   - `description`: short experiment summary
7. Keep commit only if `val_bpb` improved (lower). Otherwise reset to previous state.

## Output summary format

Expect the training script to print a block including:

- `val_bpb`
- `training_seconds`
- `total_seconds`
- `peak_vram_mb`
- `mfu_percent`
- `total_tokens_M`
- `num_steps`
- `num_params_M`
- `depth`

## Runtime policy

- Fixed training budget is 5 minutes.
- If a run exceeds 10 minutes, stop and treat as failure.
- If a crash is trivial to fix, fix and retry; otherwise mark crash and move on.
- Continue autonomous experimentation until the user stops you.
