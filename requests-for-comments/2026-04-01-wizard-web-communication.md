# Request for comments: PostHog Wizard CLI → Web App Communication

## Problem statement

The Wizard (`npx @posthog/wizard`) is becoming the default entry point for PostHog onboarding. It detects the framework, authenticates via OAuth, runs a Claude agent that installs the SDK and modifies code via MCP skills, optionally installs the PostHog MCP server into editor clients, and reports success.

Right now, the wizard runs entirely in the terminal. The web app has no idea it's happening — even if the user just started from the web app.

1. **No visibility** — The wizard has a rich TUI with real-time status, task lists, and screen-by-screen progress. None of this is visible in the web app.
2. **No continuity** — After the wizard finishes, the web app shows the same generic onboarding flow as if nothing happened. It doesn't know the user already installed `posthog-js`, configured session replay, or set environment variables.
3. **No handoff** — The wizard creates insights and dashboards, but the web app doesn't surface them.
4. **No taxonomy preserved** — The wizard is the best source of truth for the user's event schema, app structure, and even business domain. This information is valuable to PostHog Code and PostHog AI but gets lost.

We want the wizard to push its state to the PostHog API so the web app can show live progress, adapt onboarding, and preserve what the wizard learned.

## Context

### How the wizard works today

The wizard is built on the Claude Agent SDK. Here's the flow as implemented in `bin.ts` → `run.ts` → `agent-runner.ts`:

![Wizard flow](/images/2026-04-01-wizard-web-communication/wizard-flow.png)

Once the agent starts, it runs a single query. We hook into it to push state to CLI TUI. There is no input mechanism and the app is entirely controlled by the agent.

### What we already have

**`WizardSession`** (`src/lib/wizard-session.ts`) — tracks everything: `RunPhase` (Idle/Running/Completed/Error), integration, credentials, framework context, discovered features, MCP outcome, health check results, outro data. This is the state we want to push.

**`WizardStore`** (`src/ui/tui/store.ts`) — a reactive nanostore wrapper around `WizardSession`. Emits change events, tracks screen transitions, and fires analytics on every transition.

The CLI lifecycle is straightforward:

1. Agent emits changes
2. Changes update the store
3. The UI renders the store

We hook into step 2 to also push state to the web app.

## Design

### Architecture

![Architecture overview](/images/2026-04-01-wizard-web-communication/wizard-architecture.png)

### Wizard state

Agent tasks have three states:

```typescript
export enum TaskStatus {
  Pending = 'pending',
  InProgress = 'in_progress',
  Completed = 'completed',
}
```

We push state at these points:

| Trigger | Wiring |
|--------|--------|
| Wizard agent starts | `RunPhase.Running` |
| Agent shares its plan (event list) | `.posthog-events.json` → `setEventPlan` |
| Agent creates or updates todos | `TodoWrite` in `handleSDKMessage` → `syncTodos` |
| Agent starts working a todo | `syncTodos` with `status: 'in_progress'` |
| Agent finishes a todo | `syncTodos` with `status: 'completed'` |

At each point, we fire-and-forget a POST to PostHog with the current task list and run phase. The web app polls and renders the latest state.

### Onboarding experience

When the web app detects an active wizard session, it replaces the generic "Install" step with live wizard progress. This gives us a split-screen onboarding:

![Onboarding split screen](/images/2026-04-01-wizard-web-communication/wizard-onboarding-ui.png)

The left side mirrors what the wizard is doing in the terminal. The right side lets the user work through configuration steps that don't depend on the wizard (session replay settings, teammate invites, data imports).

When the wizard completes, the onboarding page surfaces the dashboards and insights it created and skips any steps it already handled.

## What we need to build to ship a V1

The goal for V1: when the user runs the wizard, the web app shows live progress so the two don't feel disconnected. The more ambitious ideas are in [Potential future work](#potential-future-work).

### Wizard
- **`--sync` flag** — A new option (`npx @posthog/wizard --sync`) that enables state pushing. The wizard already has an OAuth session or API key by this point.
- **State push module** — A new `wizard-state-push.ts` that hooks into `WizardStore.emitChange()`, debounces, and fire-and-forget POSTs the current task list, run phase, and event plan. If the push fails, the wizard doesn't notice.
- **Upfront insight and dashboard planning** — Currently done at the end via MCP tools. Should be planned upfront when we decide which events to instrument. Could be a Terraform file and a single sync call.
- **Upfront event schema planning** — Plan the schema upfront so we can push it to the web app for the user to review.
- **Split the agent query** — The wizard currently runs one locked-down, linear agent query. Splitting it into natural sections (detect → install → configure → create) gives us deterministic breakpoints we can display in the web app.

### PostHog API
- **Wizard sessions endpoint** — `POST /api/projects/{id}/wizard/sessions/` to create, `PUT .../state/` to push updates, `GET .../?status=active` to poll. Simple CRUD — state is a JSONB column overwritten on each push. Purpose-built, won't be reused.
- **Session expiry** — Mark sessions as abandoned if `run_phase = 'running'` and no update for 10+ minutes.

### PostHog Web App
- **Wizard-aware onboarding** — Detect active wizard sessions and replace the "Install" step with live task progress.
- **Split-screen layout** — Wizard progress on one side, parallel configuration steps on the other.
- **Completion handoff** — Surface the dashboards and insights the wizard created. Skip completed onboarding steps.
- **Error state** — Show the error type and last known state so support can diagnose without terminal screenshots.

## Potential future work

The wizard needs a round of refactoring before we can change its behavior significantly, especially around accepting input from the web app. But once that's done, the state channel unlocks a lot.

### Two-way communication

V1 is one-way: the wizard pushes, the web app reads. Two-way opens up several things:

- **Event schema editing on web** — The wizard proposes events to instrument. The user reviews and tweaks them in the web app — rename events, adjust properties, add or remove events. The wizard polls for the updated schema before writing code. This turns onboarding into a collaborative design step rather than "trust the AI."
- **Insight and dashboard customization** — Same pattern: the wizard proposes dashboards, the user adjusts them on web before the wizard creates them. The user gets exactly the dashboards they want without editing them after.
- **Remote configuration sync** — If the user configures a reverse proxy, custom API host, or other project settings in the web app during the wizard run, those settings get pulled into the wizard via polling. No re-run needed.
- **Pause and resume** — The web app signals the wizard to pause at a natural breakpoint, the user makes decisions on web, then the wizard resumes. Most useful for "which features do you want?" decisions.

### End-to-end verification

The wizard instruments events, but there's no way to verify the instrumentation works until the user deploys and triggers events manually.

- **Test event firing** — After finishing, the wizard starts a dev server, navigates to the app, and fires test events. The web app watches for those events through the ingestion pipeline. If they arrive, green checkmark. If not, surface diagnostics.
- **Schema validation** — Compare arriving events against the schema the wizard planned. Flag mismatches: missing properties, wrong types, events that never fire.

### Taxonomy as a first-class asset

The wizard deeply understands the user's codebase: framework, routes, components, third-party integrations (Stripe, LLM SDKs, etc.). This is the richest taxonomy data we'll ever get for a project.

- **Persist the event schema** — Store planned events, their properties, and descriptions as a proper data model in PostHog. This becomes the source of truth for PostHog AI when answering questions about the user's data.
- **Enrich PostHog Code context** — The wizard's understanding of the codebase (framework, file structure, key components) can be stored and reused by PostHog Code. Instead of rediscovering project structure every time, PostHog Code starts with what the wizard already learned.
- **Taxonomy drift detection** — As the codebase evolves, compare incoming events against the wizard's original schema. Alert when new unschema'd events appear or expected events stop arriving.

### Sandbox execution

PostHog Code is building sandboxed development environments. If we run the wizard in a sandbox, we bypass the user's machine entirely:

- The user clicks "Set up PostHog" in the web app. We clone their repo into a sandbox, run the wizard, and present the changes as a PR.
- The entire wizard run is visible in the web app in real-time — no terminal needed.
- The user reviews the diff, approves, and merges. PostHog is set up without leaving the browser.
- This also makes the wizard testable in CI: run it against a sample repo in a sandbox and verify the output.
