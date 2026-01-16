# Automating docs contributions with AI

> **Decision:** Run pilot with Inkeep Content Writer (February 2026)

## The problem

One of our Q1 goals is to [10x docs contributors](https://posthog.com/teams/docs-wizard). As PostHog scales, product docs **need** to be self-serve. We need to create processes that facilitate quality docs contributions and help us maintain accuracy.

The current workflow isn't scalable:

1. Engineer opens a PR for a new feature or product
2. It is the responsibility of that engineer to write the first draft of docs. Many times, engineers either don't write docs, or don't get a review from us. This means we either have no docs, or docs that aren't properly reviewed
3. Engineer merges PR, either with or without a docs PR

**Our challenges:**

| Challenge | Impact |
| --------- | ------ |
| 6+ products launching in 2026 | Docs team can't keep pace |
| ~40 small engineering teams | Different shipping cadences, impossible to track |
| Some engineers find writing from scratch a barrier | Docs don't get written |
| Screenshot maintenance at scale | Screenshots go stale, docs lose trust |
| Custom docs stack (React + extended Markdown) | Most tools built for simpler platforms |
| 20,000+ page sitemap | Can overwhelm AI tools |

## What we need

An AI tool that can:

- Detect code changes/PRs that need documentation
- Draft doc updates automatically
- Integrate with our GitHub workflow
- Match our style guide and voice
- Work with our custom docs platform

The ideal workflow:

1. Engineer creates a PR for a new feature or product
2. AI auto-detects and drafts a doc update, tagging docs team for review
3. Engineer reviews the draft for technical accuracy instead of writing it and approves
4. Docs team does a final review and merges

This shifts engineers from authors to technical reviewers/SMEs, a task less daunting for all.

## What we still own (and should)

Regardless of which tool we choose, we should always own:

- Contributor guidelines and processes
- Templates for different doc types
- Analytics on docs health
- Doc structure, navigation, information architecture
- Creating ergonomics for the docs site (UX)
- Markdown service
- Implementing linting and Vale

## Vendors evaluated

We evaluated two vendors: Promptless and Inkeep.

### Promptless

[Promptless](https://promptless.ai/) automates the creation of docs PRs based on code changes. It doesn't write or review code (like Greptile), it writes _docs about code_.

**Key features:**

| Feature | What it does | Why this matters for us |
| ------- | ------------ | ----------------------- |
| Voice match | Custom fine-tuned models matching your docs style guide | Maintains our unique PostHog voice at scale |
| Auto-drafting | AI generates first drafts from code changes | Engineers review instead of write |
| Promptless Capture | AI navigates product, captures screenshots, detects outdated ones | Automated screenshot maintenance |
| GitHub PR/commit triggers | Auto-detect when code changes need doc updates | Catches docs gaps before they become debt |
| Slack Listen | Monitors channels, auto-suggests docs from conversations | Captures domain knowledge from Slack |
| Automate CI fixes | Auto-detects and fixes failed CI checks, linter failures, Vale rules | Reduces manual cleanup |

**Trial (January 9-14, 2026):**

We ran a 30-day PR replay, scoped to `content/docs` only to avoid overwhelming their "product ontology" with our 20,000+ pages.

- Reviewed 58 suggestions from 1,000 PRs/commits
- Generally good quality—most rated 3-5 out of 5 (only spot tested a few)
- Did a decent job at bringing new documentation up to par with our standards

**Strengths:**

- Catches cross-references when features are deprecated
- Finds undocumented features from PR screenshots/context
- Good technical accuracy
- Screenshot automation via Promptless Capture

**Weaknesses:**

- Initial suggestions sometimes lack context before UI steps
- Too targeted, doesn't review whole page holistically, though this setting can be changed
- Doesn't follow our preferred information architecture out of the box (we want all sections to match error tracking docs structure: Getting Started → Concepts → Guides → AI Features → Resources)
- Screenshot feature limited for our monorepo. No preview links, local builds not viable

**Pricing:** $3,000/month for 2,195 md/mdx files. 20% discount for annual + co-marketing brings it to $2,400/month.

### Inkeep

[Inkeep](https://inkeep.com/) offers a Content Writer agent. We already use their RAG integration for search on our website, community portal, and Slack.

**Key features:**

| Feature | What it does | Why this matters for us |
| ------- | ------------ | ----------------------- |
| Multi-agent system | Specialized sub-agents for research, comparison, drafting | More sophisticated content generation |
| Multiple triggers | Slack mentions, Zendesk ticket closures, GitHub merges, API calls | Flexible automation options |
| GitHub-first | Creates draft PRs with human review process | Matches our workflow |
| Style guide integration | Reads style guide from repo + dashboard controls | Maintains voice consistency |
| Custom instructions | Create our own instructions to override | Enforce PostHog patterns |
| Customizable | Extensible for custom integrations | Future flexibility |
| Managed service | Inkeep handles integration, customization, maintenance | Lower burden on our team |

**Evaluation (January 14-16, 2026):**

- Builds on existing integration, already using their RAG
- GitHub-first integration matches our workflow
- Zendesk integration addresses support ticket → docs workflow gap we can solve (bonus!)
- Managed service model, they handle deployment and maintenance, we build!

**Strengths:**

- Existing relationship and integration
- Zendesk trigger for a bonus solve
- Managed service reduces our maintenance burden
- Extensible and customizable agents

**Weaknesses:**

- No auto-screenshot capture, would require custom Playwright + VM setup (I can do this with the custom agent though)
- Haven't tested PR suggestion quality yet, but, that's what the pilot is for

**Pricing:** $2,000/month bundled (includes current RAG + Content Writer agent). We're currently paying $500/month for RAG alone.

## Comparison

| Factor | Promptless | Inkeep |
| ------ | ---------- | ------ |
| Monthly cost | $2,400-3,000 | $2,000 |
| Existing relationship | None | Already using RAG ($500/mo) |
| Screenshot automation | Yes, but limited for our monorepo | No, custom build required |
| PR suggestions quality | Good but too narrow/targeted | TBD in pilot |
| Information architecture | Doesn't follow our patterns | TBD in pilot |
| Integration approach | SaaS platform | Managed service |
| Zendesk integration | No | Yes |
| Slack integration | Yes | Yes |
| GitHub integration | Yes | Yes |

## Decision: Inkeep pilot

**Why Inkeep:**

1. Lower cost ($2,000/mo vs $2,400-3,000/mo) and consolidates with existing RAG spend
2. Builds on existing relationship and integration
3. Managed service model better fits our capacity constraints
4. Zendesk integration addresses support ticket → docs workflow gap
5. Promptless's screenshot feature, its main differentiator, has limited value for our monorepo setup
6. Promptless suggestions were too targeted and didn't follow our information architecture
7. Promptless's product just felt a lot less mature in terms of using the product, and the results

See the [Inkeep pilot RFC] for pilot details, timeline, and approval process.

## Other vendors considered

| Platform | What it does | Why we didn't pursue |
| -------- | ------------ | -------------------- |
| [Dosu](https://dosu.dev/) | Detects merged PRs, finds related docs, auto-updates | Security/SOC 2 status unconfirmed, fewer features |
| [Swimm](https://swimm.io/) | Code-coupled docs with auto-sync | Focused on internal engineering docs, pivoted to legacy code modernization |
| [Qodo](https://www.qodo.ai/) | `/add_docs` command on PRs | Docs is a secondary feature (mainly code review) |

## Build vs buy

**Buy if:**

- Speed to value matters most
- We need something working now, not weeks from now
- Maintenance capacity is limited (as it is right now)

**Build if:**

- We have concerns over data privacy
- We have capacity to dedicate 6-8 weeks
- Our docs workflow proves too complicated for vendors
- SaaS pricing becomes too expensive at scale

**Estimated build effort:**

| Feature | Effort |
| ------- | ------ |
| PR-triggered docs drafting | 2 weeks |
| Screenshot automation | 2-4 weeks |
| Slack integration | 1 week |
| Zendesk integration | 1 week (we have some of this built) |
| **Total** | 6-8 weeks + ongoing maintenance |

For now, buying makes sense—we need speed and have limited capacity. We can revisit building if the vendor doesn't work out or costs become prohibitive.
