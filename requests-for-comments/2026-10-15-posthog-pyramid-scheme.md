# RFC: The PostHog Pyramid Scheme 

## Context

We grow through word of mouth and have a lot of people talking about us. That’s great!
We’ve had several runs at making a referral program and never got it off the ground. That’s bad!

It’s one of my goals this quarter to get this question resolved once and for all. Should we launch a program? What would it look like?

A surprisingly large number of people have either tried to work on this project before, or indicated they’d see value in it. So, I’m kicking off with an RFC now that I’ve done enough exploration to have a clear proposal. 

> The tl;dr proposal is that we should create a referral program, that it should be available to free and paying users, that it should offer a cash payment staggered on two conversion events, and that we should launch this with Dub as the provider. 
>
> We’ll embed the referral system in the settings menu, activate users to it by email, experiment with various triggers for this, and aim to have an MVP working well by the end of the quarter. After that we turn to optimization and rolling out further, through affiliates and influencers. 


I’ve been heavily involved with referral programs at several previous startups, so I’m intending to keep this proposal simple and establish a platform from which we can experiment later. 

## Why buy rather than build?

We’ve looked at building our own version on at least two occasions, as hackathons. 

- [The PostHog Pyramid Scheme](https://github.com/PostHog/requests-for-comments-public/issues/205), created by Ben W, Raquel, and I 
- [The New Pyramid Scheme](https://github.com/PostHog/posthog/pull/58463) by Matt P

The idea was also previously proposed at at least three other hackathons. However, none of these versions ended up launching to users — primarily because...

- There’s a complex stack of work involved, touching billing, product, and marketing. 
- Hackathon projects can often die when Normal Work resumes

We also previously offered a referral system within the PostHog for Startups program, though this was a manually run system that offered credits for VC connections. This was successful in bringing in good leads, but was ultimately retired because of the manual effort required. 

I don’t want this to happen again and I don’t want us to waste more time at future hackathons on this, so after talking to three different providers (MentionMe, Rewardful, Dub) I want to fast-track to using an existing tool. I think Dub is the best fit for a few reasons...

- Seems to be well-liked by our team and engineers, which is always a plus. 
- We use Dub already for link attribution, so we’re not having to adopt an entirely new tool. 
- They have existing integrations with platforms we’d want, like Shopify and Stripe. 
- Robust options for extending the program into social metrics and affiliate, as well as support for tiered programs if we want to test influencer programs at a later step. 

Also, of all the demos I saw, I just liked their UI the best and could see us getting this working pretty simply. 

## Audience and Incentives

We’ll offer the referral program as an option to all users, free and paying, regardless of whether they’re on a freemail account. 

These users will be able to refer a friend by generating a link. They’ll give these links to referees who can use it to create their account. 

We’ll view referrals as a paid platform in terms of management, and as a result the incentives will be based on a percentage of the existing LTV and CAC insights branded campaigns run by Demand Gen. We’ll offer two rewards to start with: 

- $20 per new org, _non-freemail_ sign-ups 
- $150 if a credit card is entered on the account

All cash would be paid out monthly and is facilitated through Dub’s platform. 

The non-freemail filter is important, both from a QA perspective and also to avoid easy gaming of the system. We’re valuing this on the basis that non-freemail accounts can then be subject to filtering and become leads for Sales. 

Dub has a pretty comprehensive risk monitoring system which should help us spot gaming of the system, as well as the ability to set exclusion criteria on conversion events — so we can prevent users claiming duplicates by removing/adding cards repeatedly. 

### Other incentives

- **Two-way incentives.**For the sake of speed and to give a baseline to start with, I’d like to launch with one-sided incentives for the referring party only. Two-way incentives typically perform better and I’d anticipate us scaling up to this — the figures above deliberately have lee-way in them so that we can expand to two-way incentives if needed. But we should see how one-way works first. 

- **Credits.** I’m open to offering PostHog credits in place of cash and we’ve discussed the option of offering a 2x benefit if users choose to take rewards as credit instead of cash. I’d like to rule the latter option out for launch, but would defer to the Billing and Finance teams if there was a strong credits-over-cash opinion. 

- **Merch**. I’ve also considered merch as a possible incentive option, but ruled it out. The reason for this is partly practical (we’d be limited by merch stocks and production times) and partly experience (having run previous referral programs, merch rewards perform worse). 

## Distribution

Primarily we’ll draw awareness to the program through email channels and lifecycle marketing, targeting only qualified leads to start with. We’ll target any users who: 

- Have successfully ingested data, _and_
- Have either paid an invoice, shipped a self-driving PR, or joined the startups program

The intent here would be to focus on users who are either heavy users of the product (above free tier), or get high/unique value (auto-PR, lots of credits). 

However anyone will be able to discover the referral program from where it is embedded in the settings menu in the web app and we’ll likely expand and test this criteria as we grow. 

## Risks

### What if this blows up?
This is why we’re keeping incentives and costs in line with what we pay based on branded search. 

But also, this would probably be a nice problem to have. If we seen a massive adoption curve we’ll have several options available, including downscaling rewards, limiting slots, implementing caps on rewards, gating the program, or minimizing distribution. 

### Brand risk

This is the biggest risk. A lot of PostHog's word of mouth carries weight because nobody is paid to say it. Some people will drop referral links into Reddit threads, Discord servers, and HN comments, and readers can't tell a genuine recommendation from a paid one. If PostHog becomes the product people spam links for, we’ll lose more than the program earns.


I’m trying to offset for this with the distribution targeting active, engaged users — but it’s also something we’ll have to monitor and make a vibes call on as we move forward. 

## Next steps

- [ ] Get feedback on this RFC and lock-in format and incentives
- [ ] Confirm budget (I’m assuming this would come from Demand Gen budget)
- [ ] Upgrade to Dub’s Advanced tier ($300 p/m) to unlock features
- [ ] Build out MVP, source needed art, create email flows
- [ ] Launch MVP
- [ ] Profit

## Success and kill criteria

I'll consider the test a success if, within 12 weeks of launch, we're paying out more than $2,000 a month and climbing and cost per referred paying account is at or below what we pay through branded search. We also need to be able to run experiments inside the program.

We'll kill or rework it if we haven't hit $2,000 a month within 12 weeks of launch, or if experimentation pushes costs up beyond the branded campaign payout. 

Dub's Advanced plan caps payouts at $15k a month, which gives us a hard ceiling while we test.
