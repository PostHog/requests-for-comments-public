---
name: 'Launch Plan'
about: For planning marketing launches and product announcements.
title: '[Launch Plan] <PRODUCT NAME>'
labels: marketing
assignees:
    - PostHog/team-billing
    - PostHog/team-marketing
    - PostHog/team-growth
---

[PRODUCT NAME] is launching on [MONTH AND DAY]

Tag relevant owners here:

Team lead: [@handle]
Product Marketer (PMM): [@handle]
Billing Lead: [@handle]
Blitzscale: [@handle]

## Marking best practices

-   Have ≥1 customer story within 3 weeks of launch
-   Publish a best-practices guide and link it from docs
-   Ship at least one ready-to-use template (or similar)
-   Publish at least one tutorial at launch
-   Complete robust docs (and product page if needed)
-   Add to email + in-app onboarding flows
-   For new products, create a sales enablement sheet

<!-- PMM is the default fallback for marketing tasks if a specific PMM isn’t assigned. -->

## Launch plan

_Keep this list current. Link PRs/issues and assign owners._
_PMM should take the lead on adding additional marketing activities_

### Before launch

-   [ ] Finalize pricing RFC — _Team lead_ ([template](https://github.com/PostHog/billing/blob/main/notes/pricing-rfc.md))
-   [ ] Create Slack channel **#<product>-launch** with all owners — _PMM_
-   [ ] Find a case-study customer — _PMM_
-   [ ] Draft content and brief website team - _PMM_
-   [ ] Confirm docs are ready — _Docs team_
-   [ ] Define [product intent](https://posthog.com/handbook/growth/growth-engineering/product-intents) and [activation criteria](https://posthog.com/handbook/growth/growth-engineering/per-product-activation) - _Team lead & PMM_
-   [ ] Draft content and brief website team - _PMM_
-   [ ] Decide beta reward — _Team lead & PMM_
-   [ ] For new products: Add email onboarding content and adjust marketing flows - _PMM_
-   [ ] For new products: Create a sales enablement docs and add it to the handbook (example) - _PMM_

### Launch day

-   [ ] Merge any remaining PRs — _PMM_
-   [ ] Enable feature flag for all users — _Team lead_
-   [ ] Disable beta feedback emails — _PMM_
-   [ ] Create a dedicated channel to centralize support - _PMM_
-   [ ] Release website product page — @corywatilo
-   [ ] Make homepage and nav updates — @corywatilo
-   [ ] Release in-app onboarding — _Team lead_
-   [ ] Internal announcement for Sales & Ads teams - _PMM_
-   [ ] Ask `#team-growth` to add an ad inside the app to let people know about the product launch - _PMM_
    -   You should write a custom and personal copy
    -   You can decide whether everyone gets the product ad or only a set of users (list of emails)

### Follow-on

-   [ ] Remove early-access feature & feature flags — _Team lead_
-   [ ] Migrate users and delete beta plans — @PostHog/team-billing
-   [ ] Funnel Zendesk tickets to created Slack channel — _PMM_

#### Billing changes

-   [ ] Set up plans in Stripe — @PostHog/team-billing
-   [ ] Create plans in billing — _team lead_ ([example](https://github.com/PostHog/billing/pull/1186))
-   [ ] Usage report PR — _team lead_ ([example](https://github.com/PostHog/posthog/pull/28313))
-   [ ] Quota limiting PR — _team lead_ ([example](https://github.com/PostHog/posthog/pull/30459))
-   [ ] Pricing calculator (site) — _team lead_ ([example](https://github.com/PostHog/posthog.com/pull/11143))

## Feedback & ideas

Marketing feedback and extra ideas welcome. Drop anything “out-there” below.
