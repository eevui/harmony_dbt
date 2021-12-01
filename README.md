# Harmony DBT Project

Curated SQL Views and Metrics for the Harmony Blockchain.

What's Harmony? Learn more [here](https://www.harmony.one/)

## Setup

1. [PREREQUISITE] Download [Docker for Desktop](https://www.docker.com/products/docker-desktop).
2. Create a `.env` file with the following contents (note `.env` will not be commited to source):

```
SF_ACCOUNT="zsniary-metricsdao"
SF_USERNAME="<your_metrics_dao_snowflake_username>"
SF_PASSWORD="<your_metrics_dao_snowflake_password>"
SF_REGION="us-east-1"
SF_DATABASE="HARMONY"
SF_WAREHOUSE="DEFAULT"
SF_ROLE="PUBLIC"
SF_SCHEMA="DEV"
```

3. New to DBT? It's pretty dope. Read up on it [here](https://www.getdbt.com/docs/)

## Getting Started Commands

Run the follow commands from inside the Harmony directory (**you must complete the Getting Started steps above^^**)

### DBT Environment

`make dbt-console`
This will mount your local harmony directory into a dbt console where dbt is installed.

### DBT Project Docs

`make dbt-docs`
This will compile your dbt documentation and launch a web-server at http://localhost:8080

## Project Overview

`/models` - this directory contains SQL files as Jinja templates. DBT will compile these templates and wrap them into create table statements. This means all you have to do is define SQL select statements, while DBT handles the rest. The snowflake table name will match the name of the sql model file.

`/macros` - these are helper functions defined as Jinja that can be injected into your SQL models.

`/tests` - custom SQL tests that can be attached to tables.

## Background on Data

`FLIPSIDE.BRONZE_CHAINWALKERS.HARMONY_BLOCKS`
Flipside Crypto has provided raw block data for Harmony. Details on the data:

1. This is near-real time. Blocks land in this table within 3-5 minutes of being minted.
2. The table is a read-only data share in the Metrics DAO Snowflake account under the database `FLIPSIDE`.
3. The table is append-only, meaning that duplicates can exist if blocks are re-processed. The injested_at timestamp should be used to retrieve only the most recent block. Macros exist `macros/chainwalkers.sql` to handle this. See `models/core/blocks.sql` or `/models/core/txs.sql` for an example.
4. Tx logs are decoded where an ABI exists.

### Table Structure:

| Column          | Type         | Description                                                            |
| --------------- | ------------ | ---------------------------------------------------------------------- |
| offset_id       | NUMBER(38,0) | Synonmous with block_id for Harmony                                    |
| block_id        | NUMBER(38,0) | The height of the chain this block corresponds with                    |
| block_timestamp | TIMESTAMP    | The time the block was minted                                          |
| network         | VARCHAR      | The blockchain network (i.e. mainnet, testnet, etc.)                   |
| chain_id        | VARCHAR      | Synonmous with blockchain name for Harmony                             |
| tx_count        | NUMBER(38,0) | The number of transactions in the block                                |
| header          | json variant | A json queryable column containing the blocks header information       |
| tx              | array        | An array of json queryable objects containing each tx and decoded logs |
| ingested_at     | TIMESTAMP    | The time this data was ingested into the table by Snowflake            |

## Target Database, Schemas and Tables

Data in this DBT project is written to the `HARMONY` database in MetricsDAO.

This database has 2 schemas, one for `DEV` and one for `PROD`. As a contributer you have full permission to write to the `DEV` schema. However the `PROD` schema can only be written to by Metric DAO's DBT Cloud account. The DBT Cloud account controls running / scheduling models against the `PROD` schema.

## Branching / PRs

When conducting work please branch off of main with a description branch name and generate a pull request. At least one other individual must review the PR before it can be merged into main. Once merged into main DBT Cloud will run the new models and output the results into the `PROD` schema.

When creating a PR please include the following details in the PR description:

1. List of Tables Created or Modified
2. Description of changes.
3. Implication of changes (if any).

## More DBT Resources:

- Learn more about dbt [in the docs](https://docs.getdbt.com/docs/introduction)
- Check out [Discourse](https://discourse.getdbt.com/) for commonly asked questions and answers
- Check out [the blog](https://blog.getdbt.com/) for the latest news on dbt's development and best practices
