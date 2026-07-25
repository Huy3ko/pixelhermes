# Request Tracing / Observability (Phase 8.2-ish, no optimization)

A local, structured tracing plugin for `hermes_hugo`, built entirely on
Hermes' own official plugin hook system. **No Hermes core file was
touched, no prompts/providers/memory config were changed, nothing was
optimized** — this is instrumentation and measurement only, per the
explicit brief for this work.

## Where it lives, how it's activated

```
~/.hermes/plugins/companion-tracing/
├── plugin.yaml     # manifest
└── __init__.py     # hook registration + trace writers
```

This is Hermes' own documented **user plugin directory**
(`~/.hermes/plugins/`, confirmed in `docs/hermes/SUPER_HERMES.md`'s
research and Hermes' own `docs/user-guide/features/plugins` page) —
the same mechanism Super Hermes uses, not a special case invented for
this.

Activation:
```bash
hermes plugins enable companion-tracing
```
`hermes plugins list` then shows it as `enabled`, source `user`. When
prompted "Allow this plugin to replace built-in tools?" the answer is
**no** (and was answered no during setup) — this plugin only observes,
it never needs to intercept or replace a tool.

Deactivation, fully reversible, no other state to clean up:
```bash
hermes plugins disable companion-tracing
```

## Why a custom plugin instead of Hermes' own bundled Langfuse plugin

Hermes ships an official, opt-in observability plugin
(`plugins/observability/langfuse/`) using the exact same hooks this
one does. It was seriously considered first. Self-hosting Langfuse
turned out to officially require Docker (Postgres + ClickHouse + Redis
+ MinIO, no lightweight native path documented) — a real conflict with
this project's Docker-free stance held since Sprint 1. Langfuse Cloud
would avoid Docker but sends trace data (potentially including prompt/
response content) to an external service, conflicting with "Self
Hosted". A custom, purely local plugin using the identical hook system
avoids both tradeoffs. Full reasoning: [ADR 0007](../../ADR/0007-local-tracing-plugin-not-langfuse.md).

## Which hooks are used, and why

| Hermes hook | Fires | Used for |
|---|---|---|
| `on_session_start` | once per brand-new session | "request received" marker (every one-shot `hermes chat -q` call creates a new session, so this fires reliably for this project's dominant usage pattern) |
| `pre_api_request` / `post_api_request` | once **per actual LLM API call** (a single turn can make several, e.g. one per tool round-trip) | LLM call timing + token usage — uses Hermes' own precomputed `api_duration` and `usage` fields verbatim, not re-measured |
| `pre_tool_call` / `post_tool_call` | once per tool invocation | every tool call (terminal, file, Honcho's `honcho_*` tools, `web_search`/`web_extract`, skill tools, ...), classified into a category by tool name |
| `on_session_end` | once per turn, at the end of `run_conversation` | closes out the request, computes total duration, aggregates and flushes the summary record |

`post_llm_call` (fires once per turn, coarser than `post_api_request`)
was deliberately **not** registered, to avoid double-logging the same
underlying event — `pre_api_request`/`post_api_request` are the
documented "preferred, more granular" pair (per a comment in Hermes'
own bundled Langfuse plugin source).

## Tool → category classification

| Category | Matched tool names |
|---|---|
| `honcho` | any tool name starting with `honcho_` (`honcho_profile`, `honcho_search`, `honcho_context`, `honcho_reasoning`, `honcho_conclude`) |
| `exa_or_web` | `web_search`, `web_extract` |
| `workspace` | `terminal`, `read_file`, `write_file`, `search_files`, `patch`, `file` |
| `skills` | `skills_list`, `skill_view`, `skill_manage` |
| `other` | everything else (e.g. `memory`, `todo`, `session_search`) |

## Output format

Two files under `~/.hermes/logs/tracing/`:

**`trace.jsonl`** — one JSON object per line, exactly the format
requested:
```json
{"request_id": "20260725_022406_588094:20260725_022406_588094:b3613244", "session_id": "20260725_022406_588094", "platform": "cli", "step": "tool:honcho_reasoning", "category": "honcho", "tool_name": "honcho_reasoning", "start": "2026-07-25T00:24:28.405+00:00", "end": "2026-07-25T00:24:51.993+00:00", "duration_ms": 23587.6}
```
A `request_summary` line closes out every request with the aggregate
counters (see below).

**`trace-summary.log`** — the same data as a human-readable block,
appended per request.

## Real, complete example trace (captured during this work, not fabricated)

Prompt: *"What do you know about my communication preferences? Also
use the terminal tool to run: date"* — a real request that exercised
Honcho recall, a workspace tool call, and multiple LLM round-trips:

```
Request: 20260725_022406_588094:20260725_022406_588094:b3613244  (session 20260725_022406_588094)
llm_request ........... 11683 ms
tool:terminal ......... 112 ms
tool:honcho_profile ... 5 ms
tool:honcho_context ... 48 ms
tool:honcho_search .... 120 ms
tool:honcho_reasoning . 23588 ms
llm_request ........... 8100 ms
tool:memory ........... 8 ms
llm_request ........... 6267 ms

Total                  . 54591 ms
  tool calls: 6 (exa/web: 0, honcho: 4, workspace: 1, skills: 0) | llm calls: 3 | tool time: 23880 ms | llm time: 26050 ms | tokens: 89511 in / 3484 out
```

**This is already a real, useful finding, not a synthetic demo:**
`honcho_reasoning` alone (23.6s) was the single largest contributor to
this request's 54.6s total — larger than any individual LLM call. That
is a genuine, previously-invisible bottleneck this instrumentation
surfaced.

## Aggregate metrics captured per request (the `request_summary` record)

`tool_call_count`, `exa_call_count`, `honcho_call_count`,
`workspace_call_count`, `skill_call_count`, `llm_call_count`,
`total_tool_time_ms`, `total_llm_time_ms`, `prompt_tokens_total`,
`completion_tokens_total`, `total_duration_ms`, `completed`,
`interrupted`.

## Honest coverage: what this can and cannot answer

The brief asked nine specific questions. Answered directly, with no
padding:

| Question | Answerable? |
|---|---|
| Was ist der Flaschenhals? | **Yes** — sum `duration_ms` by `category`/`step` across `trace.jsonl`; in the example above, Honcho's `honcho_reasoning` call dominated a single request more than any other step |
| Wie viel kostet Honcho wirklich? | **Yes**, precisely — `honcho_call_count` + `total_tool_time_ms` filtered to `category: honcho` |
| Wie viel kostet Exa? | **Yes**, precisely (same mechanism, `category: exa_or_web`) — no Exa calls happened to fire in the captured examples, since (per `docs/hermes/PRODUCTIVE_RUNTIME.md`'s unresolved finding) Grok doesn't reliably invoke `web_search` in the first place |
| Wie viel kostet Workspace? | **Yes**, precisely (`category: workspace`) |
| Wie viel kostet RAG? | **No** — Hermes has no distinct "RAG" pipeline step separate from a tool call (e.g. `session_search`); there is no dedicated hook for it, and none was fabricated here |
| Wie hoch ist die reine LLM-Latenz? | **Yes**, precisely, and exact (not estimated) — `total_llm_time_ms`, using Hermes' own `api_duration` |
| Wie hoch ist die TTFT? | **No** — not exposed by any documented Hermes hook (hooks fire on whole-API-call boundaries, not stream chunks); always logged as `null`, never guessed. Also moot for this instance specifically: `streaming: false` in the live `config.yaml` |
| Welche Komponente dominiert die Gesamtlaufzeit? | **Yes** — directly visible per request from the ranked step durations |
| Prompt/Completion Tokens | **Yes**, precisely, from Hermes' own `usage` field |

Not separately measurable, and not faked as separate steps: **Prompt
preparation**, **skill selection**, and **tool planning** all happen
*inside* a single LLM API call (the model decides what to call as part
of generating its response) — there is no Hermes hook that fires
between "model received the prompt" and "model decided to call a
tool." Getting finer resolution there would require instrumenting
inside Hermes' own request-building/response-parsing code, i.e. a real
core-file patch — explicitly out of scope for this phase. **Artifact
generation** has no equivalent concept in Hermes' architecture at all
(unlike e.g. Claude's Artifacts) and is not logged.

## A real bug caught and fixed during this work

The first version of `on_session_start` created its internal state
entry keyed by `session_id`, while every other hook keys its state by
Hermes' `turn_id` (a different string). The two never merged — the
`session_start` entry would have been silently orphaned in memory on
every single request, leaking one entry per session until the plugin's
own LRU cap evicted it, and the step would never have appeared in the
actual request trace. Caught by inspecting the first real test output
(the JSONL was missing a `session_start` line entirely) before this
was documented as "working." Fixed by making `on_session_start` a
simple, immediately-flushed, stateless record instead of trying to
pre-seed the per-request state machinery — see the code comment in
`__init__.py` for the full explanation.

## Verified with

- A plain tool-calling request (terminal `echo`).
- A Honcho-recall-heavy request (`honcho_profile`, `honcho_context`,
  `honcho_search`, `honcho_reasoning`, spanning 3 LLM round-trips).
- A Super Hermes skill invocation (`/prism-scan`, 17 tool calls) — no
  regressions, `skill_call_count: 1` correctly captured.

No performance changes were made anywhere as a result of this work.
