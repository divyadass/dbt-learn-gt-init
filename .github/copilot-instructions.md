# Copilot instructions for this repo

This is a dbt project (see `dbt_project.yml`). The guidance below is intentionally concise and focused on patterns an AI coding agent should know to be productive here.

## Project snapshot
- dbt project name: `jaffle_shop` (root `dbt_project.yml`).
- Models live under `models/` with two primary areas:
  - `models/staging/jaffle_shop/` — staging SQL files (prefix `stg_`), example: `models/staging/jaffle_shop/stg_jaffle_shop__orders.sql`.
  - `models/marts/` — marts and intermediate models; example: `models/marts/core/int_orders__pivoted.sql`.
- Default materializations: staging = `view`, marts = `table` (configured in `dbt_project.yml`).
- Local helper/internal packages: `dbt_internal_packages/` contains local macro sets (e.g. `dbt_internal_packages/dbt-bigquery/macros`).
- Build artifacts and debug targets: `target/compiled/`, `target/run/`, `manifest.json`, `run_results.json`.

## Key patterns and conventions (do not invent alternatives)
- Naming: staging models start with `stg_`; intermediate models may use `int_` or descriptive names. Follow existing prefixes for consistency.
- Model configs: materialization and other overrides are set in `dbt_project.yml` or in-model via `{{ config(...) }}` — check both places before changing behavior.
- Adapter-specific macros: look in `dbt_internal_packages/dbt-bigquery/macros` for BigQuery-specific implementations. Macros commonly use a prefix like `bigquery__` (example: `bigquery__create_table_as` in `dbt_internal_packages/dbt-bigquery/macros/adapters.sql`).
- Tests: project-level SQL tests are in `tests/` and compiled generic tests appear under `target/generic_tests/`.

## Recommended dev workflows & commands
- Build the whole project: `dbt build` (recommended initial command).
- Compile only (to inspect macro expansion and SQL): `dbt compile` → then inspect `target/compiled/`.
- Run a single model (example): `dbt build --select marts.core.int_orders__pivoted`.
- Run tests: `dbt test --select <selector>` or `dbt build` (includes tests).
- Inspect runtime artifacts: open `target/run/*.sql` and `target/compiled/<project>/models/...` for the exact SQL dbt will execute.

## Integration points & notable divergences
- `dbt_internal_packages/dbt-bigquery/macros` contains BigQuery-adapter macros with explicit divergences and comments — review those before changing behavior that affects partitioning, clustering, or contracts.
- The BigQuery `create_table_as` macro mentions a conflict between time-ingestion partitioning and contracts; treat those flags cautiously when changing configs.

## Quick guidance for editing
- When changing macros or adapter-level behavior, run `dbt compile` first to verify generated SQL; inspect `target/compiled/`.
- Prefer small, isolated edits: update one model or macro, compile, then run tests for the affected models.
- Use existing naming and folder structure; add tests under `tests/` adjacent to the affected model where feasible.

## Files to check for context
- `dbt_project.yml` (root): materializations, paths, profile.
- `models/staging/jaffle_shop/stg_jaffle_shop__orders.sql` and `models/staging/jaffle_shop/stg_jaffle_shop__customers.sql` (staging examples).
- `models/marts/core/int_orders__pivoted.sql` (intermediate/marts example).
- `dbt_internal_packages/dbt-bigquery/macros/adapters.sql` (BigQuery macro examples and divergences).

If anything here is unclear or you want me to expand examples or merge with an existing AI guidance file, tell me which sections to refine.
