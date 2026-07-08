# Entity-Relationship-Diagram-2026

Trip management system for **The Victory Leonard**, a provincial bus company — built for Data Engineering Practices (Normalization, Data Warehousing, SQL). Full walkthrough is in [`notebook.ipynb`](notebook.ipynb); this README mirrors its structure.

## Abstract

This project redesigns, implements, and verifies the relational schema for a provincial bus company's trip management system. An initial ERD (`ERD_LT1.png`) was presented and critiqued for three cardinality errors that would have caused lost maintenance history, an unrepresentable depot state, and an unrealistic staffing model. The corrected ERD ([`ERD.md`](ERD.md)) was implemented as a PostgreSQL schema on Amazon RDS, seeded with 377 synthetic rows across 8 tables (200 of them ticket transactions), and exercised end-to-end with CRUD operations. All three flagged issues are shown, with live query evidence, to be resolved in the final schema.

## Motivation

Getting cardinality right in an ERD determines what the database can and cannot represent once real data starts flowing through it. Three design choices in the initial ERD were flagged as likely to cause real operational problems:

- A **one-to-one bus↔maintenance** relation would silently overwrite service history on every repeat visit.
- A **many-to-zero depot↔bus** notation had the cardinality backwards — a new depot legitimately starts with zero buses.
- A **one-to-one employee↔schedule** relation would prevent any driver or conductor from ever working more than one trip.

## Introduction

The system tracks the operational backbone of the bus company: which buses are housed at which depots, when each bus was last serviced, which routes and stops exist, which employees are staffed as drivers or conductors, which dated trips (`schedule`) are running, and which tickets were sold against them.

## Definition of Terms

**Modeling terms** — ERD, primary key (PK), foreign key (FK), cardinality (one-to-one / one-to-many / zero-or-many), normalization, idempotent script, CRUD (Create, Read, Update, Delete).

**Domain terms** — depot (houses buses), route (start/end path of stops), stop (boarding/alighting point), schedule (one dated trip), ticket (one passenger's paid trip transaction), manifest (tickets sold on one schedule), maintenance record (one dated service event).

See the notebook's [Definition of Terms](notebook.ipynb) section for the full glossary.

## Methodology

1. **RDS Provisioning & Connection** — managed PostgreSQL on Amazon RDS; the app role/database is bootstrapped by [`setup_admin.sql`](setup_admin.sql), and the notebook connects via `jupysql` using credentials from a git-ignored `pw.txt`.
2. **Reviewing the Initial ERD** — the as-presented design (`ERD_LT1.png`) and the three issues identified above.
3. **Schema Redesign** — `maintenance` made one-to-many with `bus`; `bus.depot_id` made nullable (zero-or-many depot↔bus); `schedule` remodeled as an actual dated trip with separate `driver_id`/`conductor_id` FKs; `ticket.schedule_id` added to make each ticket a true trip transaction.
4. **Implementing the Corrected Schema (DDL)** — idempotent rebuild (drop children before parents, then `CREATE TABLE` with the fixes as constraints).
5. **Updated ERD** — final design, also saved as [`ERD.md`](ERD.md).
6. **Data Seeding** — synthetic dataset loaded from [`seed_data.sql`](seed_data.sql).
7. **Verification** — row-count sanity checks, table/column inventory, and sample rows per table.
8. **CRUD Verification** — a per-trip manifest query, one proof query per schema fix, and a live Create → Update → Delete cycle.

## Finding

- Seed data matches expected row counts exactly across all 8 tables (377 rows, 200 of them tickets).
- Per-trip manifests are queryable end-to-end; the Buendia → Davao route (longest at 1,450 km) dominates revenue.
- **Fix #1 confirmed:** bus VL-101 retains all 4 of its maintenance records instead of being overwritten.
- **Fix #2 confirmed:** Sta. Rosa Depot (newly opened) correctly holds 0 buses.
- **Fix #3 confirmed:** driver Carlo Ramos is assigned to 6 separate dated trips.
- CRUD operations (create a ticket, update maintenance status, delete the ticket) all behave correctly, leaving the schema in a consistent final state.

All three cardinality issues raised during the initial ERD's presentation were corrected in the physical schema and independently verified with live queries — not just asserted in the diagram. See [`notebook.ipynb`](notebook.ipynb) for the full walkthrough with queries and outputs.
