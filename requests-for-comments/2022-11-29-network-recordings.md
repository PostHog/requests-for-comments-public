# Request for comments: Network recordings MVP - and Web Performance Alpha to Alpha V0.2, Paul D'Ambra

Desired decision date: December 10th 2022

## Problem statement

_Who are we building for, what are their needs (include example cases), why is this important?_

This RFC is tackling two main problems:

-   Network recordings MVP: To be able to serve a customer need of seeing network logs in session recordings. This can also help with improving our own web performance
    -   This is the focus of Paul and Ben's work during their trip to Lisbon
-   Web Performance v0.2: For ourselves to support "nail slickness". We want to see where in PostHog is it slow and frustrating.

Performance monitoring can be split into Synthetic monitoring (measurements carried out by automation, often on a timer) and "real user monitoring" (measurements gathered from production traffic). Because our SDK is already running in the browser we are in a position to add "real user monitoring" to our product.

Product engineers first want to focus on reducing frustration by speeding up the really slow stuff. Then they want to move onto making the product feel really slick. As part of our mission to help people build the best products, PostHog is well suited to help them improve their applications performance. This is particularly important for the leading product teams such as Superhuman/Linear etc.

They want to answer these questions:

-   how quickly do my assets loads
-   how fast are my API responses
-   do third party requests slow my application down
-   how large are my assets
-   are my assets or server responses compressed
-   have the speed or size of my responses changed over time
-   what factors are affecting page speed of my application
-   how do these web performance measures correlate with my product analytics events
-   what was happening in the browser at a specific moment in a session recording

## Success criteria

_How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?_

-   Network recordings MVP:
    -   We'll know this is successful when 5 target customers have adopted this tool, and given positive feedback.
    -   _TODO: what are the core things that we want users to be able to do with network recordings in Session recordings?_
-   Web Performance v0.2:
    -   _We shouldn't spend too much time on this goal_
    -   The tools we are building out to measure and reduce user frustration (from the point of view of the frontend) are in PostHog.
        -   These features should be generalizable enough that others can use them
    -   We're not adding performance measurement on a timer, and we're not adding alerting on performance issues.
    -   We're not adding performance measurements to mobile or back-end SDKs
    -   For speed, we're not focus on external user feedback for this yet, that can come in a second part of work

## Context

_What are our competitors doing, what are the technical constraints, what are customers asking for, what does the data tell us, are there external motivations (e.g. launch week, enterprise contract)?_

### Similar tools

#### Sentry

https://sentry.io/for/performance

-   backend and front-end performance transactions
-   integrated with error reporting
-   drill down into traces

#### Log Rocket

https://logrocket.com/features/frontend-performance-monitoring

-   network speed measurement
-   web performance metrics like first contentful paint
-   as metrics and alongside session replay

#### GTMetrix/StatusCake/WebPageTest

various similar-ish tools

-   on a timer
-   load a web page and capture performance of network requests and browser performance
-   report on individual page loads

## Design

_What are the key user experience and technical design decisions / trade-offs?_

### Network recordings MVP:

-   Initial alpha focussed on smallest set of data to draw a performance chart for a single page view. It uses the browser's web performance API to capture whatever performance data is available at point of sending a page view event. And includes the entire dataset as JSON on the pageview event. This should be shown in the session recordings
-   This has several drawbacks
    1. the JSON blob is large and we can't query individual items within it
    2. not all of the interesting information is available at the point we send the pageview
    3. we can't (easily) see how much data we're receiving that is specifically performance data
-   But, the advantage of leaning on the browser's API is gathering the data takes very little code. And lets us gather a wide set of useful information

![an example waterfall chart](/images/2022-11-29-network-recordings/rfc-web-perf-waterfall-chart.png)

#### How should we improve this?

#### Gathering

The SDK already tracks a session ID. If we extend this to track a pageview ID (which rotates when we send a $pageview event) then we can send performance data either as a seperate event type, or through a separate channel.

This allows us to separate types of performance data so we can graph speed of API calls or asset requests.

Using a [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) we can register to be notified when performance entries are available and then send them back as events.

This avoids having to patch fetch without ruling it out doing that in future.

#### Reporting

This gives us enough information to:

-   report on API timings and sizes in Insights
-   report on Asset timings and sizes in Insights
-   report on page timing events in Inisights
-   draw performance charts for individual page views or aggregated over a page
-   draw performance timelines for individual sessions or aggregated sessions
-   place network request logs along session recordings
-   highlight slow network requests in session recordings

### This is lots of data

##### Config

We can allow users to configure which types of data they gather. They might not be interested in assets and only want to gather API call performance.

##### Infra / Performance of performance

Since this will send tens or hundreds of events per pageview we could receive a massive amount of traffic if sending each request individually.

So, similarly to how we process session recordings, it makes sense to send info from the SDK in batches `{sessionId: string, pageViewId: string, entries: PerformanceEntry[]}` and then split them on ingestion.

The individual performance entries are relatively small but will be numerous. Storing them separately to events would allow TTL deletion as we do with session recordings.

## Sprints

_How do we break this into discrete and valuable chunks for the folks shipping it? How do we ensure it's high quality and fast?_

-   Network recordings
    -   add page view ID rotation to SDK
    -   add new API endpoint to accept/validate batches of performance entries
    -   update SDK to observe performance entries and batch them to new API
    -   add new route through kafka to a ClickHouse table for performance entries
    -   add new ingestion route that splits batches to the new kafka topic
    -   update existing performance waterfall to use new data store
    -   add network listing to session logs

### Appendix

#### Getting stored performance data currently

```
select formatReadableSize(sum(length(JSONExtractString(properties, '$performance_raw')))) from events
where event = '$pageview'
and timestamp < now()
and timestamp > now() - interval 30 day
and JSONHas(properties, '$performance_raw')
```
