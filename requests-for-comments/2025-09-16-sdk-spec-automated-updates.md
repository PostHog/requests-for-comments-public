# Request for comments: Automated SDK spec updates - Vincent Ge

## Problem statement

We have SDK reference docs now! Four of them already:
- https://posthog.com/docs/references/posthog-js
- https://posthog.com/docs/references/posthog-python
- https://posthog.com/docs/references/posthog-react-native (soon)
- https://posthog.com/docs/references/posthog-node

They're currently manually built and updated, with only 1 version available.

This is a problem because:
- Manual work is annoying. It was fine as we tested these spec, but I think they're good enough now to automate.
- We have all the info needed to maintain versioned references, so why not?
- Could be useful when we release a major version bump of the SDKs, and customers are late to update.

## Proposed solution

Save a copy of SDK references for each version of each SDK, serve all of them , and automatically updating posthog.com/docs/references with the new versions.

## Success criteria

SDK references are always up-to-date without needing any manual work. All past versions are also available.

## Design 

### Where will we store the references?

We'll store them in a folder like `docs` or `references` in each SDK's repo. All of them will be versions and named like this:

- `posthog-react-native-references-latest.json`
- `posthog-react-native-references-v1.0.0.json`
- `posthog-react-native-references-v1.1.0.json`
- etc.

### How will we automate the build?

We'll use GitHub actions to automatically build the reference docs on push to main. The commands/scripts are already available.

- Push to main causes `posthog-react-native-references-latest.json` to be updated
- Releases creates a copy of `posthog-react-native-references-latest.json` and names it `posthog-react-native-references-[the version number].json`

We can consider periodically pruning old versions, but I don't think it's a huge deal.

### How will we fetch the references on posthog.com?

We'll use `gatsby/onPreBootstrap.ts` to fetch the references before building the site. For the site, we can limit to N latest versions and M latest major versions.

These can then be used to:
- Be served from the public folder for LLMs to fetch
- Be used to render pages for each version

### URL patterns on the website

Currently we have `https://posthog.com/docs/references/[SDK-ID]`. This will keep serving the latest release. We'll add a new path `https://posthog.com/docs/references/[SDK_NAME]/[VERSION]` for each version.