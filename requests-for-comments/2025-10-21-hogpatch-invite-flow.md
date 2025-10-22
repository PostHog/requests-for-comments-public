# Request for comments: Hogpatch invites (Scott)

## Problem statement
We haven't yet set up a clear, organized way to invite YC founders to the Hogpatch office. The onboarding process is manual and requires connecting lots of tools, making it difficult to ensure that every user gets the right comms and access permissions.

## Success criteria
A solution that's low friction to users, scalable to us so the experience feels consistent

## Context
We're getting close to opening up the space (from Nov 4 onwards probably). This RFC outlines the invite sequence that I've provisionally set up on Zapier to automate the onboarding process for YC founders. I'd like feedback on the flow, copy/images used, etc.

---

### Step 1: Email from James
```
hey [name], i’m one of the cofounders of posthog & a fellow yc alum. wanted to reach out as i can see you recently joined our deal, and we've been thinking of ways to help out the current yc batch even more. so next week we're opening up a top secret, invite-only coworking space round the corner from YC and i'd love for you and your cofounders to get access!

we'll open the doors on tuesday (nov 4) and i'll be there all week so would be cool to see you there

i'm inviting a very small handful of founders in the current batch, so let me know if you're interested? if so, let me know your cofounders' email addresses and i'll get our office manager to send digital passes straight to you!!

thanks,
james
```

### Step 2: Judy adds emails to table in zapier and triggers zap "create user + send waiver" on Mon Nov 3, which generates a pass and emails the following:
```
Subject: Your all-hours Hogpatch pass

Hey [Name]

Following up on James' invite to Hogpatch - here's your 24/7 access pass to our bright, lofty coworking spot in Dogpatch, exclusive to only a handful of select YC founders. Welcome to the club, team [company name]!

Our secret address (for your eyes only): [address w/ URL redacted, will be in email] - opening tomorrow!
[Image tbc - https://github.com/PostHog/posthog.com/issues/13275]

We've kitted the place out to be an insanely comfortable and quiet, founder-only coworking space to get stuff done whenever you're nearby. More info & FAQs <a href="https://posthog.com/handbook/growth/sales/hogpatch">in our handbook</a>. FYI it's completely free...no catch, no strings attached. It's just a space to work, think, and escape the noise whenever you need it.

[Button] Download your digital wallet pass

This is your personalized pass, and I've sent your cofounders their own passes too - you'll all have access for the next three months.

The office is fully self-serve, so drop by anytime. I'm around most weekdays 9am–5pm and can't wait to see you there!

Judy
```

### Step 3: User visits to digital pass link
[Example link to demo this step](https://hogpatch-access-waiver.zapier.app/review?email=scott.l+max2@posthog.com&name=Max&company=PostHog&date=10/22/25%2005:10AM&pass=https://pub2.pskt.io/07We2PjflMlfqAwMwf5f1A.pkpass)
- User has to accept Ts&Cs to proceed. Fields autopopulate from their email link. Wording is:

| By accepting below, you acknowledge that you have read, understood, and agreed to these terms. | |
|-|-|
| Full name: | [name] |
| Company: | [company] |
| Date: | [date accepted] |
| Checkbox (required): | I accept the terms and conditions below |
| Button: | [Accept terms & get my pass](https://hogpatch-access-waiver.zapier.app/pass?terms_acceptance=true&full_name=%7B%7B+params.name+%7D%7D&company=PostHog&pass=https%3A%2F%2Fpub2.pskt.io%2F07We2PjflMlfqAwMwf5f1A.pkpass&email=scott.l+max2%40posthog.com) |
| Access waiver written in full underneath |  |

- Page procceds to the success page. Wording is:

| | ![image](https://interfaces-cdn.zapier.com/85ee5293-2f71-4f70-b1c4-fb08e743eb72/hogpatch_desk.png) Here's your golden ticket  | |
|-|-|-|
|  | [Download your 24/7 digital access pass](https://pub2.pskt.io/07We2PjflMlfqAwMwf5f1A.pkpass) |  |
| The Hogpatch Handbook | | Got a question? |
| Read info & FAQs to make your experience comfortable and easy for you to settle in |  | Judy, our office manager, is dedicated to helping YC founders out |
| [View handbook](https://posthog.com/handbook/growth/sales/hogpatch) |  | [Email Judy](mailto:judy@posthog.com)|


### Step 4: Pass opens
[Example link to demo this step](https://pub2.pskt.io/07We2PjflMlfqAwMwf5f1A.pkpass) - works best on mobile/safari, otherwise redirects to a passkit page with QR code

#### Front of pass:
| Hogpatch | | [address] |
|-|-|-|
| **NAME** | **COMPANY** | **YC BATCH** |
| [name] | [company] | [batch] |
| | [qr code] | |

#### Back of pass:
| Your access pass | |
|-|-|
|**Enjoy the space, it's all yours**
Hogpatch is our dedicated, free-to-use coworking space for invited YC founders in the current batch.
We're open 24/7. No reservations or check in needed - visit to work or chill anytime. More info and FAQs <a href="https://posthog.com/handbook/growth/sales/hogpatch">in our handbook</a>| |
| Address | [redacted, will be in pass] |
| Wifi password | pineapple-on-pizza |
| Pass expiry | [date shows 3 months after invite] |
| Access waiver | accepted |
