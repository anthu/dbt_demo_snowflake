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
  enabled = true;

-- Create network rules for dbt package downloads
CREATE OR REPLACE NETWORK RULE dbt_hub_network_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = ('hub.getdbt.com', 'codeload.github.com');

-- Create external access integration for dbt deps
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION dbt_external_access_integration
  ALLOWED_NETWORK_RULES = (dbt_hub_network_rule)
  ENABLED = TRUE
  COMMENT = 'External access integration for dbt package dependencies';
```
---

## Step 2: Create dbt Project

### Option A: Snowflake UI (Recommended)

1. Navigate to **Projects** → **Workspaces** in Snowsight
2. Click on the Workspace Selector at the very to left
3. Click **From Git Repository**
3. Configure:
   - **Repository URL**: `https://github.com/anthu/dbt_demo_snowflake`
   - **Workspace Name**: `TPC-H dbt Workspace`
   - **API Integration**: `GITHUB_INTEGRATION`
4. Click **Create**

#### Then deploy the dbt Project
1. Click on the very top right Corner **Connect**
2. Click **Deploy dbt project**
3. Configure:
   - **Database**: `TPCH_MEDALLION_DBT_DB`
   - **Schema**: `PUBLIC`
   - **Select or create dbt project**: Click **Create dbt prjoject**
   - **Name**: `TPCH_DBT_REPO`
4. Click **Deploy**

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

-- Create dbt project
CREATE OR REPLACE DBT PROJECT TPCH_MEDALLION_DBT_DB.PUBLIC.tpch_medallion_dbt_project
  FROM '@TPCH_MEDALLION_DBT_DB.PUBLIC.TPCH_DBT_REPO/branches/main'
  COMMENT = 'TPC-H Medallion dbt project';

-- Verify creation
SHOW DBT PROJECTS;
```

**Security Note**: Use Snowflake secrets for credential management. Never commit credentials to version control.

---

## Step 3: Execute dbt Project

### In Snowflake UI (Snowsight)

1. Navigate to **Projects** → **dbt** → Select your project
2. Click **Develop** to access the IDE
3. Install dependencies:
   ```
   dbt deps
   ```
Note: when running `dbt deps`, use the previously created `dbt_external_access_integration` for External Network Access.

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
- [dbt Projects in Snowflake Documentation](https://docs.snowflake.com/en/user-guide/data-engineering/dbt-projects-on-snowflake)
- [dbt + Snowflake Best Practices](https://docs.snowflake.com/en/user-guide/ecosystem-dbt)
- [TPC-H Benchmark Specification](http://www.tpc.org/tpch/)