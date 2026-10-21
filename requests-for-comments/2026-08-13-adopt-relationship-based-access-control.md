# Request for comments: Adopt relationship-based access control (ReBAC)

**Author:** Reece Jones
**Date:** 2026-08-13

## TLDR

Our access control model no longer scales with our access control needs. As we add access control to more places, we are increasingly relying on one-off hacks that slow development and break easily. Additionally, auditing existing access control logic has become more difficult because it is distributed throughout many different parts of the codebase. This RFC proposes we adopt Relationship-Based Access Control (ReBAC) as a general purpose access control engine to consolidate access control code, decouple logic from consumers, and increase auditability.

## Problem statement

The current access control implementation is hard to develop for, fragile, and difficult to audit, because of the following reasons.

### Access control logic spread across different places

The logic that evaluates access control rules is spread across many places in the codebase. For example:
- *Django viewsets and serializers*
- *HogQL AST parser and printer*
- *Filesystem API*
- *Search API*
- *Frontend access checks*
- *One-off database queries*

This:
1. Couples access control logic to other unrelated logic. For example, access control has become highly coupled to the HogQL implementation. This means that HogQL contributors must be explicitly aware of the access control implementation so queries perform as users expect.
2. Re-implements access control logic in multiple places. For example, access level resolution logic is implemented on both the frontend and backend and need to be kept in sync.

### Multiple access control systems

When our access control system was first developed, it was designed specifically around Django’s data-management patterns, and under the assumption that all resources would be public by default to the owning project. For example, the following product areas have had to implement their own authorization logic, separate from "access control":

- *PostHog AI (Tasks)* - Tasks are visible to the creating user and hidden to the rest of the team by default, however our access control implementation assumes the opposite: that objects are visible to the entire team by default.
- *Billing access control* - Billing lives at the organization-level, but access controls live at the project level, so access control here is implemented by a hacky feature flag that, when enabled, requires the admin organization role to see the billing page.
- *MCP* - We cannot set separate access control policies for MCP sessions
- *Sharing* - Sharing has its own authorization system that uses its own data model and ephemeral authentication tokens.
- *members_can_\* fields* - We store a lot of one-off columns like `members_can_invite` and `members_can_use_personal_api_keys` that are used to implement authorization policies in place.
- *API key scopes* - There is a mismatch between the scopes you can select when creating an API key and what we show in the access control settings UI.

### There is no general purpose authorization library

Right now authorization is handled different ways in different contexts, but there is no cohesive throughline. For example, tenant isolation for organization data is handled completely differently than checking if a user is allowed to create new dashboards.

In the age of agents this also means we cannot easily instruct agents to follow a specific authorization pattern. Authorization is so fractured and context-dependent that we cannot reliably tell an agent how to implement authorization when building new features. This has obvious security implications.

### Auditability

Right now if a user is having trouble accessing something, we have no way of easily figuring out why. Our best bet is to dump their access control rules from the database, feed it to an agent, and ask it to figure out why it isn't working. This is because access control evaluation happens in many different places, with different behaviors, rather than a single common code path.

Additionally, we do not have an audit trail that documents access control decisions. This means we cannot retroactively determine if a user was granted or denied access to a resource, let alone understand why. As we and our customers continue to scale, this will be important to document for security incident response.

### Slow development

For all the reasons mentioned above, developing anything touching access controls is slow. Small changes become complex. As the surface area expands, it gets harder to verify access controls behave as we expect. Some access controls we don't have a good idea how to even build.

## Proposal

### Vision

A cultural shift at PostHog where authorization is a primary concern, and there is one interface that developers across the company use for authorization checks.

### Proposal

I propose we adopt relationship-based access control (ReBAC) and repackage access control as a general purpose authorization library for PostHog developers within the monorepo.

### Success criteria

1. No customer's access is impacted by the change.
2. All authorization decisions are made in a single place.
3. We have audit logs documenting authorization decisions.
4. Authorization becomes standardized.

## Context

### Customer context

Customers keep asking for new access control features that we are unable or too slow to build, because of the inflexibility of our access control system.

### Modern architecture

Modern authorization architecture is typically divided into two components: a Policy Evaluation Point (PEP), and a Policy Decision Point (PDP). The PDP is responsible for answering the question "Can the user do this action?", whereas the PEP is responsible for enforcing the resolved access for the user.

For example, a viewset which returns `Dashboards` would be a PEP. The viewset would ask the PDP if the authenticated user can perform the API action. The PDP returns a boolean response, yes or no, and if denied, the PEP is responsible for terminating the request and responding with the correct HTTP response.

This decoupled architecture improves scalability, because changes to the access control rules are contained to the PDP. The PEP treats the PDP as a black box, which reduces the blast radius of access control logic changes. Additionally, modern authorization systems rely on domain specific languages (DSLs) and specialized data structures to model access control rules. This imposes a design constraint that forces developers to model access control as concise formal logic rather than application code.

See [ABAC](https://en.wikipedia.org/wiki/Attribute-based_authorization) and [Cedar](https://docs.cedarpolicy.com/) for further reading on this architecture.

### ReBAC

ReBAC solves the problem of access control by modeling access as a graph. A user can perform an action on an object if they have a relation, direct or indirect, to that object. This is a natural fit for us, because we already model access control through relations.

For example, a user can edit a dashboard if they are a project admin, or they are an editor for dashboards, or they have edit permissions for that specific dashboard. In this example, the user's access to edit the dashboard is defined by their relationships to the dashboard and the project.

### Chosen implementation: OpenFGA

We will use [OpenFGA](https://openfga.dev/) as our ReBAC engine. It is a cloud-native implementation with a mature ReBAC model and fits our goal of running authorization infrastructure ourselves. It also has a large ecosystem with well documented production use cases for companies at a similar and larger scale than PostHog.

### OpenFGA database

We will use a standalone PostgreSQL database for OpenFGA, rather than creating a separate schema in our existing PostgreSQL database. This follows [OpenFGA's production database recommendations](https://openfga.dev/docs/best-practices/running-in-production#database-recommendations) and helps isolate critical infrastructure components.

### Authorization model preprocessor

We will build a small preprocessor language on top of the FGA modeling language. It will be a slight extension of FGA, not an entirely new language. Our complex authorization model and the large number of resources requiring coverage would make manually writing every definition excessively verbose. The preprocessor will let us:

- Inherit definitions from common types for patterns such as sharing, project-tenancy checks, and typical resource access control.
- Override inherited definitions for resources with special behavior, such as being private by default.
- Organize definitions across multiple files.

### OpenFGA migrations

We will add a dedicated migration track for OpenFGA, separate from the Postgres and Clickhouse migrations. It will be run with `hogli migrations:openfga` and will:

1. Run OpenFGA's PostgreSQL migrations, creating the required tables and indexes.
2. Upload the authorization model and set it as the current model version when the current model in OpenFGA differs from the model version in code.
3. Run any data migrations which might update the relationship graph in OpenFGA.

### Security hub

We will add an authorization section to the security hub with tools to:

- Force-sync tuples from the primary PostgreSQL database into OpenFGA.
- Manually create or remove tuples in OpenFGA for one-off customer authorization scenarios or to fix incorrect tuples.
- Run one-off authorization checks and explain the results of authorization decisions.
- Anything else that will be helpful to debug or fix customer issues.

### Pros & Cons

**Pros**

- *Centralized logic* - All logic is colocated in one place, making it easier to maintain.
- *Audit logs* - With all authorization requests routed through a single interface, we can log authorization decisions for security review.
- *Simplified SDK* - Clients are less aware of the access control implementation and can be thinner, which will help with maintainability.
- *Formalized rules* - Rules are separated from code, and written down to make it easier to reason about, develop, and maintain.
- *More authorization surfaces* - Access control can be used for more use cases such as tenant isolation, plan entitlements, sharing, contextual policies.
- *Own the rules, not the engine* - We own the rules that determine authorization, not the infrastructure code that iterates over DB rows and figures out how to turn that into an authorization decision.

**Cons**

- *More complex architecture* - Requires an additional service deployed to the cluster, with its own data store.
- *Potential for higher latency* - It is possible that there is more network or computational overhead for each authorization request.
- *ReBAC graph could become inconsistent with DB state* - We need to make sure CRUD operations to DB objects are synchronized with the ReBAC graph, and we need to develop tools to respond to potential incidents where ReBAC graph state drifts and becomes inconsistent with the DB. However, once we are fully migrated onto OpenFGA, it will become the source of truth for access control rules. The Postgres-backed access control rule table will be deprecated, and eventually removed.

## Design 
*What are the key user experience and technical design decisions / trade-offs?*

We will adopt a ReBAC framework, then expose a minimal Python SDK for clients to perform authorization checks. In this case the python SDK will act as the PDP, and clients will act as the PEP.

### Python SDK

The authorization facade is the only module other products may import. It accepts immutable data contracts and enums, and does not expose ORM instances or DRF types. The examples below describe how clients use the facade; they do not prescribe its implementation.

```python
from posthog import authorization

# Check one resource, or check several requests in one call.
can_edit = authorization.check(resource=dashboard, action=edit_action, subject=subject)
results = authorization.check([
    authorization.CheckRequest(resource=dashboard, action=view_action, subject=subject),
    authorization.CheckRequest(resource=another_dashboard, action=view_action, subject=subject),
])

# Ask which actions are allowed on a resource.
allowed_actions = authorization.permissions(resource=dashboard, subject=subject)

# List accessible object IDs and subjects who can access a resource.
object_ids = authorization.list_objects(
    resource=project_dashboards, action=view_action, subject=subject
)
subjects = authorization.list_subjects(
    resource=dashboard, action=view_action, subject_type=user_type
)

# Grant and revoke relationships in one batch.
authorization.write([
    authorization.AddRelationshipRequest(
        subject=subject, resource=dashboard, action=view_action
    ),
    authorization.DeleteRelationshipRequest(
        subject=another_subject, resource=dashboard, action=edit_action
    ),
])

# Inspect a relation or explain a decision for debugging and access-rule editors only.
has_edit_relation = authorization.dangerous_query_relation(
    subject=subject, resource=dashboard, level=edit_level
)
explanation = authorization.dangerous_explain(check_request)
```

For example, common access-control operations could use the facade like this:

```python
# A subject cannot edit a dashboard until granted access.
can_edit = authorization.check(
    resource=dashboard, action=edit_action, subject=subject
)
assert can_edit is False

# Grant view and edit access, then check the result.
authorization.write([
    authorization.AddRelationshipRequest(
        subject=subject, resource=dashboard, action=view_action
    ),
    authorization.AddRelationshipRequest(
        subject=subject, resource=dashboard, action=edit_action
    ),
])
assert authorization.check(
    resource=dashboard, action=edit_action, subject=subject
) is True

# Revoke edit access while retaining view access.
authorization.write([
    authorization.DeleteRelationshipRequest(
        subject=subject, resource=dashboard, action=edit_action
    ),
])
assert authorization.check(
    resource=dashboard, action=edit_action, subject=subject
) is False

# A project-level list grant permits listing without granting creation.
authorization.write([
    authorization.AddRelationshipRequest(
        subject=subject, resource=project_dashboards, action=list_action
    ),
])
assert authorization.check(
    resource=project_dashboards, action=list_action, subject=subject
) is True
assert authorization.check(
    resource=project_dashboards, action=create_action, subject=subject
) is False

# Project defaults may allow viewing another dashboard without an object-level grant.
assert authorization.check(
    resource=another_dashboard, action=view_action, subject=subject
) is True

# Batch checks preserve request order; permissions enumerate allowed actions.
results = authorization.check([
    authorization.CheckRequest(
        resource=dashboard, action=view_action, subject=subject
    ),
    authorization.CheckRequest(
        resource=dashboard, action=edit_action, subject=subject
    ),
    authorization.CheckRequest(
        resource=restricted_dashboard, action=view_action, subject=subject
    ),
])
assert results == [True, False, False]
assert authorization.permissions(
    resource=dashboard, subject=subject
) == {view_action}

# Get IDs for filtering a project listing, or subjects who can view one dashboard.
accessible_ids = authorization.list_objects(
    resource=project_dashboards, action=view_action, subject=subject
)
viewers = authorization.list_subjects(
    resource=dashboard, action=view_action, subject_type=user_type
)

# The same relationship API can grant a project role access to a data warehouse table.
authorization.write([
    authorization.AddRelationshipRequest(
        subject=project_role,
        resource=data_warehouse_table,
        action=view_action,
    ),
])

# These inspection helpers are for access-rule editors and debugging, not enforcement.
has_direct_relation = authorization.dangerous_query_relation(
    subject=subject, resource=dashboard, level=edit_level
)
explanation = authorization.dangerous_explain(check_request)
```

Batch checks return one boolean per request in input order. Until request-context binding is available, callers must supply the subject explicitly. `list_objects` returns project-scoped IDs; view/query results may represent included or excluded IDs, and incomplete results allow no objects. `dangerous_` operations inspect relations or explain decisions; authorization decisions must use `check`.

Supporting operations provide a request-scoped session for reusing legacy lookups, synchronize an organization's authorization graph with committed database state, and report OpenFGA status:

```python
with authorization.authorization_session():
    ...

authorization.synchronize_organization(organization_id)
status = authorization.get_openfga_status()
```

The session reuses legacy lookups within a request; it does not cache OpenFGA decisions.

### Authorization availability fallback

We will operate a separate authorization response cache in Redis, outside of OpenFGA. It will help keep the platform available if the OpenFGA infrastructure fails.

The cache will be populated from authorization responses returned by OpenFGA, but will only be read as a fallback during those failures. Once the OpenFGA infrastructure recovers, OpenFGA will become the source of truth again for authorization requests. This is intended to prevent users from being completely locked out of the entire platform when the authorization infrastructure is unavailable.

We will additionally need to invalidate certain sections of the cache when access control rules are updated. For example, when a project's rules are updated, we should invalidate all of the cached access control decisions in the cache.

**Note: this does not guarantee perfect availability. It is meant to improve UX for customers during worse-case failure scenarios.**

### Observability

We will capture authorization metrics from OpenFGA's built-in OpenTelemetry integration and add spans in the Python code to measure the end-to-end latency of authorization calls. We will also create alerts for when these metrics breach defined latency or availability thresholds.

### Keeping the authorization facade focused

We will add lint rules for the authorization facade to prevent new exported functions from being added. We will also update `AGENTS.md` and skills files with instructions for agents on how to use the authorization interface without bloating the SDK. When specific functionality is needed, implement it at the point of use or make the necessary authorization-modeling changes rather than expanding the facade with one-off helpers.

### Gotcha: Django-isms

The mega-brained reader will know that there is a lot of access control checks and enforcement which happens in Django-land. With the new Python SDK we will re-implement the many Django permission enforcement and utility classes to use the new Python SDK under the hood, instead of directly evaluating access control rules from the database.

For example, `AccessControlViewSetMixin` would need to be updated to use the new python SDK instead of implementing the access control checks directly.

### Gotcha: list operations
List operations introduce an additional challenge because our existing access control system enforces object-level restrictions directly in the queryset. This means unauthorized objects are omitted from list results entirely. We will need to be careful to maintain this pre-existing behavior while migrating. Some queries are easy to migrate, only filtering objects that match a list of specific ids, whereas others are more complicated, joining on access control tables.

### Why now?

As we've scaled up the access coverage over more products, the data warehouse, properties, and new AI products, the current access control model has not scaled well. It has slowed down development, polluted some business logic with authorization code, and become difficult to reason about.

### Alternatives considered

**Other ReBAC implementations**

- [SpiceDB](https://authzed.com/spicedb) is an authorization database with support for ReBAC configuration. It is [open source](https://github.com/authzed/spicedb), under the Apache 2.0 License, and maintained by [AuthZed](https://authzed.com), which also offers authorization as a service.
- [Permify](https://permify.co/) is an [open source](https://github.com/Permify/permify) ReBAC implementation that, like OpenFGA, aims to be cloud-native. It is licensed under the AGPL-3 license and maintained by [FusionAuth](https://fusionauth.io/), which offers authorization as a service.
- [Cedar](https://cedarpolicy.com) is an open-source policy engine that supports ReBAC authorization models. Unlike the other engines, it requires us to implement core ReBAC semantics, though it can be embedded in existing programs.
- [Zanzibar](https://storage.googleapis.com/gweb-research2023-media/pubtools/5068.pdf) is Google's in-house ReBAC implementation and the primary inspiration for most modern ReBAC implementations. But obviously we can't use it, because it is a closed-source internal Google product.

**ABAC**

We already have lots of hierarchy in our rules: project admin overrides, creator escape hatch, object-level overrides, roles, etc. that are all dependent on their relationships between each other. ABAC is better suited for flat access control structures that are dependent on inline object attributes. For us, ReBAC is a better fit since it directly models relations.

**Move existing code behind new Python SDK, but don't adopt ReBAC**

In this case, we are still spending a lot of time maintaining the code which evaluates the access control rules. Each new use case requires lots of new Python code that intermixes different concerns and is dificult to refactor.

**Build the ReBAC engine ourselves**

If we adopt a standard ReBAC engine, then code examples on the internet makes it easier for LLMs to use. Additionally,
we want to spend the bulk of our development effort on the authorization rules, not the plumbing that enables that.

### Rollout?
The python SDK will control rollout of ReBAC backed access control. Initially, we will port the existing access control code into the access control product folder, and expose it through the new python SDK. Then, once we start implementing it to be backed by ReBAC, we will set up a flag that determines whether the legacy access control code or the new ReBAC model should be used for authorization evaluation.

This flag will have multiple variants: `off`, `shadow-log`, `rebac`. In `off`, only the existing Python logic is used. In `shadow-log` the existing Python is used as the authoritative answer, but a request is made to the ReBAC service in parallel and that result is logged against the result returned by the Python implementation. In `rebac`, the ReBAC service's response is used as the authoritative answer, and the Python implementation's response is logged.

### Initial load test and rollout

Before routing production traffic to OpenFGA, we will:

1. Deploy a set of OpenFGA pods.
2. Add a Django management command that generates authorization requests for load testing.
3. Run the command from a devbox to measure OpenFGA's capacity under load.
4. Calculate the number of pods needed from the test results and update the scaling configuration to ensure that capacity is available.
5. Gradually route traffic to the `rebac` variant of the rollout flag, monitoring metrics to confirm OpenFGA has sufficient compute capacity.

### Most specific access control?

We will block this initiative on most-specific access control being shipped. Maintaining two big access control changes at once would be too difficult.

### Fine-grained access control only

This system is for fine-grained access control checks, not coarser authorization questions. Coarse checks such as API key scopes will continue to be handled by middleware and their existing mechanisms.

### What authorization questions are in or out of scope?

The goal is for the new engine to answer fine-grained authorization questions. For example, this includes:

1. Organization and project access
2. Contextual policies (i.e. timestmap, risk, etc.)
3. Subject-policies (i.e. token type, mcp user agent, non-user subject, etc.)
4. Feature entitlement checks (note: the billing service will remain the source of truth for feature entitlements)

### New vendor?

No, I think we should avoid this, and remain cloud-native since this is core infra.

### Who would own this?

Platform features would be responsible for maintaining, monitoring, scaling, and on-call response.

### On-call and escalation

We need an new on-call schedule and escalation path for the platform features team so we can respond to issues with the authorization infrastructure.

## Sprints

1. Implement python authorization SDK using existing access control code.
2. Update existing enforcement points in the Python code to use the new SDK instead of the old `UserAccessControl` interface and friends.
3. Implement access control decision logging and metric spans.
4. Set up ReBAC infra in prod, update local dev stack, update self-hosted configuration.
5. Create cronjob and django management command to start syncing relationship graph into OpenFGA.
6. Implement ReBAC backed access control behind a feature flag, add ReBAC decisions to access control logging.
7. Verify existing access control and ReBAC access control agree on authorization decisions.
8. Monitor authorization decisions for 2-4 weeks.
9. Begin phased rollout of ReBAC.
10. Remove old access control logic and make new ReBAC approach source of truth.
