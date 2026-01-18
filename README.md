# Orders Data Product

## Purpose

The **Orders Data Product** is a **domain-owned analytical data product** that exposes
**business-ready MART tables** for order analytics.

This repository defines:
- Business logic (SQL)
- Data models (STAGE → MART)
- Governance intent (access, PII, retention)
- Quality expectations

**Execution, orchestration, and enforcement are owned by the Data Platform.**  
This repository is intentionally **passive**.

---

## Platform–Product Contract

This repository follows the **Platform–Product contract model**.

| Concern | Owner |
|------|------|
| Business logic & SQL | Data Product |
| Data modeling | Data Product |
| Governance intent | Data Product |
| Execution & CI/CD | Data Platform |
| Policy enforcement | Data Platform |
| Secrets & credentials | Data Platform |

The platform **pulls and executes** this repository using `product.yml` as the contract.

---

## Repository Structure

order-data-product/
├── .github/
│ └── workflows/
│ └── data-product.yml # Delegates execution to platform
│
├── product.yml # Product contract (control plane)
│
├── transformations/
│ ├── dbt_project.yml
│ ├── packages.yml
│ ├── models/
│ │ ├── stage/ # Internal standardization
│ │ │ ├── stg_orders_header.sql
│ │ │ └── stg_orders_line.sql
│ │ └── mart/ # Business-ready outputs
│ │ └── fct_orders.sql
│ └── schema.yml
│
└── README.md


This structure is **required** by the platform runner.

---

## Data Modeling Standard

### STAGE Layer (Internal)

**Purpose**
- Column standardization
- Type casting
- Light derivations

**Rules**
- No business aggregations
- No cross-domain joins
- Not consumer-facing

STAGE models are **implementation details** and may change.

---

### MART Layer (Contractual Output)

**Purpose**
- Business facts
- Metrics and KPIs
- Analytics-ready tables

**Rules**
- Aggregations allowed
- Business logic allowed
- Consumer-facing

**Only MART models are supported outputs of this data product.**

Downstream consumers must depend **only on MART tables**.

---

## Platform Execution Flow

This repository does **not** execute pipelines itself.

Execution flow:
1. A change is pushed to this repository
2. `.github/workflows/data-product.yml` is triggered
3. Control is delegated to the Data Platform
4. The platform:
   - Clones this repository
   - Validates `product.yml`
   - Executes dbt using product SQL
   - Enforces governance and quality
   - Publishes metadata

This ensures **consistent execution across all data products**.

---

## Product Contract (`product.yml`)

`product.yml` is the **single source of truth** for platform integration.

It defines:
- Product identity and ownership
- Execution engine
- Governance requirements
- Quality SLAs

If `product.yml` is missing or invalid, the platform **will not execute** this product.

---

## Governance Model

Governance is expressed as **intent**, not implementation.

| File | Declares |
|----|--------|
| `governance/access.yml` | Who can read product outputs |
| `governance/pii.yml` | PII classification and masking intent |
| `governance/retention.yml` | Retention expectations |

The platform translates this intent into enforceable policies.

---

## Quality & SLAs

Quality expectations are declared in code and enforced by the platform.

Violations are treated as **contract breaches**, not runtime errors.

---

## Ownership & Support

- **Domain Owner**: Orders / Commerce
- **Data Product Owner**: Defined in `product.yml`
- **Execution Owner**: Data Platform team

| Issue Type | Owner |
|---------|------|
Business logic | Data Product |
Data correctness | Data Product |
Pipeline failures | Data Platform |
Governance enforcement | Data Platform |

---

## What This Repository Does NOT Do

By design, this repository does **not** contain:
- Pipeline orchestration logic
- Secrets or credentials
- Ingestion frameworks
- Platform utilities
- Policy enforcement code
---

