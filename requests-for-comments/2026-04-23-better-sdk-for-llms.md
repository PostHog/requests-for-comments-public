# Request for comments: How to make PostHog SDKs better for LLMs

## Problem statement

LLMs are becoming the default interface for developers to interact with PostHog, includingh PostHog's SDKs. There are various quirks of the PostHog SDKs that make it harder for LLMs (and humans) to implement PostHog correctly the first time. This can add friction for LLMs implementing PostHog.

It's important to highlight that in practice, LLMs will recover from most of these issues. But it may take a few turns (API round trips) to do so. On long conversations with context windows in the hundreds of thousands, this makes it significantly slower and increases cost.

This RFC covers the following:
- Catalogs the inconsistencies we've hit, grouped by area, and marks each with how it should be fixed.
- Proposes various action items to fix the inconsistencies.
- Proposes a new "commandments" style of documentation and skill to guide LLMs on how to use the SDKs correctly.

Many of these are very nit-picky. This RFC is really to surface the problems, I acknowledge that some of these are not worth the effort to fix.

## Proposed action items

In the [problem areas](#problem-areas) section, we've identified a number of problems that we've found necessary to explicitly explain to LLMs to make them work correctly. 
- Some of these can be fixed with tweaks to the SDKs
- Many of these are just language-specific quirks that we can't engineer around

What I propose is to do two things. Obviously, I'll suggest areas to improve the SDKs themselves. I also propose that we move some of the "commandments" that are special notes for LLMs to not fumble into the SDK repos themselves. I really think some of these "commandments" for LLMs to read are going to stay, but we should move them to closer to the SDKs and load them into the docs and skills during build.

Many of these are very nit-picky and are not worth the effort to fix. At least we'll ship some commandments for LLMs to read to help them avoid the quirks.

## Problem areas

Here are some problem areas we've identified while building the PostHog Wizard and various skills. These are problems that we've found necessary to explicitly explain to LLMs to make them work correctly. LLMs (specifically Claudes's various models) frequently trip over these areas when implementing PostHog.

Within each section, entries are ordered from highest to lowest LLM-tripping severity — silent failures (no error surfaces, LLM has no feedback signal) are listed before loud failures (clear errors the LLM can recover from in a round or two).

**Legend** proposed solutions:
- `[DOCS]` fix with better docs
- `[SDK]` fix with code
- `[BOTH]` needs both

### Naming and config shape

#### Feature-flag timeout has five names and two units

 `feature_flag_request_timeout_ms` (JS/Node/PHP), `featureFlagsRequestTimeoutMs` (RN), `feature_flags_request_timeout_seconds` (Python), `feature_flag_request_timeout_seconds` (Ruby). Default 3 s everywhere except RN (10 s).

**Issue:** LLMs struggle with synonyms in general. LLMs may take a round or two to figure out which option is the correct one.

**Suggested solution:** `[SDK]` Settle on one option name and one unit (suffixed with `_ms` or `_seconds`), align the default to 3 s, and keep the deprecated names as aliases.

#### Host option has four names

`[SDK]` `api_host` (JS/React/Next), `host` (Node/Python/Ruby/PHP/RN/iOS/Android/Flutter), `Endpoint` (Go), `HostUrl`/`Host` (.NET), `POSTHOG_HOST` (Flutter manifest). `host` is dominant — deprecate the rest.

**Issue:** LLMs will often hallucinate and use the wrong option name. It will easily catch it later, but waste 2-3 rounds reading SDK code to find the correct option name. This can get expensive quickly.

**Suggested solution:** `[SDK]` Find a single, common name for the host option. Keep the deprecated options for backwards compatibility.

#### `sendFeatureFlags` vs `sendFeatureFlagEvents` collision

Two unrelated options that sound identical:
- `sendFeatureFlags` / `send_feature_flags`: per-capture, attach matched flags to the event.
- `sendFeatureFlagEvents` / `send_feature_flag_events` / `sendFeatureFlagEvent` (singular on Android/RN/iOS): per-SDK, auto-emit `$feature_flag_called` on flag read.

**Issue:** This is confusing for humans and LLMs alike. LLMs may take a round or two of extra thinking to distinguish between the options.

**Suggested solution:** `[DOCS]` Document the distinction with a side-by-side example on every SDK's config page, and `[SDK]` consider renaming one to remove the collision.

#### `personProfiles` has five encodings
`[DOCS]` `'identified_only'` (JS/RN string), `.identifiedOnly` (Swift), `PersonProfiles.IDENTIFIED_ONLY` (Kotlin), no option on server SDKs (use `$process_person_profile: false` per event). Idiomatic per language — needs one comparison table.

**Issue:** LLMs porting config between SDKs will reuse the JS string form (`'identified_only'`) on Swift or Kotlin. Catchable at compile time, but wastes a round.

**Suggested solution:** `[DOCS]` Add a single cross-SDK comparison table showing the `personProfiles` encoding per SDK and the server-side `$process_person_profile` equivalent.

#### Ruby gem name ≠ require name
`[BOTH]` `gem 'posthog-ruby'` requires `require 'posthog'`, not `require 'posthog-ruby'`. Either alias the require or put the right line in the quickstart.

**Issue:** LLMs commonly write `require 'posthog-ruby'` to match the gem name. Loud error, but a near-universal first-attempt trip.

**Suggested solution:** `[BOTH]` Alias `require 'posthog-ruby'` to `require 'posthog'` and update the Ruby quickstart to show the correct line.

#### `evaluation_contexts` / `evaluation_environments` both active
Legacy name works across all SDKs and the `/flags` API. Autocomplete surfaces both. Either set a deprecation flag or document the alias.

**Issue:** Both names work, so LLMs pick whichever matches their training data. No immediate failure, but generated code is inconsistent across a codebase.

**Suggested solution:** `[DOCS]` Pick one name as canonical, document the alias, and set a removal date for the other.

#### snake_case and camelCase mixed within a single SDK

`[SDK]` `posthog-js` ships `get_distinct_id`, `set_config`, `opt_out_capturing` next to `getFeatureFlag`, `isFeatureEnabled`, `captureException`, `startSessionRecording`. React Native inherits both and adds its own.

**Issue:** Low severity, but sometimes LLMs will hallucinate and use the wrong option name. It will easily catch it later, but waste 2-3 rounds reading SDK code to find the correct option name. This can get expensive quickly.

**Suggested solution:** `[SDK]` (very low priority) Avoid mixing casing within a single SDK.

---

### Capture call shapes

#### Python `capture` treats first positional arg as `distinct_id`
`posthog.capture('event_name', {...})` silently becomes `distinct_id='event_name'` with the props dict as the event name. No error.

**Issue:** Many silent failures. The code appears to work, no exception is raised, but every event is sometimes malformed (event name stored as distinct_id). LLMs have no signal to correct themselves unless the user notices downstream.

**Suggested solution:** `[DOCS]` Call this out explicitly in the Python quickstart, and emit a warning when the first positional arg looks like an event name (contains dots, starts with `$`, etc) `[SDK]` I wonder if we can standardize the shape of the server SDKs to either all use context for identity or share the same shape for distinct_id first? Maybe named objects as the expected capture call shape.

#### Person properties have three incompatible shapes
- `$set` / `$set_once` inside `properties` (JS/Node/Python/Ruby/PHP/Go/Flutter/.NET)
- Top-level `userProperties` / `userPropertiesSetOnce` args (Android/iOS/RN)
- `personPropertiesToSet` / `personPropertiesToSetOnce` (.NET `IdentifyAsync`)

Porting JS `{ $set: {...} }` to iOS silently makes `$set` a regular event property.

**Issue:** LLMs translating a JS `$set` snippet to iOS/Android/RN silently produce wrong data — `$set` becomes a regular event property and person properties never update. No error at call site. LLMs see a ton of JS examples and sometimes will infer the wrong shape.

**Suggested solution:** `[DOCS]` Add a cross-SDK comparison table for setting person properties, and call out the iOS/Android/RN top-level args explicitly.

#### Ruby `capture_exception` is positional while `capture` is hash-based
`[SDK]` `client.capture_exception(exception, distinct_id, additional_properties)`. Passing `distinct_id:` as a kwarg silently binds it into `additional_properties`. Align on hash args like `capture`.

**Issue:** If the LLM passes `distinct_id:` as a kwarg (natural given `capture`'s hash shape), Ruby silently folds it into `additional_properties`. No error; events attribute to the wrong user.

**Suggested solution:** `[SDK]` Align `capture_exception` on hash args like `capture`, and warn when kwargs bind into `additional_properties` unexpectedly.

#### PHP uses camelCase keys inside PHP arrays
`PostHog::capture(['distinctId' => ..., 'event' => ..., 'properties' => [...]])`. Not `distinct_id`. Unintuitive for PHP devs.

**Issue:** LLMs default to snake_case (`distinct_id`, matching the rest of the ecosystem); PHP silently drops those keys. No error; events look fine but distinct_id is missing.

**Suggested solution:** `[DOCS]` Document the camelCase keys explicitly on the PHP commandments and quickstart docs. Maybe warn when snake_case keys are used.

#### Python has no `identify()` method
`[SDK]` Every other SDK does. `posthog.identify(...)` raises `AttributeError`. Use `set()` or `identify_context()` inside `new_context()`.

**Issue:** LLMs assume `posthog.identify(...)` exists because every other SDK has it. Python raises `AttributeError` — loud, catchable, but wastes a round.

**Suggested solution:** `[SDK]` Add a `posthog.identify()` method to the Python SDK that wraps `set()` / `identify_context()` so the API matches every other SDK.

---

### identify / alias / reset

#### Invalid distinct_ids return 200 OK with a refused-merge warning
`null`, `true`, `"distinct_id"`, `""`, etc.: event ingests attached to one side only, warning logged in the ingestion UI. SDKs should reject at call site.

**Issue:** LLMs generating `distinct_id='distinct_id'` (literal placeholder) or `distinct_id=None` get 200 OK with no exception. Maybe at least for the common cases, we can set some warnings.

**Suggested solution:** `[SDK]` Validate and warn on invalid distinct_ids (null, booleans, empty strings, literal `"distinct_id"`) at call site with a clear error.

#### `identify()` means different things client-side vs server-side
Client SDKs: associates all future events with the distinct_id. Server SDKs: updates persons properties only; every capture still needs `distinct_id` explicitly. Scattered in docs; belongs in every server-side quickstart.

Also, calling Identify on the server side DOES NOT establish a stable ID, because it doesn't merge the anonymous ID with a stable ID (one doesn't exist on the server). Identify actually works exactly like alias, it seems, on the server.

**Issue:** LLMs reuse client-side identify patterns on the server — one `identify()` call, then bare `capture()`s. Server-side, distinct_id is required per capture; events silently attribute to the wrong user or get dropped.

**Suggested solution:** Maybe we should all use the identify_context pattern like python everywhere.

#### `alias()` argument order and shape varies
- JS: `alias(newAlias, existingId)`
- Node/Ruby: `{ distinctId, alias }`
- Python: `alias(previous_id=newAlias, distinct_id=existingId)` — `previous_id` exists only here
- Go: `Alias{ DistinctId, Alias }` struct
- Mobile: `alias(newAlias)`, current identity implicit

**Issue:** Argument order inverts between JS (`newAlias, existingId`) and Python's explicit form. LLMs mixing SDKs will merge the wrong direction and can irreversibly join two unrelated users.

**Suggested solution:** Maybe we should standardize this :P

#### `reset(true)` vs `reset()` scope differs per SDK
Web: `true` also resets device_id. RN: `reset()` clears feature-flag cache. No table.

**Issue:** LLMs assume `reset()` is symmetrical across SDKs. Porting logout flows misses that web needs `reset(true)` to clear device_id or that RN also purges the flag cache. Hard to catch without behavioral testing.

**Suggested solution:** `[DOCS]` Add a comparison table showing exactly what `reset()` and `reset(true)` clear on each SDK.

#### `$merge_dangerously` is the only escape for two already-identified users
`[DOCS]` Irreversible. Not a first-class SDK method. Needs a dedicated troubleshooting page.

**Issue:** Not a first-class method, so LLMs rarely know it exists. Asked to merge two identified users, they'll propose `identify()` or `alias()` and silently fail.

**Suggested solution:** `[DOCS]` Add a dedicated troubleshooting page for recovering already-identified users, including the `$merge_dangerously` caveats. Maybe we should also rethink how merging really works. It's super confusing to also know which users are truly already identified.

---

### Defaults that differ silently

#### `flushInterval` unit varies
 Node/RN: ms (default 10000). Python: seconds (default 0.5). iOS: seconds (default 30). Android: `flushIntervalSeconds` (30). Go: `time.Duration`. Pasting `flushInterval: 30` between SDKs changes the rate 1000×. Suffix the name with the unit (Android does).

**Issue:** Copy-pasting `flushInterval: 30` from Node to Python changes the flush rate by 1000×. No warning; the code runs. LLMs pattern-match on option names and miss the unit sometimes, resulting in difficult to debug issues.

**Suggested solution:** `[SDK]` Suffix the option name with its unit (`flushIntervalMs` / `flushIntervalSeconds`) everywhere, and keep the legacy names as aliases.

#### Autocapture defaults diverge on mobile
`[DOCS]` iOS: lifecycle + screen views on, `captureElementInteractions` off. RN: `PostHogProvider` needs `autocapture` passed explicitly. Android: lifecycle + screens on. Web: everything on.

**Issue:** LLMs assume autocapture is either "on" or "off" across platforms. The per-platform subfeature matrix is not intuitive; generated code over- or under-captures silently.

**Suggested solution:** `[DOCS]` Add a cross-SDK autocapture-defaults table showing which subfeatures are on by default per platform.

---

### Feature flags

#### `$feature_flag_called` emission rules are non-obvious
`[DOCS]` Emits: `isFeatureEnabled`, `getFeatureFlag`, `getFeatureFlagResult`, `useFeatureFlagEnabled`, `useFeatureFlagVariantKey`. Does not emit: `getFeatureFlagPayload`, `useFeatureFlagPayload`, `useActiveFeatureFlags`, `getAllFlags`, surveys using flags. Using payload-only on an experiment = zero exposure data.

**Issue:** LLMs writing experiment code tend to reach for `getFeatureFlagPayload` because "payload" sounds most specific. That method doesn't emit the exposure event, so the experiment has zero data. Silent and severe.

**Suggested solution:** `[DOCS]` Document the exact set of methods that emit `$feature_flag_called` on each flag/experiment reference page and call out the experiment-exposure implication.

#### `useFeatureFlag` (RN) returns `undefined` during load; React returns `false`
`[DOCS]` Same-looking code, different branches. RN has three states (`undefined`/`false`/`true`), React has two.

**Issue:** LLMs port `if (flag) {...}` from React to RN. In RN, `undefined` is a third state and the loading branch silently takes the `false` path. Surfaces as flicker or wrong default treatment.

**Suggested solution:** ?

#### Mobile feature-flag cache has no TTL
`[SDK]` RN/iOS/Android keep cached values until a successful `/flags` call. A user dormant for a year sees year-old values on reopen. Add a TTL or background-refresh.

**Issue:** Not a code-time trip, but LLMs writing re-engagement flows won't know to invalidate the flag cache on app resume. Users dormant long enough see stale variants.

**Suggested solution:** `[SDK]` Add a configurable TTL (with a sensible default) and a background-refresh option for cached flags on mobile SDKs.

---

### Opt-in / opt-out

#### Four parallel APIs for consent

- `opt_out_capturing()` / `opt_in_capturing()` / `has_opted_out_capturing()` (JS/RN legacy)
- `optOut()` / `optIn()` / `optedOut` property (RN)
- `optOut()` / `optIn()` / `isOptOut()` (iOS/Android)
- `disabled` (Python/Node)
- `defaultOptIn: false` and `opt_out_capturing_by_default: true` both valid on RN init

**Issue:** LLMs hallucinate across the four names (`opt_out_capturing`, `optOut`, `disabled`, `defaultOptIn`). Each SDK has one canonical form; wrong picks cause loud errors or silent no-ops.

**Suggested solution:** `[SDK]` Pick one canonical API name per SDK (idiomatic to the language), alias the rest, and document the mapping in a single cross-SDK table.

---

### Autocapture / session replay

#### `before_send` coverage is uneven
`[SDK]` Present on web, iOS, RN, Flutter. Absent on Node, Python, Ruby, PHP, Go, .NET. Pre-send PII redaction is impossible with a hook in half the SDKs.

**Issue:** LLMs writing PII redaction on Python or Ruby backends assume `before_send` exists because it's documented on the web SDK. The generated hook fails at runtime with no good fallback pattern.

**Suggested solution:** `[SDK]` Add a `before_send` hook to the server SDKs (Node, Python, Ruby, PHP, Go, .NET) so pre-send PII redaction is possible everywhere.

#### `ph-no-capture` attachment varies by platform
`[BOTH]` Web: CSS class. iOS: `accessibilityLabel`/`accessibilityIdentifier`. Android: `tag`/`contentDescription`. RN: prop. Flutter: widget prop. Side effect: also disables autocapture on that element.

**Issue:** LLMs writing cross-platform components use the web `ph-no-capture` CSS class on native elements. Silently does nothing.

**Suggested solution:** `[BOTH]` Add a cross-platform table showing the `ph-no-capture` equivalent for each SDK, including the autocapture side-effect. Maybe we should just standardize this everywhere.

#### Screenshot mode option has different names per platform
`[SDK]` Android: `screenshot`. iOS: `screenshotMode`. RN: nested under `sessionReplayConfig`. Align.

**Issue:** LLMs mixing iOS/Android/RN config use the wrong option name per platform. Loud error, but wastes a round.

**Suggested solution:** `[SDK]` Align on one name (e.g. `sessionReplayConfig.screenshot`) across mobile SDKs, and keep the old names as aliases.

---

### Serverless and shutdown

#### Short-lived processes drop events without explicit flush
`[DOCS]` Python/Ruby CLIs, AWS Lambda, Vercel Functions, FastAPI shutdown — all lose in-flight events without `shutdown()` / `flush()` / `Close()`. `captureImmediate` exists only on Node. Needs one "serverless setup" page covering every SDK.

**Issue:** LLMs writing Lambda or cron-style scripts call `capture()` and exit. Events buffer, the process dies, events disappear. No error — a major silent data-loss category.

**Suggested solution:** `[DOCS]` Add a single "serverless setup" page covering `shutdown()` / `flush()` / `Close()` / `captureImmediate` for every SDK. Also commandments for LLMs to read for SDKs. Maybe have a standard shutdown pattern for all SDKs.

---

### Framework integration

#### Next.js: `PostHogProvider` + `instrumentation-client.ts` creates two instances
`[BOTH]` Singleton initialized in `instrumentation-client.ts` vs. provider-wrapped instance are independent. `posthog.capture(...)` goes to one, hooks to the other. Session replay fragments. Flag hooks already fall through to the singleton — the provider is redundant for most apps.

**Issue:** LLMs combine two common quickstart patterns (`instrumentation-client.ts` + `PostHogProvider`) and silently create two SDK instances. Session replay fragments; `posthog.capture(...)` and hooks hit different instances. Very hard to notice.

**Suggested solution:** (? I don't really know but maybe) `[BOTH]` Make `PostHogProvider` detect and reuse the `instrumentation-client.ts` singleton, and update the Next.js quickstart to show one instance only. 

#### Rails `posthog_distinct_id` falls through to numeric `id`
`[BOTH]` The gem tries `posthog_distinct_id`, `distinct_id`, `id`, `pk`, `uuid` in order. Frontend identifies as `email`, backend events get the integer `id`, nothing correlates. Either warn when fallback looks numeric, or require `posthog_distinct_id` explicitly.

**Issue:** LLMs accept the default fallback order and end up with integer `id` as distinct_id while the frontend uses `email` or a UUID. Events look fine; **nothing correlates across sessions** which is hard to debug.

**Suggested solution:** `[BOTH]` Warn when the fallback resolves to a numeric `id`, or require `posthog_distinct_id` to be set explicitly.

#### Rails vs plain Ruby have different surfaces
`[DOCS]` `PostHog::Client.new(...)` (instance) vs. `PostHog.capture(...)` (class-level through `posthog-rails`). Creating a `PostHog::Client.new` inside a Rails app bypasses `PostHog::Rails.configure` — `auto_capture_exceptions` silently lost.

**Issue:** LLMs cherry-pick `PostHog::Client.new(...)` examples inside a Rails app, bypassing `PostHog::Rails.configure`. `auto_capture_exceptions` and middleware hooks silently stop working.

**Suggested solution:** `[DOCS]` Document the Rails-vs-plain-Ruby difference and warn that `PostHog::Client.new` inside Rails bypasses `PostHog::Rails.configure`.

#### React Native: `PostHogProvider` must be inside `NavigationContainer`
`[DOCS]` Counter to the "providers at the top" convention. React Navigation v7 hooks don't fire otherwise.

**Issue:** "Providers at the top" is strong convention. LLMs default to placing `PostHogProvider` above `NavigationContainer`; screen-tracking hooks then silently no-op.

**Suggested solution:** `[DOCS]` Call this out explicitly on the RN + React Navigation integration page with a working example? Maybe we could find a better pattern for RN.

#### posthog-js is browser-only, breaks SSR
`[DOCS]` Imports from `layout.tsx`, `+layout.svelte`, or SSR Astro pages evaluate on the server; `window` is undefined. Needs a generic "SSR safety" note.

**Issue:** LLMs import `posthog-js` at the top of `layout.tsx` or `+layout.svelte`. `window` is undefined on the server; build or first render fails.

**Suggested solution:** `[DOCS]` Add a generic "SSR safety" note covering Next.js, SvelteKit, and Astro, with the dynamic-import / `typeof window` patterns.

#### `@posthog/react`: `usePostHog()` can return `undefined`
`[DOCS]` Provider renders children before init. Optional-chain everywhere or gate on the value.

**Issue:** LLMs assume `usePostHog()` always returns a client. First-render guards are missing; null-pointer errors when a hook runs before the provider initializes.

**Suggested solution:** `[DOCS]` Document the optional-chain / null-guard pattern on the React integration page and show it in every hook example.

#### Django: custom middleware on top of `PosthogContextMiddleware` double-tags
The SDK already ships middleware. Initialize in `AppConfig.ready()`, not `settings.py`.

**Issue:** LLMs write a custom request-tagging middleware on top of `PosthogContextMiddleware`. Events are double-tagged. Not immediately obvious in dashboards.

**Suggested solution:** `[DOCS]` Document that the SDK ships middleware and that init belongs in `AppConfig.ready()`, not `settings.py`.

#### Rails has no context middleware
`[SDK]` Django's `PosthogContextMiddleware` reads session/distinct tracing headers. `posthog-rails` has no equivalent. Rails devs porting from Django end up with uncorrelated frontend/backend events.

**Issue:** Django porters look for a Rails equivalent and find nothing. Not silent — LLMs notice the absence — but they produce subpar generated code without any context propagation.

**Suggested solution:** `[SDK]` Add a `posthog-rails` context middleware equivalent to Django's `PosthogContextMiddleware`.

---

### Hosts, keys, ingest

#### US/EU residency mismatch is a silent 200
`[SDK]` Sending a US-project token to the EU host returns 200 with no data in the project. Validate the token's region against `host` on init.

**Issue:** LLMs guess the host from docs examples without checking the token's region. Events return 200 OK and vanish. No feedback signal at all.

**Suggested solution:** `[SDK]` Validate the project token's region against `host` on init and log a clear error on mismatch. (Is this possible???) Maybe tokens should have a region encoded? idk....
