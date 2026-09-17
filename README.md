## dbt Data Transformation Pipeline for Databricks
## Executive Summary
This project delivers an automated Data Transformation Pipeline that converts raw, messy operational data into clean, structured tables ready for business analytics. Built using dbt (data build tool) and Databricks, it eliminates manual data preparation, while the uv package manager ensures consistent, rapid team deployment.
## Business Problem

* Unreliable Data: Raw data contains duplicates and missing fields, leading to inaccurate reports.
* Manual Bottlenecks: Manual data cleaning is slow, error-prone, and cannot scale.
* Environment Conflicts: Discrepancies between developer laptops cause code failures during deployment.

## Methodology

* Environment Standardization: Isolating local dependencies using uv for identical environments.
* Modular Modeling: Organizing transformations into distinct cleaning, joining, and final business layers.
* Automated Testing: Validating data integrity at every step before it reaches production.

## Data Source

* Ingests raw operational logs, transactional data, and customer records from cloud storage.
* Integrates static lookup data managed directly via dbt seeds.

## Data Pipeline Approach

* dbt Core: Handles code compilation, documentation, and table dependency mapping.
* Databricks Compute: Offloads high-volume data processing directly to scalable cloud clusters.
* Quality Guardrails: Runs automated schema checks to block null values or broken relationships.

## Skills

* Data Engineering: Pipeline orchestration, dbt architecture.
* Cloud Platforms: Databricks SQL warehouses and cloud compute.
* Database Programming: Advanced modular SQL data modeling.
* DevOps & Tools: Package management (uv, toml) and Git version control.

## Results and Business Recommendation

* Results: Establishes a single source of truth in Databricks, providing fully automated, clean, and audit-ready data for downstream dashboards.
