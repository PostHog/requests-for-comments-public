# Request for comments: Support multiple experiments per feature flag

## Problem statement

Our customers wants to be able to run multiple experiments per feature flag. Here are some use cases:
* have a single feature flag in the code, and run one experiment for one segment, and another experiment for another segment,
without having to update the code
* reuse the same feature flag for successive experiments, but with new assignments (i.e a different seed), and different configuration. F.ex, you could have a long-lived "pricing-plan" feature flag, and run multiple experiments after each other where you iterate on pricing, without having to update any code

This is not possible in PostHog today. You can only have a single multivariate configuration per flag. Also, the seed for a feature flag is constant,
so you'll always get the same assignments for a feature flag. You can't reset them. That's a common need if you want to start a new clean experiment
without changing the flag, if f.ex the A/A test showed bias and you want to see if you were just unlucky with the randomization.

Related, the following common workflow is a bit cumbersome in PostHog today:

You're a product engineer and want to implement a new feature behind a feature flag. The feature is simple, so you just need an on/off toggle
in the code, i.e a boolean flag will do. To safely test it, you do a targeted release to beta users first. Things looks good. You now
want to run an experiment on the remaining users. But annoyingly, you can't do that in PostHog without also changing your code. To run
an experiment, you'll have to choose the "Multivariate" flag type. This changes the flag type from boolean to string. You need to change
your code and deploy to be able to run an experiment. There is no good reason for this other than being an artifact of how PostHog works
today.

A better user experience here would allow the developer to add a new "rollout rule" of type "experiment" where the user targeted the remaining
users (those not matched by the "beta user rule"), and define how to split those users into variants (control 50, value: False) and (test: 50, value: True). Without having to change anything in the code.


## Suggested solution

In short, this is what I propose:
* move what we today call "multivariate config", to live under a target condition rule instead. This is the main thing
* decouple "flag return type" with "release type" by having the user specify the return type (boolean, string, JSON). This allows users to run experiments on boolean flags f.ex, and separates the concern (what flag return types you need vs. how you want to roll it out).
* more generally, the different rollout types (targeted relase, percentage rollout and experiment) all live as rules under a targeting condition
and can be combined.

To get a sense of how this will work out in practice, let's take a look at the workflow we looked at above:

With this new structure, you can have target condition for "beta users" with a roll-out rule "targeted release", and another below it which targets "the rest" with an experiment rule that splits users into variants. Note that the flag return value is still a boolean (true/false).

![Feature flag UI](/images/2026-05-04-multiple-experiments-per-feature-flag/fig1.png)


## Alternatives considered

## Success criteria

## Decisions
