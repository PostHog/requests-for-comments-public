# Request for comments: 10x-ing PostHog's engineering onboarding

**Decision maker:** Engineering team leads + the blitzscale folks

## Overview

Our engineering onboarding is inconsistent and doesn't scale. Teams handle it in isolation, creating wildly different experiences that make it impossible to assess performance or teach our ways of working effectively.

This RFC proposes cohort-based onboarding (HogBoarding) that standardizes the experience while building stronger cross-team connections.

## Why now

1. **We're hiring rapidly** and inconsistent onboarding amplifies all the problems
2. **Recent onboarding failures** - several new hires haven't ramped effectively, leading to longer time-to-productivity and people leaving during probation
3. **Internal poaching is breaking down** - we've moved people from overstaffed teams (Feature Flags) without communicating expectations upfront, creating confusion about hiring plans

## Problem

Current onboarding has too much variance in quality, operates in silos, doesn't teach the PostHog way effectively, creates an "onboarding tax" on high-performing teams, and makes inefficient use of our technical debt.

**More details:**

1. **Variance in quality.** Some teams have well-thought-out processes, others wing it. New hires get wildly different experiences depending on which team they join, making it impossible to assess performance consistently or ensure everyone learns our core ways of working.

2. **Teams operate in silos.** New engineers only meet their immediate team members during their first weeks. This limits cross-team collaboration and means people miss out on learning from other parts of the product and codebase.

3. **Not teaching the PostHog way effectively.** Our core values around autonomy, ownership, and development practices are best learned through experience, but we're trying to teach them within the narrow context of a single product area.

4. **"Onboarding tax" on high-performing teams.** Teams with strong onboarding culture (e.g. Feature Flags) end up overhiring, expecting that ramped-up engineers will get poached by teams that need them. This creates weird expectations—how do teams know they're supposed to overhire? It also unfairly burdens teams that invested in good onboarding processes.

5. **Inefficient use of starter tasks.** We have technical debt and starter tasks across all products, but currently these only get tackled by people on those specific teams. We're missing an opportunity to pay down this debt while giving new hires diverse learning experiences.

6. **Scheduling headache for offsites.** Coordinating onboarding offsites is a logistical pain when people start at different times and are spread across teams.

## Proposal: HogBoarding

**Core principle:** Teach the PostHog way first, then specialize.

**The system:** New engineers onboard in cohorts of 3-6 people for a standardized 2-3 week period, then transition to their specific teams with a solid foundation.

**Implementation details:**

- **Cohort formation:** Everyone starting within a 2-week window onboards together
- **Rotating leadership:** If 4 product teams are hiring, any mixture of those team leads can run onboarding (spreads load, prevents burnout, ensures knowledge sharing)
- **Standardized curriculum:** Development practices, architecture overview, core tools
- **Cross-product work:** Rotating pool of "good first issues" across all products
- **Pair programming and code review sessions** with onboarding buddies who work together rather than in isolation
- **Cross-team social activities and team introductions**
- **Cohort-based offsites** that are easier to schedule and more cost-effective
- **Clear handoff process** from cohort onboarding to team-specific integration

## Benefits

**TL;DR:** Consistent experience, cross-team connections from day one, better cultural transmission, pays down technical debt, scalable, and easier logistics.

**Detailed benefits:**

1. **Consistent, high-quality onboarding.** Every new engineer gets the same foundational experience, making it easier to assess performance and identify areas for improvement.

2. **Stronger cross-team connections.** People build relationships across the engineering org from day one, improving collaboration and knowledge sharing.

3. **Better cultural transmission.** Teaching autonomy and ownership across multiple product contexts helps new hires internalize these values more effectively.

4. **Efficient use of starter tasks.** We systematically pay down technical debt across all products while giving new hires diverse learning opportunities.

5. **Scalable and sustainable.** The rotating team lead model prevents onboarding from becoming one person's full-time job while ensuring institutional knowledge is preserved.

6. **Enhanced team bonding.** Cohort-based onboarding creates stronger peer relationships (we can include custom cohort merch!) and shared experiences.

## Why not?

**"We want hires shipping ASAP - this approach might slow them down"**

Fair concern, but the current system is already slow due to inconsistency. Some teams ramp people in days, others take weeks or months. HogBoarding frontloads 2-3 weeks of structured learning, but people transition to teams with a much stronger foundation. The goal is faster time-to-meaningful-contribution across the entire engineering org, not just for teams that already have good onboarding.

**"Hard to assess autonomy if folks are going through a bootcamp - it's a facsimile of real work"**

True, bootcamp work isn't the same as real product work. But autonomy isn't just about working independently - it's about judgment, asking good questions, and understanding our systems. The current approach makes it impossible to assess these consistently since everyone has different experiences. HogBoarding gives us a shared baseline to evaluate from, then teams can assess real autonomy during the transition period with actual product work.

## Implementation

**Pilot approach:** Start with 2-3 cohorts to test the approach and refine the curriculum. Create standardized onboarding materials that any team lead can pick up and run with.

**Success metrics:** Track time-to-first-meaningful-contribution, cross-team collaboration frequency, and new hire satisfaction to measure effectiveness and iterate on the process.

## References

This approach is inspired by [Facebook's Engineering Bootcamp](https://engineering.fb.com/2009/11/19/production-engineering/facebook-engineering-bootcamp/), adapted for PostHog's culture and scale.
