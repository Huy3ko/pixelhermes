# Hermes Agent Self-Evolution — Installed, Not Run (Sprint 7)

Real installation and configuration of `NousResearch/hermes-agent-self-
evolution` for `hermes_hugo`. **No optimization run was executed** —
only installation and a zero-cost, zero-API-call dry-run validation, as
explicitly requested. Every claim below is a direct observation or
source citation.

## What it is (verified as genuinely official, not assumed)

Verified via the GitHub API (`api.github.com/repos/NousResearch/hermes-
agent-self-evolution`), not just trusted from the URL: `owner.login ==
"NousResearch"`, `owner.type == "Organization"`, matching the same org
ID as the main `hermes-agent` repository. Genuinely official.

Purpose (from its own `pyproject.toml`): "Evolutionary self-improvement
for Hermes Agent — optimize skills, prompts, tool descriptions, and
code using DSPy + GEPA." MIT license, authored by "Nous Research
<team@nousresearch.com>".

## How it's meant to be used against an existing installation

An **offline, separate optimization tool** — it does not hook into a
running Hermes instance or modify it live. It reads skill files from a
pointed-at Hermes repo (`HERMES_AGENT_REPO`, here
`~/.hermes/hermes-agent`), generates candidate variants via DSPy+GEPA
reflective prompt evolution, evaluates them against constraint gates
(test suite, size limits, semantic fidelity), and is designed to
produce **pull requests for human review** — it never commits directly
to the target repo. This project is currently in "Phase 1 (skills
only)"; tool descriptions/prompts/code optimization are stated as
planned, not yet implemented.

## Cost/API-call check performed before installing anything

The task required confirming whether install/config alone triggers any
API calls, before proceeding. Checked directly in
`evolution/skills/evolve_skill.py`:

- The `evolve()` function loads the target skill file, prints its
  metadata, and — if `dry_run=True` — **returns immediately**
  (`if dry_run: ... return`) at line 72.
- `dspy.LM(eval_model)`, the first point anything resembling an API
  client is constructed, appears at **line 140** — unreachable when
  `dry_run=True`, since the function has already returned.
- Dataset generation (`SyntheticDatasetBuilder.generate(...)`, the
  step that actually calls an LLM to build synthetic eval examples)
  happens even later, also unreachable in dry-run mode.

Conclusion: `--dry-run` performs **zero API calls** — confirmed by
source inspection, not just trusted from the `--help` text, and then
confirmed again empirically (see below).

## Installation (real commands, no optimization run)

```bash
sudo -u hermes_hugo -H bash -lc '
  git clone https://github.com/NousResearch/hermes-agent-self-evolution.git ~/hermes-agent-self-evolution
  cd ~/hermes-agent-self-evolution
  uv venv
  source .venv/bin/activate
  uv pip install -e ".[dev]"
'
```
Own venv (`~/hermes-agent-self-evolution/.venv`), kept fully separate
from Hermes' own venv (`~/.hermes/hermes-agent/venv`) — this tool is
not part of the Hermes runtime, it's a standalone offline utility that
happens to read from the same repo checkout.

## Configuration / first test (dry-run only)

```bash
export HERMES_AGENT_REPO=~/.hermes/hermes-agent
python -m evolution.skills.evolve_skill --skill ascii-art --dry-run
```
Output:
```
🧬 Hermes Agent Self-Evolution — Evolving skill: ascii-art
  Loaded: skills/creative/ascii-art/SKILL.md
  Name: ascii-art
  Size: 10,388 chars
  Description: ASCII art: pyfiglet, cowsay, boxes, image-to-ascii....

DRY RUN — setup validated successfully.
  Would generate eval dataset (source: synthetic)
  Would run GEPA optimization (10 iterations)
  Would validate constraints and create PR
```
Target skill chosen deliberately low-risk: `ascii-art` is a small,
self-contained bundled skill (per `docs/hermes/SKILLS.md`'s real
inventory) — this is validation only, no variant of it was ever
generated or considered for promotion.

**Empirically confirmed, not just asserted:**
- `ls datasets/skills/` (where a real run would save generated eval
  data) → empty, no files created.
- `env | grep -i OPENAI_API_KEY` → no match — the dry-run needed and
  used zero credentials, consistent with the source-code finding above.

## What was explicitly NOT done

- No `--optimizer-model`/`--eval-model` API key was configured for
  real use (no `OPENAI_API_KEY` set anywhere for this tool).
- No dataset was generated, no GEPA optimization iteration ran, no
  candidate skill variant was produced, no pull request was created.
- Nothing under `~/.hermes/` was modified by this tool — it only read
  from `~/.hermes/hermes-agent/skills/`.

## Preserved untouched

Grok, Exa, Honcho, the embedding server, and the Hermes runtime itself
were not touched by this installation — this tool lives entirely in
its own directory (`~/hermes-agent-self-evolution/`) with its own
Python environment, and was never invoked in any mode that writes
anywhere.

## Before running a real optimization (future sprint, not now)

Per the task's own cost note ("$2-10 per optimization run"), and per
this project's established practice of never spending against an
external API without an explicit, informed decision: a real run needs
(a) a provider/credential decision for `optimizer-model`/`eval-model`
(currently defaults to `openai/gpt-4.1` / `openai/gpt-4.1-mini` — a
new provider, or an override to something already in use, would need
to be decided explicitly), and (b) the target skill and acceptance
criteria agreed in advance. Not decided or done here — deliberately
deferred.
