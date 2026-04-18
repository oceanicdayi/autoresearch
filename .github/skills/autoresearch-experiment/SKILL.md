---
name: autoresearch-experiment
description: Run autonomous autoresearch experiments in this repository. Use this when asked to set up and execute iterative train.py experiments and track val_bpb results.
---

# autoresearch experiment skill

Use this skill when the user asks to run autoresearch style experiment loops in this repository.

## Repository scope

- Read `README.md` for high-level context.
- Treat `prepare.py` as read-only.
- Edit only `train.py` for experiments unless the user explicitly asks otherwise.

## Setup checklist

1. Propose a run tag (`autoresearch/<tag>`).
2. Create a fresh branch for the run.
3. Verify `~/.cache/autoresearch/` contains data and tokenizer artifacts.
4. If artifacts are missing, ask the user to run: `uv run prepare.py`.
5. Ensure `results.tsv` exists with header:

```tsv
commit	val_bpb	memory_gb	status	description
```

## Experiment loop

1. Start from current kept commit.
2. Make one focused change in `train.py`.
3. Commit.
4. Run:

```bash
uv run train.py > run.log 2>&1
```

5. Read metrics:

```bash
grep "^val_bpb:\|^peak_vram_mb:" run.log
```

6. If run crashes, inspect:

```bash
tail -n 50 run.log
```

7. Append one row to `results.tsv`:
   - `commit`: short hash
   - `val_bpb`: metric (`0.000000` on crash)
   - `memory_gb`: `peak_vram_mb / 1024` to 1 decimal (`0.0` on crash)
   - `status`: `keep` / `discard` / `crash`
   - `description`: short experiment summary
8. Keep commit only when `val_bpb` improves (lower). Otherwise revert to previous kept commit.

## Constraints

- Do not modify `prepare.py`.
- Do not change evaluation logic (`evaluate_bpb` in `prepare.py`).
- Do not add new dependencies.
- Prefer simpler changes when gains are similar.
- Kill runs that exceed 10 minutes and treat as failure.
