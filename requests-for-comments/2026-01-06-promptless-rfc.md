# Using Promptless to help 10x docs contributors

## What Promptless does

[Promptless](https://promptless.ai/) automates the creation of docs PRs based on code changes. It doesn't write or review code (like Greptile), it writes _docs about code_.

It works by detecting code changes, PRs, or team discussions that require documentation updates (sourced from GitHub, Slack, etc.), then opens a docs PR with drafted updates. It eliminates manual overhead of maintaining docs and ensures customers have accurate, up-to-date docs.

## Capabilities

Promptless has a lot of features. I listed ones that may be useful for us:

| Feature                   | What it does                                                                                      | Why this matters for us                                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Voice match               | Custom fine-tuned models matching your docs style guide                                           | Maintains our unique PostHog voice at scale                                                                                     |
| Auto-drafting             | AI generates first drafts from code changes                                                       | Engineers review instead of write, reducing barrier to entry and scale burden                                                   |
| Citation transparency     | Show exact sources (files, commits, convos) for each draft                                        | Can verify sources when reviewing the AI generated content                                                                      |
| Custom instructions       | Custom generation rules                                                                           | Can enforce PostHog specific patterns (templates, style rules, etc.)                                                            |
| Changelog publishing      | Auto-creates changelog entries when features ship                                                 | Changelog takes a lot of time already. We could merge our changelog automations in Promptless for a unified management solution |
| Context assembly          | Pulls info from GitHub for context (for example, issues)                                          | No more hunting through tickets/threads/etc. for context                                                                        |
| Promptless Capture        | AI agents navigates product, captures screenshots, detects outdated ones, regenerates screenshots | We can have the best of both worlds: screenshots AND accurate ones at that                                                      |
| Screenshot editor         | Built-in editor to crop, annotate, highlight, add text/shapes/arrows                              | Keeps screenshot workflow in one place                                                                                          |
| Slack image support       | Add screenshots from Slack messages to docs                                                       | Could potentially use screenshots from changelog Slack messages                                                                 |
| GitHub PR/commit triggers | Auto-detect when code changes need doc updates                                                    | Catches docs gaps before they become debt, keeping us proactive                                                                 |
| Trigger refinements       | Filter which repos or directories trigger                                                         | Control noise and trigger only on relevant changes                                                                              |
| Slack Listen              | Monitors channels, auto-suggests docs from conversations                                          | Captures domain knowledge that lives in Slack                                                                                   |
| Automate CI fixes         | Auto-detects and fixes failed CI checks, linter failures, Vale rules                              | Reduces manual cleanup, which is huge for contributors (and often why contributors don’t follow through with PRs)               |

## Why now?

One of our Q1 goals is to [10x docs contributors](https://posthog.com/teams/docs-wizard). As PostHog scales, product docs **need** to be self-serve. To complete this goal, there's a lot of moving parts. We need to create processes that facilitate quality docs contributions and help us maintain the accuracy of published docs.

Promptless can help us reduce barrier to entry for docs contributors, increase contributor rate and quality, and automate screenshot maintenance.

## Features

Promptless has features that can solve some of our biggest docs problems. I’d estimate it can get us 40-50% of the way for 10x-ing docs contributors:

| Challenge                               | How Promptless helps                                                                                       |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 6+ products launching in 2026 (or more) | Auto-detects when new features need docs, drafts content proactively                                       |
| 200+ engineers shipping code at scale   | Reduces engineering burden and barrier to entry, they can review AI drafts instead of writing from scratch |
| Screenshot maintenance at scale         | Automate screenshot capture/updates as UI changes                                                          |
| Docs debt                               | Catches gaps before they accumulate, triggers on relevant PRs                                              |
| Inconsistent voice and style            | Voice match maintains PostHog’s docs style guide consistently                                              |

## What we still own (and should)

Promptless empowers us to move fast and maintain the guardrails. We are free to focus on strategic decisions, which are decisions we should always own:

- Contributor guidelines and processes
- Templates for different doc types
- Analytics on docs health
- Doc structure, navigation
- Creating ergonomics for the docs site (UX)
- Markdown service
- Implementing linting and Vale

## What would the new workflow look like?

Before going over the new workflow, I'll highlight the issues with the current workflow:

1. Engineer opens a PR for a new feature or product
2. It is the responsibility of that engineer to write the first draft of docs, and to ask the docs team for a review. Many times, engineers either don’t write docs, or don’t get a review from us - this means we either have no docs, or docs that aren’t properly reviewed
3. Engineer merges PR, either with or without a docs PR

With Promptless, the new workflow could be:

1. Engineer creates a PR for a new feature or product
2. Promptless auto-detects and drafts a doc update in the posthog.com repo, tagging the docs team for review
3. Engineer reviews the draft for technical accuracy (instead of writing it) and approves
4. Docs team does a final review and merges

## How much is this going to cost?

From what I could find, Promptless charges by documentation page count, not headcount. This works as companies scale, but we have a lot of docs.

But, there could be ROI here. I am just going to roughly estimate across docs/eng teams:

- Screenshot maintenance: 5-10 hours/week (if we actually kept up with them!)
- Docs drafting: 15-20 hours/week
- Context gathering: 3-5 hours/week
- **Total:** ~25-35 hours/week saved

We need to know the actual price for our current number of docs pages: 882 MDX files. We might be on the higher end of pricing tiers.

To figure out if it’s worth the $$$$, I am going to need info to fill out this formula:

At a loaded cost of $Y/hour for docs team time, we’re saving $Z/month in labor. If Promptless costs less than $Z/month, it’s a ROI-positive on time savings alone.

## My honest assessment

Promptless does some things well:

- Screenshot validation is needed, and quickly. This is a debt area that keeps me up at night.
- Docs drafting could help us scale better. The AI does the first pass, so Engineers don’t have to, and they can move quickly.
- Context assembly from GitHub/Slack/Google is really useful. Saves time hunting for context.

My biggest concerns are:

- **Will this reduce engineer contribution or just shift the work?** The best case is that Engineers review AI drafts, catch errors, and approve them. We get drafts that are 80% of the way. The worst case is Engineers rubber-stamp AI drafts without reading them and we inherit a massive backlog of unchecked docs to review.
- **Our docs site is uniquely complex.** Our docs site lives in a giant website repo, has tons of custom components, unique styling, and specific patterns. Promptless seems to be aimed at people using rather simple platforms, like Fern or Mintlify. Promptless was built for the majority of docs sites, and we are the minority. The risk is that we push Promptless to its limits and don’t get the full value of the platform.
- **Is this a short term solution or a long-term platform?** Promptless is definitely an accelerant. It multiplies our output but it doesn’t provide the foundation, all that work still comes from us (and it should!!!) It’s missing features we’ll eventually need if we want a pulse on the comprehensive health of our docs.
- **Its docs testing features are lacking.** It can update screenshots, but how does it validate code snippet syntax and usability? How does it test CLI commands? If we want a holistic overview of our docs health, we need to build these tools (or they need to build them).

## My take

For now, it’s a promising tool. We need speed in 2026, and it supports the scale we are facing so we can focus on foundational projects. Once we have nailed these foundations in 2026, we can discuss what bandwidth we have to build more features in house.

## Next steps

We have a demo scheduled for Friday January 9th. I have a list of questions I need to ask!

- How does Promptless Capture work for authenticated applications?
- What's the review workflow for captured/regenerated screenshots?
- How extensive are the customization capabilities? For example, if we have a library of templates, style guide, etc. can Promptless use this to generate all its drafts?
- How is voice match trained?
- How does this work with custom docs platforms?
- What’s on the roadmap for more docs testing/docs maintenance features? Specifically for code snippet validation (syntax, functionality, etc.) and SDK/API references?
- What's included in a custom plan? What's the actual cost for 842 pages?

Then, I suggest we sign up for their free trial and run a pilot with an Engineering team. Their free trial gives us access to Promptless for 200 pages. We should test:

- Customizing Promptless with our style guide and docs site customizations
- Implement Promptless for an Engineering team, and have them give use feedback on whether Promptless's first drafts are technically sound, and if they like the new workflow
- Test screenshot capture and push it to its limits
