# Super Hermes — Third-Party Skill Pack (Sprint 7)

Real installation of `Cranot/super-hermes` for `hermes_hugo`, evaluated
and installed under the policy in
[ADR 0005](../../ADR/0005-third-party-extension-policy.md). Not an
official Nous Research project — every claim below is either a direct
observation or a source citation, and the non-official status is
stated plainly rather than glossed over.

## What it is (verified, not assumed)

- Repository: `github.com/Cranot/super-hermes`, individual maintainer
  ("Cranot"), MIT license, 318 stars/19 forks at time of review.
- Purpose: teaches Hermes to write its own custom analytical
  "prisms" (structured thinking instructions) for code/artifact
  analysis, then execute them and report findings *and* blind spots.
- Ships **5 skills** (`prism-scan`, `prism-full`, `prism-3way`,
  `prism-discover`, `prism-reflect`) and **7 prism reference files**
  (`error_resilience.md`, `l12.md`, `optimize.md`, `identity.md`,
  `deep_scan.md`, `claim.md`, `simulation.md`).

## Due diligence performed before installing

1. **Full repository read**, not just the README — cloned to a
   temporary location (`/tmp/super-hermes-review`, deleted after
   review), every file listed and every `SKILL.md` and prism `.md`
   read in full.
2. **Content safety review:** all 7 skill/prism files are pure
   analytical prompt text — no code execution logic, no credential
   requests, no exfiltration patterns, no hidden/adversarial
   instructions. Two skills (`prism-scan`, `prism-reflect`) even
   self-restrict via `allowed-tools: ["Read"]` / `["Write", "Read"]`
   frontmatter.
3. **`install.sh` was read but never executed** — it does exactly what
   the manual copy below does (`cp -r skills/* ~/.hermes/skills/`,
   `cp prisms/*.md ~/.hermes/prisms/`), confirming the manual path is
   equivalent, not a lesser substitute.
4. **Core-file-modification check:** confirmed by inspection — nothing
   in the repository touches anything under `~/.hermes/hermes-agent/`
   (Hermes' own source checkout); everything lands under
   `~/.hermes/skills/` and `~/.hermes/prisms/`, both user-writable,
   Hermes-owned directories per its own documented extension model.

## Installation (manual, no third-party script executed)

```bash
git clone --depth 1 https://github.com/Cranot/super-hermes.git /tmp/super-hermes-review
cp -r /tmp/super-hermes-review/skills/prism-scan     ~/.hermes/skills/
cp -r /tmp/super-hermes-review/skills/prism-full     ~/.hermes/skills/
cp -r /tmp/super-hermes-review/skills/prism-3way     ~/.hermes/skills/
cp -r /tmp/super-hermes-review/skills/prism-discover ~/.hermes/skills/
cp -r /tmp/super-hermes-review/skills/prism-reflect  ~/.hermes/skills/
cp /tmp/super-hermes-review/prisms/*.md ~/.hermes/prisms/
rm -rf /tmp/super-hermes-review   # scratch clone, not kept
```

`hermes skills list` confirms all 5 recognized: `Source: local, Trust:
local, Status: enabled` — Hermes' own skill discovery picked them up
with zero additional configuration, exactly as its documented
`~/.hermes/skills/` mechanism promises.

## Verification (real tests, not assumed to work)

- **`/prism-scan hello.py`** on the real `hello.py` from Sprint 6's
  workspace test: genuinely searched for the file (several real
  `find`/`read` tool calls, correctly handled a wrong initial path),
  checked for `.prism-history.md` (correctly reported none found —
  first run), generated a bespoke analytical lens for this specific
  2-line file, executed a 3-way adversarial re-implementation
  analysis, and produced a real, substantive findings table. 19 real
  tool calls, 56 seconds — the skill genuinely works, not just loads.
- **Regression check — ordinary chat still works:** `hermes chat -q
  "Reply with exactly: REGRESSION_OK"` → correct reply, real session
  created.
- **Real, unrelated finding surfaced during this check:** a new
  warning, `⚠ Auxiliary title generation failed: Connection error`,
  appeared on every chat invocation from this point on. Investigated
  before attributing it to Super Hermes: `hermes config get auxiliary`
  shows every auxiliary sub-provider (`vision`, `web_extract`, and by
  the same pattern the title-generation task) configured as `provider:
  auto` with an **empty** `base_url` — a latent, pre-existing gap in
  the auxiliary-model config from Sprint 6, never exercised until this
  test happened to trigger session auto-titling. **Confirmed
  unrelated to Super Hermes**, not just assumed: the warning persisted
  identically after fully removing the skills (see below), proving no
  causal link. Not fixed here — out of scope for this sprint, noted as
  a real, minor open item.
- **Removability — actually tested, not assumed:**
  ```bash
  rm -rf ~/.hermes/skills/prism-{3way,discover,full,reflect,scan} ~/.hermes/prisms
  hermes skills list | grep prism   # → no output, clean
  hermes chat -q "..."               # → still works correctly
  ```
  Confirmed clean, total removal with zero residual state (no config
  keys were ever set for this — it's pure file presence). Then
  re-installed via the same manual copy, since the goal is to keep it
  active — the removal was a verification step, not the end state.

## File uploads

Not applicable to this CLI-based installation — "file uploads" in the
sense of a chat UI attachment feature does not exist in the Hermes CLI
surface used throughout this project; file *access* (reading/writing
workspace files) was exercised extensively above and is unaffected.

## Data footprint / removability summary

| Location | Contents | Removable how |
|---|---|---|
| `~/.hermes/skills/prism-*/` | 5 `SKILL.md` files | `rm -rf`, no other trace |
| `~/.hermes/prisms/*.md` | 7 reference prism files | `rm -rf`, no other trace |
| `.prism-history.md` (if ever created, project-local) | Skill's own "growth" log, written per-project by `/prism-reflect` when used | Not yet created in this session (first runs only used `/prism-scan`); would live wherever the artifact being analyzed lives, not in `~/.hermes/` |

No config.yaml/.env keys, no database rows, no systemd units — the
entire footprint is the file list above.
