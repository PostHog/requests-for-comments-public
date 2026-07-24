# Request for comments: Error Tracking - release management

## Big picture

### How it works now

Currently for JS our CLI injects random `chunkId` into every `.js` file and then it creates a release. Every chunk is tied to one release. Cymbal when resolving an exception checks the `chunkId`, looks up the release and attaches it to an event. It is possible that it will attach multiple releases.

For ios, android and other native technologies we rely on `debugId` which is stable but then we get all other sort of problems. Main one is that some chunks will have same `debugId` across different versions and we do different hacks to create a release.

### Main problems

- unstable `chunkId`s - every time someone uploads source maps in JS, all chunks need to be reuploaded,
- monorepos - an app which someone uploads source maps for might not be located at the repo root. We don't really store good metadata on that. This causes problems when we want to then link stack traces to github for example,
- releases - fact that a single exception might belong to multipe releases is wrong. Hacks we do to create releases when `debugId`s are stable are wrong.

### Solution

#### Unstable `chunkId`s

Most platforms/frameworks/build systems expose `debugId`. It is becoming a standard today. We can just use that.

- rollup and vite - `sourcemapDebugIds: true`,
- webpack - `debugIds: true`,
- turbopack - `debugIds: true`,
- ios/android - we already rely on debug ids,
- native technologies - we also rely on debug ids.

We also support raw CLI injection. This is where we won't rely on debug ids. We can easily generate stable chunk ids though - it shouldn't be a problem.

#### More metadata (monorepos)

It shouldn't be complex. We just have to collect more metadata when we run CLI

#### Releases

##### SDK

Ok so the plan is that we drop chunk-release association and start injecting release somehow into the SDK which is running. Obvious solution in JS would be to do the same thing we do with chunk ids where we inject code snippet which makes it accessible to the runtime.

I don't know if that is the best solution and this is something I'll check when working on that.

There was an idea with using virtual modules but turbopack doesn't seem to support it.

SDK will send release in event properties.

##### Cymbal

Cymbal will no longer look up release from the chunk id. It will read it from event properties. To make this backwards compatible we won't completely drop the column. Cymbal will first attempt to read the release associated to the chunk.

##### Risks

In technologies where we control SDK+CLI bundle this will go smooth because when you update, you will get release ids injected and SDK will attach it to events. However - if you are using raw CLI + SDK there is a chance that you will update your CLI but not your SDK.

In such case CLI won't associate chunks with a release and SDK won't send it. As a result Cymbal won't be able to associate release to the exception.

Two things we can do here probably:
- cli - add some piece of logic which tries to detect projects around and check SDK versions. If version is too old, display a very visible warning,
- in app banner - we can detect from in app that user is producing releasless chunks but then exceptions are not tied to a release. We can also check SDK versions. Then we can display a banner that you probably have a version mismatch