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

**Decision:** we went with option a. See the [Outcome](#outcome) section below for what was built.

### Option d) Something easy I'm not seeing right now?

- Let me know.

## Outcome

_Updated July 2026: the system has been implemented and is in production. The living documentation is [`posthog/schema_migrations/README.md`](https://github.com/PostHog/posthog/blob/master/posthog/schema_migrations/README.md); this section summarizes it._

Migration logic lives only on the backend, in [`posthog/schema_migrations/`](https://github.com/PostHog/posthog/tree/master/posthog/schema_migrations):

-   Query nodes carry an optional integer `version` field. A missing version means version 1. Versions are tracked per node kind (e.g. `TrendsQuery`), so changing one kind doesn't force migrations for the others.
-   Migrations are numbered files like Django migrations (e.g. `0001_retention_query_show_mean.py`). Each defines a `SchemaMigration` with `targets` (the node kinds and versions it upgrades from) and a `transform` that rewrites the query dict. A CI check enforces that versions per kind stay linear, without gaps or duplicates.
-   The backend upgrades queries at read/execution time wherever persisted queries enter the system: the query endpoint, insight serialization, cache warming, alerts, exports, cohort queries, and endpoint snapshots.
-   The frontend never re-implements transforms. Queries constructed in frontend code are stamped with the latest versions (`setLatestVersionsOnQuery`, backed by a generated `latest-versions.json`), so they never need migrating. For stored queries the backend can't upgrade in place (URL-embedded queries, notebook nodes), the frontend checks versions locally and only calls the dedicated `POST /api/environments/:id/query/upgrade` endpoint when a query is actually outdated. In practice this makes the extra round-trip from option a's cons rare.

The open questions were resolved as follows:

-   **Version field:** yes, per node kind, defaulting to 1 when absent.
-   **Keeping migrations:** migrations are kept indefinitely; old queries are always upgraded forward, never discarded. Insights stored in Postgres are additionally backfilled by a scheduled Temporal workflow to shrink the long tail of old versions. Other stores (notebooks, endpoint snapshots, dashboard templates, cohorts) rely on read-time upgrades forever. Deprecated fields stay in the schema until we're confident no un-upgraded data remains.

Legacy `filters`-based insights still exist and are converted at runtime via `filter_to_query`. That conversion is composed with version upgrades in the `upgrade_query()` context manager, so one call handles both legacy filters and outdated query versions.
