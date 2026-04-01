# Request for comments: PostHog Wizard CLI → Web App Communication

## Problem statement

We're pushing the Wizard (`npx @posthog/wizard`) to become the default entry point for PostHog onboarding. It detects the framework, authenticates via OAuth, runs a Claude agent that installs the SDK and modifies code using MCP skills, optionally installs the PostHog MCP server into editor clients, and reports success.

Right now, the wizard runs entirely in the terminal. The web app has no idea it's happening, even if you just started in the web app and ran the command there.

1. **No visibility** — The wizard has a rich TUI (Ink-based React terminal UI) with real-time status, task lists, and screen-by-screen progress. None of this is visible in the PostHog web app.
2. **No continuity** — The wizard finishes, the user opens PostHog in their browser, and they see the same generic onboarding flow as someone who hasn't run the wizard. The web app doesn't know they already installed `posthog-js`, configured session replay, or set up environment variables.
3. **No handoff** — The wizard creates insights and dashboards, but the web app doesn't know about them.
4. **No information preserved about the taxonomy** — The wizard is the best source of truth for the taxonomy of the user's app and even business. Information super valuable to PostHog Code and PostHog AI should be preserved, event schema should be preserved, etc.

We want the wizard to push its state to the PostHog API so the web app can show live progress, adapt onboarding, and capture structured diagnostics.

## Context

### How the wizard actually works today

The wizard is built on the Claude Agent SDK. Here's the real flow as implemented in `bin.ts` → `run.ts` → `agent-runner.ts`:

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

### Architecture overview

![Architecture overview](/images/2026-04-01-wizard-web-communication/wizard-architecture.png)

### Wizard state

Todo items decided by the agent have three states:

```typescript
export enum TaskStatus {
  Pending = 'pending',
  InProgress = 'in_progress',
  Completed = 'completed',
}
```

We hook into the following points:

| Trigger | Wiring |
|--------|--------|
| The wizard agent starts | `RunPhase.Running` |
| The wizard agent shares what it plans to do (event list) | `.posthog-events.json` → `setEventPlan` |
| The wizard agent creates or updates todos | `TodoWrite` in `handleSDKMessage` → `syncTodos` |
| The wizard agent is working a todo | `syncTodos` with `status: 'in_progress'` |
| The wizard agent finishes a todo | `syncTodos` with `status: 'completed'` |

This means, for each of these points, we can `POST` to PostHog with the updated todo list status, so the web app can show the updated todo list.

### PostHog app onboarding

When the app receives signal that a wizard session has started, we can switch to displaying the wizard progress instead of the generic onboarding flow. This essentially replaces the existing `Install` flow.

![Onboarding split screen](/images/2026-04-01-wizard-web-communication/wizard-onboarding-ui.png)

While this is running, we can take the user through the other steps such as:
- `Configure`
- `Session replay`
- `Import data`
- ... etc

We can split the screen into a progress list of what the wizard is doing, and the configuration steps that are available to the user.

The final step can be taking the user to the dashboards and insights that the wizard created.

## What we need to build to ship a V1

I think we can start a V1, where as the user runs the wizard, we sync the state to the web app so they don't feel completely disjoint. There are many super cool things we can do with this I'll discuss in the potential future work section.

### Wizard
- **A new subcommand/option**, maybe `npx @posthog/wizard --sync` that syncs progress to the PostHog API. The wizard will already have an oauth session or API key.
- **State push** — A new `wizard-state-push.ts` that hooks into `WizardStore.emitChange()` and debounces fire-and-forget POSTs to the PostHog API with the current task list, run phase, and event plan. Zero impact on the TUI — if the push fails, the wizard doesn't notice.
- **Upfront insight and dashboard planning**: Right now we do it at the end with MCP tools. It should be planned up front when we decide which events to instrument. It can also just be a terraform file and a single sync call.
- **Upfront event schema planning**: Plan it up front, we can probably push this to the web app for the user to look at.
- **Split the big query**: Right now the Wizard has a locked down, linear, single query. We can split it into the various natural sections of the wizard's workflow. This will give us deterministic hard breaks we can display in the web app/UI.

### PostHog API
- **Wizard sessions endpoint** — `POST /api/projects/{id}/wizard/sessions/` to create a session, `PUT .../state/` to push state updates, `GET .../?status=active` to poll for active sessions (for the future). Simple CRUD, state is probably just a JSONB column overwritten on each push. This will not be reused.
- **Session expiry** — Mark sessions as abandoned if `run_phase = 'running'` and no update for 10+ minutes.

### PostHog Web App
- **Wizard-aware onboarding page** — When an active wizard session is detected, replace the generic "Install" step with live wizard progress (task list with pending/in-progress/completed states).
- **Split-screen onboarding** — Show wizard progress on one side, and configuration steps the user can work through in parallel (Configure, Session replay, Import data, etc.) on the other.
- **Completion handoff** — When the wizard finishes, surface the dashboards and insights it created, and skip any onboarding steps it already completed.
- **Error state** — When the wizard errors, show the error type and last known state so support can diagnose without terminal screenshots.

## Potential future work

The Wizard needs a pretty good round of refactoring before we can really start messing with its behavior, especially with more input from the web app. But when that's possible, there are many super cool things we can do:

- **Two-way communication** — The wizard could poll the web app for commands, or the web app could push commands to the wizard. This would allow the wizard to wait for user input, or to be paused and resumed by the web app.
  - Example 1: On web, the user can tweak which events to instrument, what they look like, which insights and dashboards to create, etc.
  - Example 2: On web, if the user configures remote proxies or makes other config suggestions, we can sync this to the wizard. Maybe via polling.
- **Test correctness** - We can figure out a way to fire test events from the wizard to the web app to test correctness of the wizard's implementation. This builds confidence.
- **Taxonomy** — The wizard is the best source of truth for the taxonomy of the user's app and even business. Information super valuable to PostHog Code and PostHog AI should be preserved, event schema should be preserved, etc.
- **What if we run the wizard in a sandbox?** PostHog Code's gonna build these, I think we can entirely bypass the user's computer. This is already in the plans.
