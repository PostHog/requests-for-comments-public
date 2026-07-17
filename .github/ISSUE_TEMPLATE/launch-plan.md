---
name: 'Launch Plan'
about: For planning product releases and marketing launches.
title: '[Launch Plan] <PRODUCT NAME>'
labels: marketing
assignees:
    - PostHog/team-billing
    - PostHog/team-marketing
    - PostHog/team-growth
---

[PRODUCT NAME] is **being released** on **[MONTH AND DAY]**

[PRODUCT NAME] is **being launched** on **[MONTH AND DAY]**

> For the distinction between releases and launches, see [releases vs. launches](https://posthog.com/handbook/marketing/product-announcements).

Tag relevant owners here:

Product Marketer (PMM): [@handle]
Team lead (TL): [@handle]
Product Manager (PM): [@handle]
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

## Release plan (primary target: existing users)

_Optional to fill out. Owned by team lead or PM, see guidance [here](https://posthog.com/handbook/product/releasing-new-products-and-features)_

-   [ ] Create Slack channel **#<product>-launch** with all owners — _TL_, _PM_ or _PMM_
-   [ ] If needed, create a taskforce for setting a release _and_ launch date. This should be limited to the PMM, PM, and product Team Lead. - _TL_, _PM_ or _PMM_
-   [ ] Create Slack canvas with a [release checklist](https://posthog.com/handbook/product/releasing-new-products-and-features#releases), incl. rollout plan — _TL_ or _PM_
-   [ ] Finalize pricing RFC — _TL_ or _PM_ ([template](https://github.com/PostHog/billing/blob/main/notes/pricing-rfc.md))
-   [ ] Confirm docs are ready — _Docs team_
-   [ ] Define [product intent](https://posthog.com/handbook/growth/growth-engineering/product-intents) and [activation criteria](https://posthog.com/handbook/growth/growth-engineering/per-product-activation) - _Team lead & PMM_
-   [ ] Decide beta reward — _TL_, _PM_ & _PMM_

### Release day

-   [ ] Migrate organizations to new pricing plans — @PostHog/team-billing
-   [ ] Roll out feature flag according to the rollout plan — _Team lead_
-   [ ] Email existing users according to the rollout plan — _PMM_
-   [ ] Disable beta feedback emails — _PMM_
-   [ ] Create a dedicated channel to centralize support — _Support team_
-   [ ] Release in-app onboarding — _Team lead_

### Follow-on

-   [ ] Remove early-access feature & feature flags — _Team lead_
-   [ ] Funnel Zendesk tickets to created Slack channel — _PMM_
-   [ ] Continue to roll out feature flag according to the rollout plan — _Team lead_
-   [ ] Continue to email existing users according to the rollout plan — _PMM_

#### Billing changes

-   [ ] Set up plans in Stripe — @PostHog/team-billing
-   [ ] Create plans in billing — _team lead_ ([example](https://github.com/PostHog/billing/pull/1186))
-   [ ] Usage report PR — _team lead_ ([example](https://github.com/PostHog/posthog/pull/28313))
-   [ ] Quota limiting PR — _team lead_ ([example](https://github.com/PostHog/posthog/pull/30459))
-   [ ] Pricing calculator (site) — _team lead_ ([example](https://github.com/PostHog/posthog.com/pull/11143))
-   [ ] Set up product and tax codes on Anrok — @PostHog/team-people-and-ops
-   [ ] Add product usage/engagement/mrr tracking to Vitally — @PostHog/team-billing

___

## Launch plan (primary target: new users)

_Keep this list current. Link PRs/issues and assign owners._
_PMM should take the lead on adding additional marketing activities_

### Before launch

-   [ ] Find a case-study customer — _PMM_
-   [ ] Draft content and brief website team — _PMM_
-   [ ] Draft content and brief website team — _PMM_
-   [ ] For new products: Add email onboarding content and adjust marketing flows — _PMM_
-   [ ] For new products: Create a sales enablement and add to handbook — _PMM_
-   [ ] For new products: Ensure product is integrated with Wizard — _PMM_

### Launch day

-   [ ] Merge any remaining PRs — _PMM_
-   [ ] Release website product page — @smallbrownbike
-   [ ] Make homepage and nav updates — @smallbrownbike
-   [ ] Publish announcement on blog if needed — _PMM_
-   [ ] Also publish the blogpost on X as a native article — _PMM_
-   [ ] Internal announcement for Sales & Ads teams — _PMM_
-   [ ] Ask `#team-growth` to add an ad inside the app to let people know about the product launch — _PMM_
    -   You should write a custom and personal copy
    -   You can decide whether everyone gets the product ad or only a set of users (list of emails)

### Follow-on

-   [ ] If this is a [new product](https://posthog.com/handbook/marketing/product-announcements#new-product-announcements), create and upload a sales enablement doc to [the enablement repo](https://github.com/PostHog/sales-enablement) - _PMM_

## Feedback & ideas

Marketing feedback and extra ideas welcome. Drop anything “out-there” below.
