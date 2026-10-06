# Request for comments: One alpha or beta per team only

Borrowing some maths [from Paul](https://github.com/PostHog/requests-for-comments-internal/pull/1286), we currently have:

- 7 products marked as BETA
- 10 products marked as ALPHA
- 63 products total

That's more than a quarter of the platform marked as not ready for GA, and the number is going up. We're adding new alphas and betas faster than we're shipping existing ones to GA.

I've been banging this drum for a little while. Paul's recent RFC on removing alpha and beta tags from the sidebar ([#1286](https://github.com/PostHog/requests-for-comments-internal/pull/1286)) tackles the symptom (how we label things), but I want to tackle the cause (how many things we let sit in that state, and for how long).

Paul used an open mic night story for how he thinks of this. I’ll use a creative writing story. 

In creative writing there's a rule called "murder your darlings". You cut the bits you love when they aren't serving the work, both because the things _you_ think are your best work often aren’t and also because it best serves the story. At the moment, we're not murdering our darlings. We're keeping all the darlings, and PostHog is suffering because of it.

## Why this is a problem

There are other issues, but these are the big ones I see. 

**It erodes our positioning.** We want to be seen as shipping fast. We also want to be seen as reliable. A constantly growing surface of unfinished products works against both points. 

**It makes marketing resources hard to allocate.** Marketing at PostHog is small. We can't easily justify putting or hiring developer marketers for products that then sit in closed beta for a year.

**It overwhelms users.** More than a quarter of what users see in the sidebar is knowingly unfinished, but still taking up space and cognitive load.

**It's cultural drift.** AI makes it easier than ever to ship code, yet we're getting slower at shipping products to GA. That’s a gap worth talking about elsewhere, but it manifests here.

**It sets users up to have a bad time.** Support for alphas and betas is uneven, and users on them often don't know what they've signed up for.

## Do we need alphas and betas at all?

Yes.

Paul recently proposed replacing the Alpha/Beta label with a “New” label, while Lizzie suggested a “Preview” label instead for the upcoming Managed Data Warehouse beta. 

I disagree with these proposals for a few reasons:

- Labels set user expectations, which is useful when they're used honestly
- Used well, they position us on the cutting edge
- They're a good way to test demand and get feedback
- Some alphas and betas are (or become) business priorities that warrant marketing investment before they reach GA
- Letting people experiment autonomously is core to how we want to work

The problem isn't the word we’re using. It's that we’re not forcing ourselves to take the label off. 

## What I'm proposing

I’m suggesting we put the following three rules into the handbook and into practice:

- Each team can have only one alpha or beta running at a time
- You can't move an alpha to beta (or launch straight into beta) without a release date
- Teams can't request marketing support for a beta without a release date

This works alongside Paul's proposal rather than against it. His component with built-in expiry is a good mechanism for enforcing the release date part of this.

## Why do I think we're doing this wrong right now?

Here’s what the current Alpha and Beta situation looks like, building further on Paul’s data to indicate owners. 

| Item | Stage | Team | Nav label added | Days with label | Where it shows |
|---|---|---|---|---|---|
| Links | Alpha | Growth (top builder: Rafael Audibert) — low confidence, no team claims it | 2025-05-15 | 501 | Nav |
| User research | Alpha | No clear team owner; built by Paul D'Ambra, now on Blitzscale | 2025-05-27 | 489 | Nav |
| Live debugger | Alpha | Developer Experience (top builder: Julian Bez) — low confidence | 2025-10-31 | 332 | Nav |
| Visual review | Alpha | Self-Driving — named in their Q3 objectives | 2026-03-06 | 206 | Nav |
| Metrics | Alpha | APM | 2026-03-13 | 199 | Nav + early access |
| Traces | Alpha | APM | — | — | Early access |
| Managed DuckDB Data Warehouse | Alpha | Managed Warehouse | — | — | Early access |
| Taggers | Alpha | AI Observability | 2026-05-05 | 146 | Nav |
| Business knowledge | Alpha | Conversations — built by Aleks Veryayskiy, and in their objectives | 2026-06-05 | 115 | Nav |
| Engineering analytics | Alpha | Developer Experience — the product README says "Owner: team-devex" | 2026-06-15 | 105 | Nav |
| Identity matching | Alpha | Growth | 2026-06-18 | 102 | Nav |
| AI gateway | Alpha | AI Gateway — a dedicated team now; the ownership table still says Agent Infrastructure | 2026-06-22 | 98 | Nav |
| Code review | Alpha | Self-Driving — built by Alex Lebedev (ReviewHog) | 2026-07-14 | 76 | Nav |
| Pulse | Alpha | Query Performance (top builder: Vasco De Krijger) — low confidence | 2026-07-17 | 73 | Nav |
| MCP servers | Alpha | Context and MCP — owns the MCP store; Self-Driving co-owns the catalog | 2026-07-31 | 59 | Nav |
| Heatmaps | Beta | Web Analytics | 2025-10-08 | 355 | Nav |
| Marketing analytics | Beta | Web Analytics | 2025-12-06 | 296 | Nav + early access |
| Customer analytics | Beta | Customer Analytics (new team); the ownership table still says Web Analytics | 2025-12-22 | 280 | Nav + early access |
| Datasets | Beta | AI Observability | 2026-02-04 | 236 | Nav |
| Self-driving | Beta | Self-Driving | 2026-06-23 | 97 | Nav |
| MCP analytics | Beta | MCP Analytics | 2026-06-26 | 94 | Nav |
| Data catalog / Semantic layer | Beta | Data Modeling — one initiative with two names | 2026-08-19 | 40 | Nav + early access |
| No Code Web experiments | Beta | Experiments | — | — | Early access |
| PostHog Desktop | Beta | Surfaces | — | — | Early access |
| Dashboard widgets | Beta | Product Analytics | — | — | Early access |
| Vim mode for SQL Editor | Beta | Data Tools | — | — | Early access |
| Incoming webhook sources | Beta | Workflows — low confidence; "pipeline sources" belong to Warehouse Sources | — | — | Early access |
| Feature flag notifications | Beta | Feature Flags surface, built by Platform Features — low confidence | — | — | Early access |

Two products have carried a label for more than a year. Nine have carried one for more than six months. 

Three alphas or betas (Desktop, MCP Analytics & Managed Data Warehouse) have a Developer Marketer assigned to them. Six teams own more than one alpha or beta at the moment. 

## Questions I've already been asked

I raised this as a discussion at the marketing offsite last week and the AI offsite this week and her were some questions I got asked:

**"My team has more than one alpha or beta. Do I have to close some?"**

Ideally, yes. Forcing those decisions is the point. It makes us confront whether we're actually investing in everything we've started. Heatmaps has been in beta since October 2025. Do we _really_ want to keep it there?

**"Won't the release dates just change?"**

Probably. That's fine. The date isn't a rigid line in the sand. It's a criterion for moving forward, and it gives other teams the context they need to make their own decisions.

**"I have a really strong idea for a new beta. Why is marketing trying to stifle me?"**

I think this should push you to move your existing alpha or beta forward first, or to make a call about deprecating it. If there's a genuinely strong case for a second one moving fast (say because it's a business priority) that's probably a strong case for spinning up a new small team around it.

**"What if I don't want to kill an alpha or beta?"**

Murder your darlings.

## Why this is a good idea

- It forces tough decisions we're currently too reluctant to make
- It gives clear criteria for what is and isn't OK
- It forces clarity while leaving teams a lot of flexibility
- It prioritizes things from the user's point of view

## Why this is a bad idea

- It's a rigid process, and only works if we actually commit to it
- It's a marketing person telling engineering what to do
