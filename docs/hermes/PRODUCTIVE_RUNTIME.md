# Hermes Agent — Productive Runtime (Phase 6, hermes_hugo, 2026-07-24)

Real observations from bringing `hermes_hugo`'s Hermes installation into
productive use with exactly two providers: **Grok (xAI)** as the sole
LLM, **Exa** as the sole search backend. Hermes itself was not
modified — only `config.yaml`/`.env` were changed via the documented
`hermes config set` mechanism. Everything below is either a literal
captured command/log output or an explicit inference labeled as such —
no guessing.

## Aufgabe 1 — Grok

**Configuration** (via `hermes config set`, never by hand-editing
`.env`/`config.yaml`):
```
hermes config set XAI_API_KEY <key>       # → .env
hermes config set model.provider xai
hermes config set model.default grok-build-0.1
```
`hermes status` afterwards showed `Model: grok-build-0.1`, `Provider:
xAI`, `xAI / Grok ✓ xai-...xzay`. Note: `model.base_url` in
`config.yaml` was left at its stale template value
(`https://openrouter.ai/api/v1`) — real observation: this did **not**
matter. Hermes resolves the real xAI endpoint internally from the
named provider `xai`, ignoring the irrelevant `base_url` field for
named (non-`custom`) providers.

**Verified — Chat:** `hermes chat -q "Reply with exactly: HERMES_GROK_OK"`
→ correct reply, session `20260724_205258_bc6efe`, 7s.

**Verified — Tool calls:** asked it to run `echo TOOLCALL_VERIFY_...`
via the terminal tool → real `terminal` tool call executed
(`agent.tool_executor: tool terminal completed`, confirmed in
`~/.hermes/logs/agent.log`), correct output relayed back.

**Verified — Coding:** asked it to write `hello.py` (function `add`)
and self-test it. It wrote the file (`write_file` tool), tried
`python hello.py` (real failure: `exit 127`, `python: command not
found` — this venv has no `python` alias, only `python3`), self-
corrected to `python3 hello.py`, got `5`, reported it correctly. Three
real tool calls in sequence, all confirmed in the log
(`tool_terminal_completed`/`write_file completed`).

**Verified — Workspace:** see "Aufgabe 3" below — real git/file
operations succeeded inside `hermes_hugo`'s home directory.

## Aufgabe 2 — Exa

**Configuration:**
```
hermes config set EXA_API_KEY <key>
hermes config set web.backend exa
hermes config set web.search_backend exa
hermes config set web.extract_backend exa
```
Before the explicit `web.backend`/`web.search_backend` keys were set,
`hermes config get web` showed all three backend fields as empty
strings (`''`) despite the Exa plugin already auto-registering at
startup (`hermes_cli.plugins: Plugin 'web-exa' registered web
provider: exa`, seen in `agent.log` from the very first CLI invocation
after the key was set) and `hermes doctor`'s `web` toolset already
showing ✓. Real observation: **credential presence alone was not
sufficient** — the explicit backend keys had to be set for the
documented pattern to be complete, which is exactly what this task
asked for.

**Critical finding — web_search is not actually invoked (unresolved):**
Three independent tests, all with explicit, forceful instructions to
call `web_search` and not answer from memory, all resulted in the
model producing a fully fabricated, plausible-looking "raw tool
result" instead of a real tool call:

1. `hermes chat -q "Search the web for: current Debian trixie point
   release..."` → confident-sounding answer with a citation URL, but
   log shows `tool_turns=0`.
2. `hermes chat -q "You must call the web_search tool right now for
   ... show me the raw tool result before answering. Do not answer
   from memory."` → a detailed, formatted fake "raw tool result" for
   kernel.org, `tool_turns=0` confirmed in log.
3. `hermes chat --toolsets web -q "Call web_search now for the exact
   string XKCD927_PROBE_TEST..."` (toolset forcibly narrowed to `web`
   only) → another elaborate fake result set (irrelevant JTAG/FPGA/
   VxRail tech pages), `tool_turns=0` again. Repeated with the
   `grok-4.20-0309-reasoning` model variant — same result, this time
   with visibly garbled/duplicated fake content.

Every one of these is confirmed directly from
`~/.hermes/logs/agent.log`, e.g.:
```
agent.conversation_loop: Turn ended: reason=text_response(finish_reason=stop)
  model=grok-build-0.1 provider=xai ... tool_turns=0 ...
```
i.e. the API call returned a plain text response, never a tool call —
the model fabricated the entire "tool output" as text.

**Ruled out:** the identical fake URLs (`craigjb.com/.../probe-rs`,
`openocd.org/.../riscv-013_8c_source.html`, etc.) recurring verbatim
across three different sessions and two model variants raised the
question of whether this is a hardcoded example baked into Hermes'
own system prompt/tool description. Checked: `grep -r` across the
entire `hermes-agent` source tree for these distinctive strings —
**zero matches**. Not a Hermes-side canned example. Root cause not
identified (would require reading xAI's Responses-API tool-calling
integration in `providers/` in depth, out of scope for this phase —
Hermes stays unmodified either way).

**Conclusion:** Exa is *configured* correctly per the documented
pattern, and the `web` toolset is technically available, but **it is
not reliably being exercised** by this Grok model/provider
combination in practice. Treat "Exa search" as **not production-
verified** until this is root-caused — do not assume search-grounded
answers from this setup are real without checking the log.

## Aufgabe 3 — Workspace (real tasks, not synthetic tests)

Three genuinely useful tasks run in `~/hermes-notes/` (new directory,
not the earlier `~/workspace-test/` coding-verification scratch dir):

1. **Git + Markdown:** "Initialize a git repo, write a README.md
   documenting this directory, commit it." → real `git init`,
   `write_file`, `git add`, `git commit`, `git log` — 8 tool calls, all
   confirmed executed. Real finding: git author was auto-detected as
   `Companion Agent hermes_hugo <hermes_hugo@vps-...>`, sourced from
   the Linux GECOS field set in Sprint 3 — nobody configured git
   identity explicitly.
2. **Project planning:** "Write ROLLOUT_NOTES.md: what's needed to
   bring hermes_christiane onto the same Grok+Exa setup." Real finding:
   this single request triggered **107 tool calls over 4m 6s** — the
   model extensively self-investigated the live system (tried
   `sudo -u hermes_christiane` — got a real permission timeout, i.e.
   it correctly discovered it cannot access that account; read its own
   `hermes-agent` skill's reference docs; read the actual `providers/`
   source for the xai/exa plugins; grepped `.env` for key *names* only,
   never echoed values) before writing a well-grounded, mostly accurate
   plan. One inaccuracy in its own output: it recommended
   `model.base_url https://openrouter.ai/api/v1` for the new user,
   copying the stale, functionally-unused field from the live config
   rather than recognizing it's irrelevant for the `xai` provider
   (see Aufgabe 1 above) — a real instance of the model over-trusting
   incidental config content.
3. **Reasoning task (no tools):** asked for a short, tool-free
   comparison of systemd user vs. system services for an unattended
   gateway. Delivered accurate, well-structured content in 17s with
   zero tool calls — confirms the model's non-tool reasoning quality is
   solid; consistent with what `docs/hermes/DEPLOYMENT.md` already
   documented about this tradeoff from the official docs.

## Aufgabe 4 — Sessions

Real command surface tested against the now-nonzero session set (12
sessions, 149 messages, `state.db` = 1.28 MB after this phase's
activity — confirmed via `sqlite3 ~/.hermes/state.db "SELECT
COUNT(*)..."` matching `hermes sessions stats`):

- **`hermes sessions list`** — accurate, shows preview/workspace/age/
  source/ID per session.
- **`session_search` tool** (invoked via chat, not a CLI subcommand):
  asked the agent to recall the ROLLOUT_NOTES conversation — correctly
  found session `20260724_210715_5accf6` with an accurate one-line
  summary. Real FTS5 search confirmed working, unlike `web_search`.
- **`hermes sessions export --session-id <id> <path>`** — worked,
  produced a 288 KB JSONL file for the 107-tool-call session. Note:
  positional `output` argument, not `-o`; `--session-id` is a filter
  flag, not positional — differs from a plausible first guess.
- **`hermes sessions archive`** — **real anomaly, unresolved:**
  `--older-than 15m`, `--older-than 1m`, even against sessions clearly
  20+ minutes old per `sessions list`, all returned `No sessions match
  (started before ...)` — zero sessions ever matched, despite the
  cutoff timestamps printed being clearly in the past relative to the
  sessions' visible ages. Not investigated further (would require
  reading `hermes_cli`'s session-filtering source, out of scope) — flag
  as an open, reproducible CLI behavior worth re-testing after a real
  multi-day gap rather than minutes.
- **`hermes sessions optimize-storage`** — "Search index is already on
  the compact layout — nothing to do" (clean no-op, as expected for a
  small, fresh DB).
- **`hermes sessions repair`** — "opens cleanly — no repair needed."
- **Real crash found:** running `hermes chat -q ...` for the
  session-search test once from the wrong working directory (my own
  shell's `/home/debian`, not readable by `hermes_hugo`) produced an
  **uncaught, unhandled exception**: `Error: [Errno 13] Permission
  denied: '/home/debian/.git'`, and the CLI exited immediately
  ("Goodbye!") instead of degrading gracefully. Root cause: Hermes'
  git-context detection (`git_repo_root`/`git_branch` in the sessions
  schema, per earlier audit) walks up the directory tree looking for
  `.git` and does not catch permission errors on directories the
  process can't read. Real, reproducible robustness gap — worth noting
  for anyone scripting Hermes invocations across users.

## Aufgabe 5 — Curator (observation only, nothing triggered or changed)

`hermes curator status`:
```
curator: ENABLED
  runs:           0
  last run:       never
  interval:       every 7d
  stale after:    30d unused
  archive after:  90d unused
  consolidate:    off (prune-only; LLM merge pass opt-in)

agent-created skills: 68 total
  active     68 / stale 0 / archived 0
```
Real finding: this "68" is a **third** distinct skill count,
alongside the installer's own sync-log figure ("69 new") and
`hermes skills list`'s figure ("65 builtin enabled") noted in
`docs/hermes/SKILLS.md` and `docs/hermes/OPEN_QUESTIONS.md` — three
different numbers from three different Hermes-internal sources on the
same unmodified installation. Not reconciled; flagged as-is.

Also real: the curator's per-skill usage telemetry is already live —
`hermes-agent` (the self-referential built-in skill) shows
`activity=6, use=3, view=3, last_activity=5m ago`, proving the
`skills_list`/`skill_view` progressive-disclosure mechanism documented
in `docs/hermes/SKILLS.md` is genuinely being exercised by our own
chat sessions above, not just theoretical.

`auxiliary.curator` config is entirely empty (`provider: auto, model:
''`) — curator has never run (`min_idle_hours: 2` combined with
`interval_hours: 168` means it won't fire during a single active
session regardless). Not forced to run — "nur beobachten" was taken
literally: config/status inspected, nothing triggered, nothing changed.

## Aufgabe 6 — Productive use (scope honestly stated)

The task calls for "mehrere Tage" (several days) of ordinary productive
use before drawing conclusions. That is not something a single
conversation turn can produce — what's captured here instead is one
intensive, real verification session (11 real chat invocations, one
real git-backed workspace, real session/curator/tool observations) run
today, 2026-07-24. This is **not** a substitute for multi-day usage;
[ASSESSMENT.md](ASSESSMENT.md) is written with that limitation stated
up front rather than pretending multi-day coverage was achieved.

**Strengths observed:** terminal/file tool calling is reliable and
correctly logged; coding tasks (write + self-test + self-correct on a
real error) worked smoothly; non-tool reasoning quality is good;
session search (FTS5) works and is accurate; git identity and workspace
scoping (`~/.hermes/` per-CWD session tracking) worked without any
manual setup.

**Weaknesses / friction observed:** `web_search`/Exa integration does
not actually fire despite correct configuration — the model fabricates
plausible fake results instead, silently (see Aufgabe 2); a real,
uncaught crash on a permission error during git-context detection; the
`sessions archive --older-than` filter never matched anything in this
session's testing; a single "write a planning doc" request triggered
107 tool calls / 4 minutes of self-investigation — thorough, but
possibly excessive for what should be a quick task; three different,
unreconciled skill counts from three different Hermes-internal
sources.

**Surprises:** the model volunteering a fully-formed, formatted fake
tool-call transcript indistinguishable from a real one at a glance —
this is the most consequential finding of this phase, since it means
naive trust in a Hermes chat transcript alone (without checking
`agent.log`) is not safe for verifying whether a tool actually ran.
