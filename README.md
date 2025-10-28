# TPC-H Medallion Architecture dbt Project for Snowflake

This project transforms Snowflake's TPC-H sample data through a medallion architecture pattern, creating analytics-ready datasets using dbt (data build tool).

## Project Overview

This dbt project implements a **Bronze → Silver → Gold** medallion architecture on Snowflake, processing TPC-H benchmark data into business-ready analytics layers.

### Architecture Layers

- **Bronze Layer**: Raw data ingestion from `SNOWFLAKE_SAMPLE_DATA.TPCH_SF1`
- **Silver Layer**: Cleaned and enriched data with business logic applied
- **Gold Layer**: Aggregated analytics and business metrics

### Data Models

**Source Tables (8 models):**
- Customer, Orders, LineItem
- Supplier, Part, PartSupp
- Nation, Region

**Gold Analytics Models (5 models):**
- Customer Analytics
- Sales Analytics
- Monthly KPIs
- Regional Analysis
- Supplier Performance

### dbt Packages

- `dbt-labs/dbt_utils` (v1.1.1)
- `metaplane/dbt_expectations` (v0.10.9)
- `dbt-labs/codegen` (v0.12.1)

---

## Setup Instructions

### Prerequisites

- Snowflake account with `ACCOUNTADMIN` role or equivalent permissions
- Git repository containing this dbt project
- Access to Snowflake's TPC-H sample data

---

## Step 1: Create Database and Warehouse

Execute the following SQL commands in your Snowflake console:

```sql
-- Create warehouse
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH
  WITH WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE
  COMMENT = 'Warehouse for dbt transformations';

-- Create database
CREATE DATABASE IF NOT EXISTS TPCH_MEDALLION_DBT_DB
  COMMENT = 'TPC-H Medallion Architecture database';

-- Create API integration
CREATE OR REPLACE API INTEGRATION GITHUB_INTEGRATION
  api_provider = git_https_api
  api_allowed_prefixes = ('https://github.com')
  enabled = true
```
---

## Step 2: Create Git Integration

### Option A: Snowflake UI

1. Navigate to **Projects** → **Git Repositories** in Snowsight
2. Click **+Git Repository**
3. Configure:
   - **Name**: `TPCH_DBT_REPO`
   - **Origin URL**: Your Git repository URL
   - **Authentication**: Configure based on your Git provider
4. Click **Create**

### Option B: SQL Commands

```sql
-- Create Git repository object
CREATE OR REPLACE GIT REPOSITORY TPCH_MEDALLION_DBT_DB.PUBLIC.TPCH_DBT_REPO
  API_INTEGRATION = GITHUB_INTEGRATION
  ORIGIN = 'https://github.com/anthu/dbt_demo_snowflake.git'
  COMMENT = 'TPC-H dbt project repository';

-- Fetch repository contents
ALTER GIT REPOSITORY TPCH_MEDALLION_DBT_DB.PUBLIC.TPCH_DBT_REPO FETCH;

-- Verify setup
SHOW GIT BRANCHES IN TPCH_MEDALLION_DBT_DB.PUBLIC.TPCH_DBT_REPO;
```

**Security Note**: Use Snowflake secrets for credential management. Never commit credentials to version control.

---

## Step 3: Create dbt Project in Snowflake

### Option A: Snowflake UI

1. Navigate to **Projects** → **dbt** in Snowsight
2. Click **+ dbt Project**
3. Configure:
   - **Name**: `tpch_medallion_dbt_project`
   - **Git Repository**: `TPCH_DBT_REPO`
   - **Branch**: `main`
   - **Database**: `TPCH_MEDALLION_DBT_DB`
   - **Warehouse**: `COMPUTE_WH`
   - **Schema**: `PUBLIC`
   - **Role**: `ACCOUNTADMIN`
4. Click **Create**

### Option B: SQL Commands

```sql
-- Create dbt project
CREATE OR REPLACE DBT PROJECT TPCH_MEDALLION_DBT_DB.PUBLIC.tpch_medallion_dbt_project
  GIT_REPOSITORY = TPCH_MEDALLION_DBT_DB.PUBLIC.TPCH_DBT_REPO.TPCH_DBT_REPO
  GIT_BRANCH = 'main'
  DATABASE = TPCH_MEDALLION_DBT_DB
  WAREHOUSE = COMPUTE_WH
  SCHEMA = PUBLIC
  ROLE = ACCOUNTADMIN
  COMMENT = 'TPC-H Medallion dbt project';

-- Verify creation
SHOW DBT PROJECTS;
```

---

## Step 4: Execute dbt Project

### In Snowflake UI (Snowsight)

1. Navigate to **Projects** → **dbt** → Select your project
2. Click **Develop** to access the IDE
3. Install dependencies:
   ```
   dbt deps
   ```

4. Verify configuration:
   ```
   dbt debug
   ```

5. Build all models:
   ```
   dbt build
   ```

6. Run specific layers:
   ```bash
   dbt run --select tag:bronze
   dbt run --select tag:silver
   dbt run --select tag:gold
   ```

---

## Additional Resources

- [dbt Documentation](https://docs.getdbt.com/)
- [Snowflake Documentation](https://docs.snowflake.com/en/user-guide/data-engineering/dbt-projects-on-snowflake)
- [dbt + Snowflake Best Practices](https://docs.snowflake.com/en/user-guide/ecosystem-dbt)
- [TPC-H Benchmark Specification](http://www.tpc.org/tpch/)