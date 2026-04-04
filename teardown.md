# Teardown — DbtEngineer: NYC Parking Violations

A full end-to-end dbt transformation project demonstrating the Medallion Architecture (Bronze → Silver → Gold) applied to the NYC Parking Violations dataset, backed by DuckDB as the local analytical database.

---

## Stack Choices & Rationale

| Component | Decision Rationale |
|---|---|
| **dbt** | SQL transformation layer with built-in testing, documentation, lineage, and materialisation management. Turns raw SQL into a maintainable, version-controlled project. |
| **DuckDB** | Lightweight, file-based OLAP engine requiring zero infrastructure. Perfect for local development — reads Parquet, CSV, and JSON natively and is fast enough to process the full NYC dataset on a laptop. |
| **Medallion Architecture** | Separates raw ingestion (Bronze) from cleansed data (Silver) from business-ready aggregates (Gold). Each layer has a clear contract and can be tested independently. |
| **Ephemeral models (Silver)** | Silver models that are intermediate computation steps are materialised as ephemeral to avoid materialising tables that are never queried directly, reducing storage overhead. |
| **Table materialisation (Gold)** | Gold models are materialised as tables because they are the query target for analytics. Table scans are faster than chained view resolution at query time. |

---

## Key Design Decisions

- **Bronze is read-only** — raw data is never modified post-ingestion. All transformation happens in downstream models, preserving auditability.
- **Failures are stored** via `+store_failures: true` in dbt tests, enabling inspection of failing rows rather than just a pass/fail count.
- **dev and prod environments** point to separate DuckDB files, preventing accidental pollution of production data during development.
- **Custom macros** centralise reusable SQL logic, preventing duplication across models in different layers.

---

## Trade-offs

| Decision | Benefit | Cost |
|---|---|---|
| DuckDB (local file) | Zero ops overhead, instant setup | Not suitable for multi-user concurrent writes or cloud-native deployment without migration to MotherDuck |
| Ephemeral Silver models | No intermediate tables to maintain or version | Harder to debug mid-pipeline; cannot query Silver directly in a BI tool |
| dbt over raw SQL scripts | Testing, lineage, docs, incremental support built in | Requires learning dbt conventions; adds a layer of abstraction over plain SQL |

---

## Extensions & Real-World Use Cases

- Migrate from DuckDB to **MotherDuck** (managed DuckDB cloud) for team collaboration without changing a line of dbt SQL.
- Swap DuckDB for **Snowflake or BigQuery** using dbt adapters — the model logic is database-agnostic.
- Add a **Metabase or Superset** connection directly to the Gold DuckDB file for self-serve analytics on NYC parking data.
- Extend the Gold layer with **geospatial aggregations** by borough or street block — DuckDB supports spatial extensions natively.
- Use as a **teaching template**: the medallion structure and dbt conventions are transferable to any tabular dataset.

---

## Portfolio Signal

This project demonstrates the ability to set up and configure a full dbt project from scratch — a skill that is underrepresented in data engineering portfolios where most candidates inherit an existing dbt setup rather than initialise one.
