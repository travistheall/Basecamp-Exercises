# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Session materials for **Partner Basecamp** — notebooks and supporting files for a two-day
training, organized `day1/` and `day2/`, each folder numbered in run order (`01_...`, `02_...`).
Prepared for Basecamp participants and **not for reproduction or redistribution as training
material** — participants are free to apply the patterns in their own client work, but the
exercises themselves aren't to be repackaged.

Most sessions are "build-alongs": a Jupyter notebook with `✏️` stub cells that participants fill
in live during the session, using the Claude API directly (no agent framework). One session
(`day1/04_diagnosing-ai-problems`) is a diagnostic exercise with no code to write — the fix lives
in editable prompt/tool files. Read each folder's own `README.md` before working in it; it has
the exercise-specific narrative, learning goal, and run steps that this file doesn't repeat.

## Setup & running

No package/build system beyond pip. From the repo root:

```bash
python3 -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...                  # or set AWS_BEARER_TOKEN_BEDROCK + AWS_REGION
```

- Every notebook/script **refuses to run on system Python** — it checks for an active venv/conda
  env before installing anything and exits with a fix-it message if it isn't in one. This is
  intentional (see `SETUP.md`), not a bug to work around, except via the documented
  `BASECAMP_ALLOW_SYSTEM_PYTHON=1` escape hatch for a facilitator's already-provisioned box.
- A single `.env` at the repo root (gitignored) is discovered by every exercise by walking up
  parent directories from the script/notebook location — never put a key in a notebook cell or
  commit one.
- Runs against either an Anthropic API key or an Amazon Bedrock key (`anthropic[bedrock]` extra);
  provider is auto-detected from which env vars are set, Anthropic wins if both are present.
  Exercise code should use the `MODEL` / `FAST_MODEL` / `BIG_MODEL` variables the setup cell
  defines rather than hardcoding a model ID — that's what makes the same code run on both
  providers (Bedrock prefixes IDs with `anthropic.`).
- To run an exercise: open its `.ipynb` in VS Code/Cursor (select the `.venv` interpreter as
  kernel) and run cells, or `cd` into the folder and run `claude` / the `.py` script directly.
  There is no test suite or linter in this repo — "correctness" for a build-along is defined by
  the notebook's own eval/scoring cells, not an external check.

## Structure shared across exercise folders

Each exercise folder is close to self-contained (its own README, notebook, and any `.py`/data
files it needs) and follows the same conventions:

- **Setup cell pattern**: every notebook/script starts with an `_ensure_packages(...)` helper that
  installs only missing deps into the *running* interpreter (normal install → `--user` →
  `--break-system-packages` fallbacks, silent unless everything fails), then the venv/system-Python
  guard, then `.env` discovery, then provider/model resolution. This block is boilerplate — don't
  "clean it up," it's handling real corporate-laptop failure modes (PEP 668, proxies, wrong
  kernel) described in `SETUP.md`.
- **`.py` alongside `.ipynb`**: where both exist (e.g. `day2/02_inference-optimization`), the `.py`
  is the canonical source in jupytext percent format (`# %%` cell markers) and the notebook is
  *generated from it* — edit the `.py`, not the `.ipynb`, or the two will drift. It's also directly
  runnable as a script.
- **Cost awareness**: several exercises make deliberately wasteful/expensive API calls as part of
  the lesson (e.g. inference optimization's v0 baseline costs ~$2-4 to run once). Don't add retries
  or "helpfully" re-run cells to fix something that isn't broken — check the README's cost note
  first.

## `day1/04_diagnosing-ai-problems` — special case

This exercise has its own `CLAUDE.md` in that folder — read it before working there. Key points
it makes that generalize to how you should behave in that folder specifically:

- It's a diagnostic exercise, not a coding one: work the diagnostic method *with* the participant
  (baseline → is it the model? → read the trace → fix the levers → holdout → client brief) rather
  than jumping to the answer.
- The fix lives in `system-prompt-*.txt` and `*-tools.json` (plain text/JSON "agent" definitions
  edited directly, no code). Editing `Diagnose_Fix_Brief.py`'s graders/tickets/scoring to move the
  number defeats the point — that file is the harness, not a lever.
- Every scoreboard run appends to `runs.jsonl` (gitignored) and is worth comparing against before
  re-running; a run takes a minute or two since each ticket runs 5 trials.
