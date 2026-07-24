# Hermes Agent — Architecture Assessment (Phase 6)

**Scope caveat, stated up front:** the task brief calls for this
assessment to follow several days of ordinary productive use. What
actually happened is one intensive, real verification session (see
[PRODUCTIVE_RUNTIME.md](PRODUCTIVE_RUNTIME.md)) on 2026-07-24. This
assessment is grounded only in what was truly observed today, plus the
Sprint 2/4 documentation research and install verification. It should
be **revisited after real multi-day use**, especially the Curator
behavior (`min_idle_hours`/`interval_hours` mean it cannot have fired
yet) and the `web_search` finding (needs re-testing over time/across
prompt styles before concluding it's a permanent limitation rather
than a one-off).

## Vollständig durch Hermes gelöst

- **Agent loop, tool dispatch, coding workflow.** Terminal and file
  tools work reliably and are accurately logged; a real coding task
  (write → run → hit a real error → self-correct → verify) completed
  correctly without any PixelHermes-side scaffolding.
- **Session storage and full-text search.** `state.db`, session
  listing, and `session_search` (FTS5) all work exactly as documented
  in `docs/hermes/MEMORY.md`/`ARCHITECTURE.md` — no gap found.
- **Skill discovery/progressive disclosure.** Confirmed live via real
  usage telemetry (`hermes-agent` skill's activity counters moved from
  0 to non-zero during today's sessions) — this is not just documented
  behavior, it's working behavior.
- **Provider abstraction.** Switching to Grok (xAI) as the sole LLM
  required exactly three `hermes config set` calls and worked
  immediately for chat/tools/coding. No custom code, no PixelHermes
  provider layer needed — matches "Hermes orchestriert" from
  `ARCHITECTURE.md`.
- **Git identity / workspace ergonomics.** Git commits from inside a
  Hermes-driven task picked up sensible authorship from the OS account
  automatically; per-CWD session scoping worked without configuration.

## Sinnvoll erweiterbar

- **Multi-user rollout (`hermes_christiane`).** The real
  `ROLLOUT_NOTES.md` produced during Aufgabe 3 is a solid, mostly
  accurate starting point (one stale-field inaccuracy noted in
  PRODUCTIVE_RUNTIME.md) — extending the same Grok+Exa pattern to a
  second Companion user is a small, well-understood step once the
  `web_search` issue below is resolved or accepted.
- **Curator-driven skill hygiene**, once enough real usage accumulates
  over days — nothing to add here, just needs time, not new code.

## Besser nicht anfassen

- **Session storage engine, tool registry, provider abstraction, skill
  engine.** All observed working correctly on their own terms; no
  evidence found that would justify a PixelHermes-side wrapper or
  replacement around any of these. Consistent with the Sprint 2
  PixelHermes-mapping conclusions in each `docs/hermes/*.md` chapter.
- **`config.yaml`/`.env` editing mechanism.** `hermes config set` is
  sufficient and safe (routes secrets correctly, never required a
  manual secrets-file edit today) — no need for a PixelHermes
  configuration-management layer on top.

## Potenzielle Erweiterungspunkte

- **A verification habit/tool for "did the model actually call a
  tool, or fabricate the transcript?"** — today's most important
  finding (Aufgabe 2) is that a Hermes chat transcript alone is not
  trustworthy evidence that a tool ran; only `agent.log`
  (`tool_turns=N`) is. This is a real gap between what Hermes shows
  the user and what's verifiable — worth a documented operating
  procedure (e.g. "trust but verify via `agent.log`" as a habit,
  possibly a small read-only helper script later) rather than a
  Hermes-side change.
- **Robustness around cwd/permission edge cases** (the git-context
  crash found in Aufgabe 4) is a real Hermes-side rough edge — not
  something to patch ourselves (would violate "Upstream First"), but
  worth reporting upstream and/or documenting as an operational
  constraint ("always invoke Hermes from a directory the target user
  can read").

## Klare Upstream-Grenzen

- **`web_search` via Exa not reliably invoked by Grok in this setup**
  — root cause not identified (would require reading xAI Responses-API
  provider integration code, which is squarely upstream Hermes/xAI
  territory, not something PixelHermes should patch around with its
  own workaround). This is now the single most important open question
  blocking a claim of "Exa fully productive" — see
  `docs/hermes/PRODUCTIVE_RUNTIME.md` for full reproduction evidence.
- **`sessions archive --older-than` matching anomaly** — same
  category: an upstream CLI behavior to report/watch, not to
  work around locally.
- **Unreconciled skill counts (69 / 65 / 68)** across three different
  Hermes-internal views of the same installation — an upstream
  bookkeeping inconsistency, not a PixelHermes configuration problem;
  nothing to fix on our end.

## Was fehlt wirklich? Was ist notwendig? Was ist überflüssig?

- **Fehlt wirklich:** verified, reliable web search. As configured,
  Exa is not actually contributing grounded answers — this is a real
  gap against the "Search: Exa (einzige Websuche)" goal stated for this
  phase, not yet met in practice despite correct configuration.
- **Notwendig, aber nicht mehr in diesem Sinne "fehlend":** nothing
  else identified as missing from Hermes' own capabilities for the
  workloads tested today (chat, coding, git, planning, session
  search).
- **Überflüssig (ursprünglich angedachte Erweiterungen, die sich als
  unnötig erweisen):** any PixelHermes-side memory/session/skill
  layer — Hermes' native versions already cover Aufgabe 1–5 completely
  and correctly. mem0/Humalike remain explicitly out of scope for this
  phase and nothing observed today creates new pressure to introduce
  them early — Hermes' native memory/curator stack has not shown any
  functional gap in this session (only that it hasn't run yet, which
  is expected given its 7-day/2-hour-idle trigger conditions).

**No implementation follows from this document** — per the phase
brief, this is evaluation only.
