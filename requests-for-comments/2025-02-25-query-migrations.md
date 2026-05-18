# Request for comments: Query Migrations

## Problem statement

We'd like to evolve the [query schema](https://github.com/PostHog/posthog/blob/master/frontend/src/queries/schema/index.ts) while keeping backwards compatibility for existing queries.

## Context

Queries are persisted in various places:

1. The `query` field of the Insights model.
2. Local storage for the "query draft" feature.
3. The url for shareable insight links.
4. Notebook nodes (and canvas element).
5. The activity log, when changing a query.
6. Queries that are converted from `filters` e.g. when users create an insight by directly calling the api.
7. ...? _(let me know)_

Some of these places are available frontend-side (e.g. queries in a url) and some are available backend-side (e.g. queries in an insight model).

This makes it difficult to migrate a query with a non-backwards compatibly change e.g. converting a boolean to an enum. Backward compatible changes e.g. adding an optional field are no problem.

Any errors in a query cause Pydantic to fail the validation and thus insights with the error will fail to compute.

## Design

Migrating the schema from one version to the next should be as easy as a database migration for developers.

### Option a) Add a backend side migration and allow the frontend to use this in a new endpoint

We could add only a backend side migration and let the frontend call a `/query/validation` endpoint that also migrates queries to newer versions.

#### Pros

-   Only one place where we'd need to add a new migration
-   Probably the easiest one to maintain

#### Cons

-   The frontend needs to do an additional round-trip whenever a new query is encountered
-   Might be hard to implement frontend side due to the async nature e.g. when migrating notebook nodes

### Option b) Add a backend side and frontend side migration

#### Pros

-   This is how we are currently doing it / trying to do it
-   No round-trip to the backend necessary for new queries

#### Cons

-   We need to maintain a frontend side and backend side migration. It'll be hard to keep them in sync
-   We don't have immediate feedback from the Pydantic validation on validity of queries that are migrated frontend side

### Option c) Allow only backward compatible changes

#### Pros

-   Easiest to implement
-   Easiest to maintain

#### Cons

-   We can only add optional fields
-   Technical debt will increase

I'm favoring option a, where we add a backend side migration and call that from the frontend, as it seems to be the easiest (and as such, most scalable) solution for future developers adding query migrations.

### Option d) Something easy I'm not seeing right now?

- Let me know.

## Implementation

-   Get in touch with customers to deprecate `filters` based insights (already happening). Remove all remaining `filters` conversions.
-   Implement the above.

To be discusssed:
-   Should we add a version field to the queries, so that we know which migrations to run
-   Should we keep migrations forever or should we have a system in place that discards frontend side queries that are older than a certain time
