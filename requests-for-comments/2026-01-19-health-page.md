# Request for comments: Unified Health Page

## Problem statement

PostHog surfaces various health and diagnostic information across disconnected pages throughout the platform. We have 4 of them right now, but 3 of them were created in the last couple months, so they might start proliferating:

-   **Ingestion Warnings** – Surfaces broken events detected during ingestion (this is very old)
-   **SDK Doctor** – Alerts users when they're using outdated SDKs
-   **Data Pipelines Problems** – Shows when materialized views fail to materialize
-   **Web Analytics Health** – Checks for incorrect events, missing proxy configuration, etc.

This fragmentation creates several problems:

1. **Discoverability** – Users often don't know these diagnostic tools exist until they hit a problem. There's also no standard way to figure out if there's something broken.
2. **Troubleshooting friction** – When something goes wrong, users have to hunt across multiple pages to diagnose the issue. Even worse, they might not know there's something wrong.
3. **Incomplete picture** – There's no single place to get an overview of your project's health status.
4. **Developer overhead** – Each health check system is implemented independently with no shared infrastructure. We're seeing different teams building these tools now, this is a lot of wasted development time.

## Context

### What are customers experiencing?

Users frequently reach out to support with issues that would have been self-diagnosable if they had found the relevant health page. Common scenarios include:

-   Events not appearing in insights (often caused by ingestion warnings or SDK misconfiguration)
-   Web analytics showing unexpected data (missing proxy, incorrect event tracking)
-   Pipelines silently failing without users noticing

Growth and Onboarding team got together and learned that a lot of the problems the Onboarding team are having to solve manually could be automated since they're usually the same.

### What do we need?

1. **A unified technical infrastructure** for defining and running health checks across all PostHog products
2. **A single Health page** in the UI that aggregates all checks and guides users toward fixes
3. **Proactive surfacing** via the Support panel (`?` icon) – alongside platform status, billing links, etc.
4. **Cost optimization recommendations** surfaced alongside health issues (link to Usage & Billing)
5. **AI-powered suggestions** to help users fix issues without reading docs
6. **Clear handoff to Onboarding team** when self-service isn't enough

## Design

### Architecture overview

The Health system follows a **database-driven, type-based architecture**:

1. **Dagster jobs detect issues and write to the database** – Each health check is a scheduled Dagster job that writes issues as rows with a `type` discriminator and type-specific JSON payload
2. **API serves health issues from the database** – Simple read from DB, no on-the-fly computation
3. **Frontend renders type-specific components** – Each issue `type` maps to a React component that knows how to render that type's JSON payload

This architecture is **scalable**: any team can add a new health issue type by creating a Dagster job, writing to the DB, and registering a frontend component.

We need some preliminary Dagster work to create a framework other teams can use, but we should make sure they can easily add new checks very easily - with proper monitoring for performance problems (iterating over our 300k teams is very slow).

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              Dagster                                     │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                  Scheduled Health Check Jobs (daily)               │  │
│  ├────────────────────────────────────────────────────────────────────┤  │
│  │                                                                    │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │  │
│  │  │ sdk_outdated    │  │ ingestion       │  │ pipeline        │     │  │
│  │  │ _check          │  │ _warning_check  │  │ _error_check    │     │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘     │  │
│  │                                                                    │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │  │
│  │  │ web_analytics   │  │ autocapture     │  │ cost_optimize   │     │  │
│  │  │ _check          │  │ _freq_check     │  │ _check          │     │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘     │  │
│  │                                                                    │  │
│  └──────────────────────────────┬─────────────────────────────────────┘  │
│                                 │                                        │
│                                 ▼                                        │
│                     ┌───────────────────────┐                            │
│                     │   health_issues table │                            │
│                     │      (PostgreSQL)     │                            │
│                     └───────────────────────┘                            │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
                       ┌───────────────────────┐
                       │    /api/.../health    │
                       └───────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────┼────────────────────────────────────────┐
│                            Frontend                                      │
│                                 ▼                                        │
│                  ┌───────────────────────────┐                           │
│                  │    HealthIssueRenderer    │                           │
│                  │   (type → component map)  │                           │
│                  └───────────────────────────┘                           │
│                                 │                                        │
│        ┌────────────┬───────────┼───────────┬────────────┐               │
│        ▼            ▼           ▼           ▼            ▼               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │  SDK     │ │Ingestion │ │ Pipeline │ │WebAnalyt.│ │ <future> │        │
│  │ Outdated │ │ Warning  │ │  Error   │ │  Issue   │ │          │        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘        │
└──────────────────────────────────────────────────────────────────────────┘
```

### Database schema

```sql
CREATE TABLE health_issues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id INTEGER NOT NULL REFERENCES posthog_team(id),

    -- Type discriminator: determines which component renders this issue
    type VARCHAR(100) NOT NULL,  -- e.g., 'sdk_outdated', 'ingestion_warning', 'pipeline_error'

    -- Common fields for all types
    severity VARCHAR(20) NOT NULL,  -- 'critical', 'warning', 'info'
    status VARCHAR(20) NOT NULL DEFAULT 'active',  -- 'active', 'resolved', 'dismissed'

    -- Type-specific payload (each type defines its own schema)
    payload JSONB NOT NULL,

    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    resolved_at TIMESTAMP WITH TIME ZONE,

    -- Prevent duplicate active issues of the same type
    -- Each type can define what makes an issue unique via a hash
    unique_hash VARCHAR(64),

    UNIQUE(team_id, type, unique_hash) WHERE status = 'active'
);

CREATE INDEX idx_health_issues_team_type_status_active ON health_issues(team_id, type) WHERE status = 'active';
```

### Type system

Each health issue type defines:

1. A unique `type` string identifier
2. A JSON payload schema
3. A frontend component to render it

#### Example types and their payloads

**`sdk_outdated`** – SDK is out of date

```typescript
interface SDKOutdatedPayload {
    sdk_name: string // 'posthog-js', 'posthog-python', etc.
    current_version: string // '1.120.0'
    latest_version: string // '1.142.0'
    last_seen_at: string // ISO timestamp
    docs_url: string // Link to upgrade docs
}
```

**`ingestion_warning`** – Event ingestion detected an issue

```typescript
interface IngestionWarningPayload {
    warning_type: string // 'invalid_property', 'missing_distinct_id', etc.
    event_name: string // '$pageview'
    sample_event_ids: string[]
    affected_count: number // Number of events affected in last 24h
    first_seen_at: string
    description: string
}
```

**`pipeline_error`** – Data pipeline failed

```typescript
interface PipelineErrorPayload {
    pipeline_name: string
    pipeline_id: string
    error_message: string
    failed_at: string
    retry_count: number
    logs_url: string
}
```

**`web_analytics_misconfiguration`** – Web analytics setup issue

```typescript
interface WebAnalyticsMisconfigurationPayload {
    issue_type: 'missing_proxy' | 'incorrect_events' | 'domain_mismatch'
    affected_domain: string
    recommendation: string
    setup_url: string
}
```

#### Implementation quality checks

**`autocapture_too_frequent`** – Autocapture firing excessively, causing noise and cost

```typescript
interface AutocaptureTooFrequentPayload {
    events_per_day: number
    top_elements: Array<{ selector: string; count: number }>
    estimated_monthly_cost: number
    recommendation: string // "Consider using allowlist/blocklist to reduce noise"
}
```

**`missing_custom_events`** – User is only using autocapture, missing out on structured analytics

```typescript
interface MissingCustomEventsPayload {
    autocapture_percentage: number // e.g., 98% of events are autocapture
    days_since_last_custom_event: number
    suggested_events: string[] // AI-generated suggestions based on their product type
}
```

#### Cost optimization checks

**`cost_optimization_opportunity`** – User could reduce spend with configuration changes

```typescript
interface CostOptimizationPayload {
    opportunity_type:
        | 'session_replay_sampling'
        | 'disable_web_vitals'
        | 'enable_local_evaluation'
        | 'reduce_identify_calls'
        | 'unused_groups_addon'
    current_usage: Record<string, number>
    potential_savings_percent: number
    recommendation: string
    settings_url: string
}
```

Examples of cost optimization issues:

-   **Session Replay sampling**: "You're recording 100% of sessions. Consider sampling to reduce costs."
-   **Web Vitals**: "Web Vitals are enabled but you're not using Web Analytics. Disable to reduce event volume."
-   **Local evaluation**: "You're making 50k feature flag requests/day. Enable local evaluation to reduce latency and costs."
-   **Over-identifying**: "You're calling `identify()` on 100% of users. Consider only identifying users who sign up."
-   **Unused Groups addon**: "Groups addon is enabled but you haven't sent any group events in 30 days."

### Adding a new health issue type

Any team can add a new type by following these steps:

1. Create a Dagster job (backend) following an already existent framework
2. Register the frontend component (which knows how to render the JSON payload)

That's it. The Health page will automatically pick up and render the new issue type.

### Frontend rendering

The Health page can use a simple component registry pattern:

```tsx
// frontend/src/scenes/health/HealthIssueRenderer.tsx
import { HEALTH_ISSUE_COMPONENTS } from './issueTypes'

export function HealthIssueRenderer({ issue }: { issue: HealthIssue }): JSX.Element {
    const Component = HEALTH_ISSUE_COMPONENTS[issue.type]

    if (!Component) {
        // Fallback for unknown types (e.g., new type deployed to backend but not frontend yet)
        return <GenericHealthIssue issue={issue} />
    }

    return <Component issue={issue} />
}

// Generic fallback renders common fields
function GenericHealthIssue({ issue }: { issue: HealthIssue }): JSX.Element {
    return (
        <div>
            <strong>{issue.type}</strong>
            <p>Severity: {issue.severity}</p>
            <pre>{JSON.stringify(issue.payload, null, 2)}</pre>
        </div>
    )
}
```

### UI Design

#### Main Health page (`/project/:id/health`)

The Health page provides an at-a-glance overview of project health with drill-down capabilities:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Health Overview                                                   [Refresh]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐                │
│  │   ⚠️ 2 issues   │ │   ✅ SDKs       │ │   ✅ Pipelines  │                │
│  │   Ingestion     │ │   Up to date    │ │   All healthy   │                │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘                │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  ⚠️  Ingestion Issues (2)                                               ││
│  ├─────────────────────────────────────────────────────────────────────────┤│
│  │                                                                         ││
│  │  🔴 Invalid event properties detected                                   ││
│  │     15 events in the last 24h have malformed $set properties            ││
│  │     [View affected events]  [Learn how to fix]                          ││
│  │                                                                         ││
│  │  🟡 High cardinality property detected                                  ││
│  │     Property `request_id` has 50,000+ unique values                     ││
│  │     [Review property]  [Learn more]                                     ││
│  │                                                                         ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  ✅  SDKs                                                               ││
│  ├─────────────────────────────────────────────────────────────────────────┤│
│  │     All 3 detected SDKs are up to date                                  ││
│  │     posthog-js v1.142.0 · posthog-python v3.8.2 · posthog-node v4.3.1   ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Category sections

Each category expands to show:

-   Individual check results with status icons
-   Human-readable descriptions of what's wrong
-   Direct links to fix the issue or learn more
-   Historical context when relevant (e.g., "This started 3 days ago")

#### Support panel integration

Rather than adding a new sidebar entry, we integrate with the existing **Support panel** (the `?` icon on the right side of the app). This keeps the navigation clean and puts health information where users naturally look for help.

```
┌─────────────────────────────────────────────────────────┐
│  ❓ Help & Support                              [Close] │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────────┐│
│  │  ⚠️ Project Health                                  ││
│  │  2 issues detected                                  ││
│  │  [View Health Page →]                               ││
│  └─────────────────────────────────────────────────────┘│
│                                                         │
│  ┌─────────────────────────────────────────────────────┐│
│  │  ✅ Platform Status                                 ││
│  │  All systems operational                            ││
│  │  [View Status Page →]                               ││
│  └─────────────────────────────────────────────────────┘│
│                                                         │
│  ───────────────────────────────────────────────────────│
│                                                         │
│  📚 Documentation                                       │
│  💬 Contact Support                                     │
│  💰 Usage & Billing                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

The support panel should:

-   Show a summary of active health issues with severity indicator
-   Link to the full Health page
-   Show platform status (existing feature, but being moved around since we're removing right sidebar)
-   Existing support content

When there are critical issues, the `?` icon itself can show a badge indicator.

#### Cross-linking with Usage & Billing

The Health page should be prominently linked from the **Usage & Billing** section, since cost optimization issues directly relate to spend:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Usage & Billing                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Current spend: $1,234.56                                                   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  💡 3 cost optimization opportunities                                   ││
│  │     We found ways you could reduce your bill. [View in Health →]        ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### AI-powered suggestions

For implementation quality issues, we can leverage **PostHog AI** (Max) to generate contextual recommendations:

```typescript
interface AIEnhancedIssue {
    // Standard issue fields...
    ai_suggestion?: {
        summary: string // "Your autocapture is capturing every button click..."
        code_example?: string // Example code to fix the issue
        estimated_impact: string // "This could reduce your event volume by ~40%"
    }
}
```

Examples:

-   **Autocapture too frequent**: AI analyzes the top captured elements and suggests a specific allowlist config
-   **Missing custom events**: AI looks at the user's product type (from onboarding) and suggests relevant events to track
-   **Cost optimization**: AI calculates estimated savings based on actual usage patterns

The AI suggestions are generated asynchronously and stored in the payload. The frontend component can render them inline with a "✨ Suggested by PostHog AI" badge. They are not LLM-based, just AI - A very smart posthog engineer writing Intelligent code

### Onboarding team handoff

For issues that can't be self-served or require human assistance, include a **"Get help from our team"** action:

```typescript
interface HealthIssuePayload {
    // ... standard fields
    can_request_help?: boolean // Show "Get help" button
    help_context?: string // Pre-filled context for the onboarding team
}
```

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔴 Complex SDK configuration issue                                         │
│                                                                             │
│  Your SDK is sending events with conflicting distinct_id values,            │
│  which may cause person merge issues.                                       │
│                                                                             │
│  [View affected events]  [Learn more]  [🙋 Get help from our team]          │
└─────────────────────────────────────────────────────────────────────────────┘
```

Clicking "Get help from our team":

1. Opens a pre-filled support request with issue context attached
2. Routes to the Onboarding team queue (not general support)
3. Includes diagnostic data: issue type, payload, affected time range, sample events

This creates a direct handoff path for the Onboarding team to help users who need more than self-service docs.

This should, of course, be gated behind the usual access to the Onboarding team OR redirected to the `/merch` store where people can buy our consulting services.

## Alternatives considered

Not do it, but this feels important enough to get right and improve our retention by having streamlined implementations without expanding the Onboarding team

## Success criteria

1. All existing health checks are migrated to the new system
2. Users can access all health information from a single page
3. Time to diagnose common issues is reduced (measure via support tickets)
4. New health checks can be added with minimal boilerplate (<50 lines of code per type)
5. Users discover cost optimization opportunities and reduce spend (measure via billing changes after viewing Health page)
6. Onboarding/Support team receives fewer repetitive manual requests (issues are either self-served or routed with full context)

## Decisions

### 1. Do we persist health check history?

**Decision: Yes, via the `resolved_at` timestamp and keeping resolved issues.**

We keep resolved/dismissed issues in the database (with `status = 'resolved'` or `status = 'dismissed'`). This gives us:

-   **Trend analysis**: "You had 5 ingestion warnings last week, now you have 2"
-   **Audit trail**: "This issue was first detected on Jan 15, resolved on Jan 18"
-   **Recurrence detection**: "This pipeline has failed 3 times this month"

We can add a periodic cleanup job to delete very old resolved issues (e.g., > 90 days) to prevent unbounded growth.

### 2. Should we do some kind of proactive notifications?

i.e. should we reach out to people via email for some of these issues?

**Decision: Tiered approach based on severity and persistence.**

| Severity   | Notification                                                           |
| ---------- | ---------------------------------------------------------------------- |
| `critical` | Badge on `?` icon immediately. Optional email after 24h unresolved.    |
| `warning`  | Shown in Support panel summary. No badge on icon. No email by default. |
| `info`     | Only visible on Health page.                                           |

Users can configure notification preferences in project settings:

-   Enable/disable email notifications per severity
-   Set quiet hours
-   Mute specific issue types

This does NOT need to be included on our initial launch, but it's ideal that we follow-up with this.
