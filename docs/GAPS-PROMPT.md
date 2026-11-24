You have agent-watch-inventory.md. I'm rebuilding agent-watch fresh (same functionality, not a port). I have the architecture, the main-context key table, and the CLI list. I need the GAPS filled — answer ONLY these, terse, no code, copy values verbatim from the source. If a value is genuinely unknown in the source, say UNKNOWN (don't guess).

1. KEYS — full table, every context (main, each picker, PR dialog, Mac queue, prompt, confirmation, footer) + all modifiers. Format: key | context | what it does. Note any key that does different things in different contexts.
2. CLI — every subcommand + flag + exit code + output format. Resolve the doc's own note: is `launch [names…]` and `jump` really missing from SKILL.md Commands, or is SKILL.md the authoritative list?
3. CONFIG — all ~110 watch_* keys + the 5 key families: name | type | default | what it does | AGENT_WATCH_* env override. List BOTH env-naming schemes and the irregular names.
4. FILE FORMATS — for each (state.json, events.jsonl, reports/agents.json, mailbox, next.d, lessons, prefs, merged-PR records, pr-cache.json, config.yaml): fields + one example line. For events.jsonl list every event kind + its fields.
5. CLASSIFICATION — tests/fixtures/*.txt names + the pane states/profiles (working/blocked/done/error, spinner glyphs, input box) + the exact detect rules (GENERIC_WAIT, GENERIC_ERROR, claude vs opencode differences).
6. DISPATCH — the defang rules, the Enter-took check, and per-profile typing behaviour (typed text vs prompt).
7. POLL ORDER — the exact fixed step sequence in one poll, and which steps run under which config flags.
8. OPEN QUESTIONS — the doc raised questions (e.g. M11 pr-cache refresh path). Give the ANSWER to each, not the question.
9. WEB — the exact API endpoints + which actions the web dashboard exposes vs the terminal.
10. PERSONAS/SUGGEST — how personas/lessons are stored, how `suggest` calls the LLM (which model), and what runs tests/test_persona.py.

Keep it short. Terse tables/lists. No prose padding.
