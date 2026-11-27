# Addendum 6: How the PoC code works

The PoC lives in a 5-PR draft stack in `PostHog/context-mill` (`#571` to `#575`) under `scripts/docs-ia/` and `facts/`:

1. [`#571`](https://github.com/PostHog/context-mill/pull/571): Core data models, JSON schemas, shared prompts, and `facts/*.yml` product configs (+1.8K lines).
2. [`#572`](https://github.com/PostHog/context-mill/pull/572): Deterministic tools, Tree-sitter pins, and step-scoped MCP server (+2.0K lines).
3. [`#573`](https://github.com/PostHog/context-mill/pull/573): Workflow runner, `pnpm docs-ia` CLI, and 4 core workflows (+1.8K lines).
4. [`#574`](https://github.com/PostHog/context-mill/pull/574): Claude Code SDK adapter, stateful read/write IA engine, and PR replay workflows (+6.5K lines).
5. [`#575`](https://github.com/PostHog/context-mill/pull/575): Generated `facts/registers/mcp-analytics/*.json` registers and replay scorecards (+22.9K lines, data only).

The design separates deterministic work from model judgment:

- **Plain scripts do the deterministic work:** parsing code, hashing functions, matching ports, splitting MDX, re-anchoring human edits, walking blast radius, and minting permanent IDs.
- **Agents only run where judgment is needed:** extracting facts, mapping cross-team seams, binding doc claims, and judging whether a changed function actually breaks a doc line.

## 1. The five workflows

Every operation is a named workflow in `scripts/docs-ia/workflows/`, run through one CLI (`pnpm docs-ia`):

| Workflow | What it does | Script steps (0 LLM) | Agent steps |
|---|---|---|---|
| `build-ia` | Crawl a product's code and build its coded IA | Index symbols, wire internal ports, mint permanent codes | Extract facts, map `platform-*` seams, assemble procedures |
| `map-docs` | Map live `.mdx` pages to code-pinned facts | Split MDX, rank candidate facts, tally coverage | Bind claims, second-pass contradiction judge |
| `refresh-drift` | Check moved commits and flag broken doc lines | Hash check (`commit` -> `md5`), reconcile human `.mdx` edits, trace blast radius, retire deleted codes | Judge only changed or deleted symbols |
| `replay-prs` | Replay historical PRs to measure accuracy | Same 2-gate diff and reconciler, score predictions | Judge affected sections per PR |
| `compile-skills` | Turn use cases into lean agent skills | Render surface-sliced `UC-*` files, verify 0 broken anchor links | None |

## 2. How the runner keeps agents on rails

Each workflow in `scripts/docs-ia/workflows/` is just a list of `scriptStep` and `agentStep` objects. Before step 1 starts, `runtime/runner.mjs` checks that every prompt file, JSON schema, and tool name actually exists.

When an `agentStep` runs:

1. **One shared tool catalog:** Every deterministic tool (`read_pinned_slice`, `verify_verbatim_quote`, `hash_refresh_pins`, `lookup_facts_by_symbol`, and more) lives once in `tools/catalog.mjs`. Scripts call these functions directly.
2. **Step-scoped MCP server:** For an agent step, the runner launches `tools/mcp-server.mjs` exposing **only** the tools listed on that step. An extractor step can verify quotes and read slices, while a judge step can inspect blast radius and audit trails.
3. **Shared prompt rules and schemas:** Prompts live as plain Markdown files under `prompts/` and pull shared rules from `prompts/shared/` (verbatim quote check, source quarantine, port list, pointing-and-calling). Every agent response validates against a JSON Schema in `models/schemas/`.
4. **Swappable runner backends:** `--provider` switches between `agy` (default) and `claude-sdk` (Claude Code SDK) without touching workflow code.

## 3. Where to look in the stack

Across PRs [`#571`](https://github.com/PostHog/context-mill/pull/571) to [`#575`](https://github.com/PostHog/context-mill/pull/575), everything fits in three buckets:

- **`facts/` (data only):** Pinned repo commits (`repos.yml`), closed product/surface lists (`registries.yml`), per-product repo scopes and exclusions (`products/*.yml`), and permanent registers/scorecards (`registers/mcp-analytics/`).
- **`scripts/docs-ia/models/` and `tools/` (deterministic code):** `models/` holds detached `{ commit, code_hash }` pins, 13 fact types, 10 port kinds, the quote gate, claim states, and the append-only register with tombstones. `tools/` holds the Tree-sitter indexer, 0-LLM port linker, MDX splitter and reconciler, blast-radius walker, and skill compiler.
- **`scripts/docs-ia/workflows/`, `prompts/`, and `runtime/` (orchestration):** The five workflow DAGs, shared Markdown prompts, runner, provider adapters, and CLI (`cli.mjs`).

## 4. What's working vs what's still rough

- **Working now:** All five workflows, the stateful read/write IA engine, deterministic models and tools, the step-scoped MCP server, and CLI lookup (`pnpm docs-ia query`) are wired up. All 26 unit tests (`pnpm test:docs-ia`, 0 LLM calls) pass.
- **Partial data migration:** `facts/registers/mcp-analytics/` holds real replay registers and scorecards. The earlier Surveys artifacts still live in local experiment scripts until we migrate them cleanly.
- **Local CLI only:** Nothing is wired into GitHub Actions, webhooks, or `products/knowledge_writer`.

## Details and stuff

<details>
<summary>How to inspect, run, and extend the PoC stack (#571 to #575)</summary>

### Quick commands (from `PostHog/context-mill` branch `vincent/docs-ia-5-mcpa-data`)

```bash
# Run the deterministic unit test suite (6 files, 26 tests, 0 LLM calls)
pnpm test:docs-ia

# List every workflow, step order, model, prompt, schema, and bound MCP tools
pnpm docs-ia list

# Inspect one workflow's full config as JSON
pnpm docs-ia inspect refresh-drift --json

# Dry-run a workflow without invoking models
pnpm docs-ia run build-ia --product mcp-analytics --dry-run

# Query an IA/register file directly (fact | symbol | page | audit)
pnpm docs-ia query audit MCPA-TS-F-000301 --file facts/registers/mcp-analytics/mcpa.codes.json

# Start the step-scoped stdio MCP server manually
pnpm docs-ia mcp-serve --tools read_pinned_slice,verify_verbatim_quote --state <state.json>
```

### Stack commits

- `0fe62fee` ([`#571`](https://github.com/PostHog/context-mill/pull/571), `vincent/docs-ia-1-models`): Core models, JSON schemas, shared prompts, `facts/*.yml`, `models.test.mjs` (7/7 tests).
- `9de4cb0e` ([`#572`](https://github.com/PostHog/context-mill/pull/572), `vincent/docs-ia-2-tools`): Deterministic tools (`ast-pins`, `tree-sitter-parser`, `port-linker`, `mdx-splitter`, `blast-radius`, `skill-compiler`, `catalog`, `mcp-server`) and `prompts-and-tools.test.mjs` (13/13 tests).
- `56fc6d1e` ([`#573`](https://github.com/PostHog/context-mill/pull/573), `vincent/docs-ia-3-workflows`): Workflow runner, `cli.mjs`, 4 core workflows (`build-ia`, `map-docs`, `refresh-drift`, `compile-skills`), `README.md`, and `workflows.test.mjs` (16/16 tests).
- `b10a34a7` ([`#574`](https://github.com/PostHog/context-mill/pull/574), `vincent/docs-ia-4-replay-and-claude-sdk`): Claude Code SDK adapter, `replay-prs.workflow.mjs`, stateful read/write IA engine (`stateful-ia-engine.mjs`), 8-week replay benchmark (`stateful-8wk-benchmark.mjs`), and tests (26/26 docs-ia tests).
- `a20063d9` ([`#575`](https://github.com/PostHog/context-mill/pull/575), `vincent/docs-ia-5-mcpa-data`): Generated `facts/registers/mcp-analytics/*.json` registers and scorecards including `stateful-8wk-scorecard.json` (37/37 test files, 481/481 tests).

### Key file map

```text
facts/
├── repos.yml
├── registries.yml
├── products/{mcp-analytics,surveys}.yml
└── registers/mcp-analytics/{mcpa.codes.json,mcpa.uc.json,outline.json,t0-summary.json,replay-scorecard.json,claude-sdk-2arm-scorecard.json,stateful-8wk-scorecard.json}

scripts/docs-ia/
├── cli.mjs
├── data/{mcp-analytics-replay-events,oct-nov-8wk-dataset,planted-hard-events}.mjs
├── workflows/{index,build-ia,map-docs,refresh-drift,replay-prs,compile-skills}.workflow.mjs + stateful-8wk-benchmark.mjs
├── models/{pin,fact,claim,register,workflow}.mjs + schemas/*.schema.json
├── prompts/{loader.mjs,shared/*.md,*.md}
├── tools/{catalog,mcp-server,ast-pins,tree-sitter-parser,port-linker,mdx-splitter,blast-radius,skill-compiler}.mjs
├── runtime/{runner,agent-adapter,claude-sdk-adapter,stateful-ia-engine,git}.mjs
└── tests/{models,prompts-and-tools,workflows,tree-sitter-and-replay,claude-sdk-adapter,stateful-ia-engine}.test.mjs
```

### Adding a new product

1. Add the product's code and surfaces to `facts/registries.yml`.
2. Add `facts/products/<slug>.yml` with `repo_prefixes`, `seam_paths`, `watched_manifests`, `surface_rules`, and `exclusions`.
3. Run `pnpm docs-ia run build-ia --product <slug>` followed by `map-docs` and `compile-skills`.

</details>
