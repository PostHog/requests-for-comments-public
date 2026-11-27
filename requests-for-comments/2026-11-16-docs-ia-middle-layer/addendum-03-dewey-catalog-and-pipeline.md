# Addendum 3: Surveys catalog, pins, and compiled skills

Our Surveys experiment indexed 2,548 code symbols across 5 repos, built a coded catalog, generated the SDK feature support matrix, and compiled 15 self-contained skills.

## What the pipeline indexed

- **5 repos + 3 platform seams:** Indexed 2,548 symbols across `posthog`, `posthog-js`, `posthog-ios`, `posthog-android`, and `posthog-flutter`, plus shared `mcp`, `hypercache-server`, and mobile bridge seams.
- **Large-file windowing:** Splitting functions over 500 lines into overlapping 380-line windows lifted recall on large files to 85.5%.
- **Port linking:** Deterministic matching wired 2,229 internal links with 0 LLM calls. A small agent pass mapped 173 cross-team calls onto pinned platform seams.
- **Verified pins:** All 2,548 `<commit>-<md5>` pins verified against source commits.

## Surveys catalog (`SV`)

Chapters (`SV-0..SV-8`), sections (`SV-x.y`), and use cases (`UC-1..UC-15`) group by user feature rather than splitting by SDK. Individual records carry a surface tag (`UI`, `API`, `MCP`, `WEB`, `RN`, `IOS`, `AND`, `FLT`, or `ALL`).

| Entity | Code format | Count |
|---|---|---:|
| Chapters | `SV-0..SV-8` | 9 |
| Capability sections | `SV-x.y` | 32 |
| Use cases (9 from docs/marketing, 6 code gaps) | `UC-1..UC-15` | 15 |
| Prerequisites | `SV-<SURF>-P-*` | 9 |
| 1-surface procedures | `SV-<SURF>-PR-*` | 62 |
| Code-pinned facts | `SV-<SURF>-F-*` | 6,611 |
| Mapped live docs claims | `SV-<SURF>-C-*` | 725 |
| Recorded editorial decisions | `SV-ALL-D-*` | 21 |

### Generated SDK feature matrix

Missing capabilities on an SDK show up as explicit gaps instead of silent omissions. This same data generated `/docs/surveys/sdk-feature-support` and fixed 20 dead links on `master`:

| Capability | Web | React Native | iOS | Android | Flutter |
|---|---|---|---|---|---|
| `SV-0.2` Install and initialize SDK | Yes | Yes | Yes | Yes | Yes |
| `SV-2.1` Question types, rating scales, auto-submit, shuffling | Yes | Yes | Yes | Yes | Yes |
| `SV-2.2` Conditional branching | Yes | Yes | Yes | Yes | Yes |
| `SV-2.3` Survey translations | Yes | Yes | Yes | No | Yes |
| `SV-3.1` URL targeting and audience estimate | Yes | No | No | No | No |
| `SV-3.2` Event and action triggers | Yes | No | Yes | Yes | Yes |
| `SV-3.3` Linked flags, variants, and person properties | Yes | No | Yes | No | No |
| `SV-4.1` Custom colors, fonts, position, branding | Yes | No | No | No | No |
| `SV-4.2` Feedback button widgets and hosted surveys | Yes | No | No | No | No |
| `SV-5.1` Fetch and cache active surveys | Yes | Yes | Yes | No | Yes |
| `SV-5.2` Match users and show popovers | Yes | Yes | Yes | Yes | Yes |
| `SV-5.3` Submit responses and partial answers | Yes | Yes | Yes | Yes | Yes |
| `SV-5.4` Dismissals, page-unload abandonment, SDK reset | Yes | Yes | Yes | Yes | Yes |

## Compiled one-pager skills (`UC-1..UC-15`)

Every use case compiles into one standalone Markdown file, and an automated check verified all 13,638 internal anchor links with 0 broken links. Unsliced all-surface dumps run 4,407 to 9,170 lines, so agents need surface-sliced skills by default.

| Use case | Source | Lines (all surfaces) | Procedures | Facts | Anchor links | Broken |
|---|---|---:|---:|---:|---:|---:|
| `UC-1` Launch first in-app survey | Live docs | 6,056 | 20 | 611 | 757 | 0 |
| `UC-2` Create survey with PostHog AI | Live docs | 4,407 | 13 | 451 | 527 | 0 |
| `UC-3` Conditional branching survey | Live docs | 5,992 | 12 | 614 | 728 | 0 |
| `UC-4` Implement custom survey UI | Live docs | 7,965 | 20 | 797 | 1,242 | 0 |
| `UC-5` Send responses to Slack and webhooks | Live docs | 6,995 | 10 | 711 | 904 | 0 |
| `UC-6` Summarize open-ended responses with AI | Live docs | 5,969 | 7 | 605 | 805 | 0 |
| `UC-7` Manage and query surveys with MCP | Live docs | 5,781 | 8 | 588 | 721 | 0 |
| `UC-8` Recurring NPS with response cap | Marketing | 8,403 | 22 | 849 | 1,136 | 0 |
| `UC-9` Capture partial web responses | Marketing | 9,170 | 27 | 921 | 1,399 | 0 |
| `UC-10` Public `/api/surveys` token and sanitization | Code gap | 5,865 | 8 | 594 | 791 | 0 |
| `UC-11` Draft preview and site snapshot PNG | Code gap | 7,764 | 14 | 787 | 1,073 | 0 |
| `UC-12` Test notification with last response | Code gap | 6,471 | 6 | 661 | 815 | 0 |
| `UC-13` Contextual Feedback tab | Code gap | 6,121 | 13 | 617 | 833 | 0 |
| `UC-14` Trigger survey via Workflow Show Survey | Code gap | 6,625 | 13 | 663 | 1,010 | 0 |
| `UC-15` Duplicate across projects and archive responses | Code gap | 6,689 | 11 | 676 | 897 | 0 |
| **Total** | | | | | **13,638** | **0** |

## Details and stuff

<details>
<summary>How to reproduce (pipeline commands, models, chapter list, and skill verification)</summary>

### Reproduction commands

From `docs-ia-crawl-experiments/typed-ports-pilot/`:

```bash
# 1. Extract facts, wire internal ports (0 LLM), connect platform seams, and assemble procedures
node run-hybrid.mjs --full --concurrency 6 --out out/full-surveys-cold

# 2. Build Dewey catalog, append-only register, SDK matrix, leaderboard bundle, and 15 compiled skills
node build-dewey-ia.mjs \
  --export-dir out/full-surveys-cold/export \
  --docsmap-dir out/docsmap-cold \
  --out out/surveys-dewey-cold

# 3. Verify Dewey code allocator invariants
node test-assign-codes-v2.mjs
```

The generalized workflow version lives in `PostHog/context-mill` draft PRs [`#571`](https://github.com/PostHog/context-mill/pull/571) to [`#575`](https://github.com/PostHog/context-mill/pull/575) (`vincent/docs-ia-5-mcpa-data`, `pnpm docs-ia list`, `pnpm test:docs-ia`).

### Stage models and gates (`run-hybrid.mjs` + `build-dewey-ia.mjs`)

- **Stage 1 (Extract):** Gemini 4 Hydrogen (`medium` effort on `<150`-line batches, `high` effort on `>=150`-line symbols). Functions over `500` lines split into overlapping `380`-line windows (`surveyLogic`, `SurveyEdit`, `SurveyEditQuestionGroup`), raising recall on 150+ line symbols to `85.5%`. Deterministic quote gate rejects any fact whose quote isn't a verbatim line inside the pinned symbol or pins under `contents/docs/`.
- **Stage 2 (Internal port linker, 0 LLM):** Exact normalized port match + HTTP path wildcard match + implied Django `model_hook` linking (`2,229` links).
- **Stage 3 (Platform seam connect):** Gemini 4 Neon (`high` effort) maps candidate boundary facts onto the pinned `platform-*` seam menu (`173` links, `0` refusals).
- **Stage 4 (Procedures + Dewey allocator):** Gemini 4 Neon (`medium` effort) assembles 1-surface procedures citing verified facts, then `lib/assign-codes-v2.mjs` mints permanent surface-scoped IDs (`SV-<SURF>-{P,F,PR,D,C}-<ID>`, 4-digit padding on `F`/`C` and 2-digit on `P`/`PR`/`D`).

### Catalog breakdown and link checks

- **9 Surveys chapters (`SV-0..SV-8`):** `SV-0` Prerequisites (3 sections), `SV-1` Creating and managing surveys (5), `SV-2` Questions, branching, and translations (3), `SV-3` Targeting, triggers, and schedules (5), `SV-4` Appearance, branding, and hosted surveys (2), `SV-5` Showing surveys and collecting responses (4), `SV-6` Notifications and workflows (4), `SV-7` Results, exports, and AI analysis (3), `SV-8` REST API, public delivery, and MCP (3).
- **Fact filtering:** Out of `6,611` runtime facts, `18` flag-gated beta facts were tagged for exclusion and `66` UI-chrome facts were filtered out. `52` quarantined `sdk-specs` prose facts were tracked separately.
- **Anchor validator:** Checked every Markdown anchor and in-drawer jump target across all 15 compiled skills (`13,638` links, `0` broken, `51/51` unit tests passing in the pilot validator).

</details>
