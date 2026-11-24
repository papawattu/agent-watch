# agent-watch rebuild + coxswain integration — plan

Repo: papawattu/agent-watch (fresh; copy functionality, not a port).
Branch: `rebuild`. Dirs: agent_watch/{core,tui,web,cli}, tests/fixtures.

## 0. Layering (the design rule)
- Coxswain = the brain: Loop CRD (phase machine Pending->Planning->AwaitingApproval->Implementing),
  PR gating, operator on k8s. Decides the work.
- agent-watch = the control surface: deterministic tmux-pane watcher, dashboards, keys, mail,
  dispatch, PR review/merge, run-on-mac queue, mutes, notifications.
- Rule: agent-watch READS coxswain state (Loop status + alerts), never re-derives it.
  One source of truth for loop state = coxswain.

## 1. Coxswain integration surface (confirmed from the repo, main 48fad61)
- Loop CRD status (api/v1alpha1/loop_types.go):
  * status.phase (LoopPhase: Pending/Planning/AwaitingApproval/Implementing/...)
  * status.iteration, status.planAttempt, status.desiredPhase, status.observedPhase, status.baseCommit
  * conditions: PlanApproved (planHash), PlanRejected (onReject)
  * status.progress (name + exit code ride in for the next iteration)
- internal/alerts (alerts.go): Alert / Record structs; WriteRecord(w, rec) -> one JSONL line
  (operator's `kubectl logs` shows one-per-line); ClassifyLine(line) -> (Alert, LineKind);
  sources: kubeArmor, egress, toolProxy. This is the structured event feed agent-watch ingests.
- K8s access: kubectl get loop -o json (or the watch on the Loop resource) -> status.

## 2. Build phases (each a working commit)
### Phase 1-2: core (generic, no coxswain yet)
1. Typed config module: all watch_* keys + both AGENT_WATCH_* env schemes, load + re-read on change.
2. Core state: tmux pane capture, classification (profiles: claude/opencode/coxswain/...),
   change tracking, state files (state.json, events.jsonl, reports/agents.json).
3. Key/action registry + core actions (full key table).
4. watchd poll loop + fixed automation order (wake orchestrator, banners, dispatch, mailbox,
   mudge, notify, checks, keep-main).
5. Dispatch (defang, Enter-took, per-profile typing) + notifications (notify_step, mutes.json).
6. Mailbox/agent-msg + run-on-mac queue (next.d).

### Phase 3: coxswain awareness (the differentiator — born coxswain-aware)
7. `coxswain` profile in the classifier: recognise a coxswain loop pane (loop.yaml, .coxswain/,
   phase markers) distinct from a generic coding-agent pane.
8. Coxswain event ingestor:
   * Poll `kubectl get loop <name> -o json` (or a watch) -> status.phase/iteration/planAttempt/
     conditions/progress. On phase change -> emit an agent-watch event (Events tab).
   * Tail the operator's alerts JSONL (WriteRecord one-per-line) -> agent-watch events
     (egress/kubeArmor/toolProxy alerts surface in the Events/Mail tab).
   * AwaitingApproval phase -> raise a "gate" item in the PR/queue dialog (your y/n merge gate).
9. PR-gate dialog backed by coxswain: the `m`/`o`/`q` PR dialog reads coxswain's open PRs
   (gh pr list in the loop's repo) and merge -> `gh pr merge`. One key flow for both.

### Phase 4: surfaces
10. TUI: 6 tabs (Overview/Focus/Events/Mail/Git/Backlog) + peek + dialogs + reusable widgets.
    Overview shows coxswain loop state (phase, iteration, pending gate) at a glance.
11. Web dashboard (127.0.0.1:18090): subset of actions (no merge/Mac/engine changes),
    live-updating loop state + events.
12. CLI (all subcommands) + persona/lessons + suggest + Jira/backlog + git/keep-main/checks.

### Phase 5: finish
13. Tests (hermetic: no tmux/git/network/LLM) + SKILL.md + AGENTS.md.

## 3. Gaps to fill before coding (the must-not-change contract)
G1 keys (all contexts) / G2 CLI / G3 config (~110 keys) / G4 file formats / G5 classification
ground truth / G6 dispatch / G7 poll order / G8 open questions (answered) / G9 web endpoints /
G10 persona+suggest internals.  -> see agent-watch-gaps-prompt.md (ask the Mac agent).

## 4. What "done" looks like
- coxswain runs the loop unattended; agent-watch is where you watch it (Overview: phase + gate)
  and gate it (PR dialog: m/o/q backed by gh). Stay-quiet model: agent-watch only pings on a
  hard blocker or a gate, not routine status.
- agent-watch reads coxswain (Loop status + alerts JSONL); no duplicate loop-state source.
