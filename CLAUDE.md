# genesis-e2e-tictactoe

## Goal

Build a two-player tic-tac-toe web app deployed to GitHub Pages.

Requirements:
- A single static page (HTML + CSS + inline or co-located JS) served from GitHub Pages on the `main` branch (either root or `/docs`).
- Two players alternate clicks: X goes first, then O. After each click, check for a win or draw and display the result.
- A "reset" control restores an empty board.

Testability contract (must be honored — an external Playwright e2e test depends on these selectors):
- Each cell is an interactive element (button or div) with attribute `data-cell="N"` where N is 0..8, ordered left-to-right then top-to-bottom.
- After a move, the cell's text content is exactly "X" or "O" (uppercase, no extra whitespace).
- A status element with `id="status"` displays the current game state. Its text contains the word "win" (case-insensitive) when a player wins, "draw" when the game ends in a draw, and neither otherwise.
- A reset element with `id="reset"` clears the board when clicked.

Quality: no frameworks needed, but the code should be clean and tested if practical. Deploy via GitHub Actions or the standard Pages-from-branch mechanism — whichever you prefer.




## Meta-Concepts

These are the principles this dev system operates by. Evolve them as the project matures.

- **GitHub as coordination layer** — issues track progress, PRs deliver changes, CI/CD enforces quality. Humans and agents speak the same protocol.
- **Quality gates and e2e testing** — code, tests, CI/CD, deployment are all first-class concerns.
- **Self-improvement** — continuously evolve agents, skills, and strategies.
- **Self-monitoring** — monitor progress, detect stuck/looping states, try to unblock, escalate to human when stuck.
- **Minimal human-in-the-loop** — do everything possible autonomously. Highlight what requires human action and offer to do it if given access.
- **Deterministic over agentic** — if a task is well-understood and doesn't need LLM judgment, build a deterministic tool (script, CLI, CI step). Reserve LLMs for fuzzy reasoning.
- **Incremental planning** — only detail the current milestone. Future milestones stay high-level until they're next.

## Agent Roster

- **Onboarding** — refines goal with human, produces milestones (runs once at project start)
- **Project manager** — owns roadmap, tracks progress, drills down current milestone into tasks
- **Human interaction** — all comms with user (reports, escalations, access requests). Speaks A2H protocol.
- **Evolver** — evolves the dev system itself (new agents, tools, skills, memory design, CLAUDE.md refinement). Escalates framework-level improvements to genesis.
- **Health / self-review** — monitors for stuck/looping, audits quality
- **Workers** — designed by the dev system for the specific goal

## Execution Model

GitHub Actions serve as the trigger layer:
- **Scheduled workflows** (cron) — periodic advancement of project state
- **Event-triggered workflows** — issue/PR events, human feedback, comments

Each trigger launches a Claude Agent SDK session as the orchestrator.

## Lessons Learned (curated by Evolver)

- **GitHub Pages enablement is human-gated.** The Genesis App lacks `pages: write`, so `gh api -X POST .../pages` returns 403. Don't keep retrying — surface a `needs:human` task explaining Option A (toggle Settings → Pages), Option B (grant the App `pages: write`), or Option C (Actions-based deploy with default `GITHUB_TOKEN`). See issue #5 for the template.
- **Orchestrator runs are now serialized.** Workflows `genesis-orchestrator.yml` and `genesis-events.yml` share `concurrency.group: genesis-orchestrator` to prevent overlapping runs from creating duplicate plan/completion issues (regression observed: two M1 plan issues #2 and #3 were created 30s apart by concurrent `workflow_dispatch` invocations).
- **Two `workflow_dispatch` runs of the orchestrator within seconds will race** — even with the duplicate-issue rule in the agent prompt, the second run won't see the first's not-yet-posted issue. The concurrency group is the structural fix; the prompt rule is defense in depth.
- **Scheduled orchestrator skips when human-gated.** `genesis-orchestrator.yml` has a deterministic precheck: if any open issue has label `needs:human` AND title matching `\b(plan|complete)\b`, the scheduled tick exits without invoking Claude (was burning ~$0.30 and ~1m per tick to re-derive "wait"). Manual `workflow_dispatch` runs are *not* skipped — a human invoking the workflow always means they want it to run. When the human closes the gate, the `issues.closed` event in `genesis-events.yml` triggers a real run.
- **Events orchestrator needs the same turn budget as scheduled.** Run #25650615146 (issue #2 plan creation) failed with `error_max_turns` at 10 turns and cost $1.04 for an aborted session. Plan/completion work routinely needs 10-15 turns (assess state + draft body + create issue + label). `genesis-events.yml` now uses `--max-turns 20` to match the scheduled orchestrator.
- **Evolver skips when no activity since last run.** `genesis-evolver.yml` has a deterministic precheck: it queries new commits, closed issues, and failed workflow runs since the last successful evolver run. If all three are zero, the Claude invocation is skipped. Without this, a quiet project (e.g. paused on a `needs:human` gate) was paying ~$1.20-$1.87/day for evolver sessions that produced no commits. Manual `workflow_dispatch` always runs.
- **Skipping the scheduled orchestrator also disables its "stuck for 2 cycles" escalation.** The orchestrator agent's guideline "If something is stuck for more than 2 cycles, escalate via the human interaction agent" is dead code while a `needs:human` plan/completion gate is open, because the scheduled tick never invokes Claude. The current design accepts this: human gates are *intentional* waits, and we don't want to pester the human. Stale-gate nudges are handled by `genesis-stale-gate-reminder.yml`: a deterministic daily cron that posts exactly one comment on each `needs:human` issue older than `STALE_DAYS` (default 2) and stamps `reminder-sent` to prevent re-firing. This matches the human-interaction agent's "One reminder for blocking requests. Don't escalate further" rule without re-enabling a Claude session for a fixed message.

## Tech Stack Preferences

Defaults (override as needed):

- **Open source + free tier only**
- **Backend:** Rust (Go if K8s-heavy)
- **CLI:** Rust
- **Frontend:** Vite + React + TanStack Router + TanStack Query, Tailwind CSS, TypeScript (strict)
- **Desktop:** Tauri
- **Mobile:** React Native (Expo)
- **Internal services:** gRPC
- **Auth:** Ory stack (K8s), Rust crates (simple apps), Clerk (managed fallback)
- **Observability:** OpenTelemetry + Grafana Cloud free tier
- **Database:** Neon (serverless Postgres)
- **Deployment:** Cloudflare, cloud free-tier
- **Local dev:** Tilt + kind (K8s), LocalStack (AWS)
