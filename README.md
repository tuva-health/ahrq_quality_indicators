[![Apache License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
![dbt version](https://img.shields.io/badge/dbt-1.10.5%20to%202.x-FF694B?logo=dbt)

# AHRQ Quality Indicators

The AHRQ Quality Indicators package calculates Prevention Quality Indicators
(PQIs) from claims data standardized by Tuva Core. PQIs are population-level
measures developed by the [Agency for Healthcare Research and
Quality](https://qualityindicators.ahrq.gov/measures/PQI_TechSpec.aspx) that
use inpatient administrative data to identify potentially avoidable
hospitalizations and possible access-to-care concerns.

This package implements PQIs 01, 03, 05, 07, 08, 11, 12, 14, 15, and 16. It
produces encounter-level numerator and exclusion records alongside annual
population denominators and rates.

## Outputs

By default, public models are built in the `ahrq_quality_indicators` schema.
Set `tuva_schema_prefix` in the root project to prepend a shared schema prefix.

| Relation | Description |
| --- | --- |
| `pqi_summary` | Encounter-level PQI results with measure and encounter context for reporting. |
| `pqi_rate` | Numerator, denominator, and rate per 100,000 by PQI, year, and data source. |
| `pqi_num_long` | Long-format numerator records by PQI. |
| `pqi_denom_long` | Long-format annual denominator records by PQI. |
| `pqi_exclusion_long` | Long-format encounter exclusions and exclusion reasons by PQI. |

Common uses include monitoring potentially preventable inpatient utilization,
comparing PQI rates across populations or years, and drilling from aggregate
rates into qualifying encounters and exclusions.

## Prerequisites and dependency ownership

Use this package in a dbt project that already installs a compatible version
of Tuva Core and supplies Tuva's standardized claims models. In a normal Tuva
deployment, the connector is the root dbt project and owns the Tuva Core
dependency. This package therefore does not install or pin Tuva Core itself;
it expects these relations to exist in the shared dbt graph:

- `core__condition`
- `core__encounter`
- `core__member_month`
- `core__patient`
- `core__procedure`
- `terminology__calendar`

The package requires dbt `>=1.10.5,<3.0.0`. Claims processing must be enabled
with the native YAML boolean `claims_enabled: true`. Quoted boolean strings
are not supported by Tuva Core's configuration contract.

## Installation

Once this package is listed on the dbt Package Hub, add it to the root
project's `packages.yml`:

```yaml
packages:
  - package: tuva-health/ahrq_quality_indicators
    version: 0.1.0
```

Before Hub registration is complete, install the same release directly from
GitHub:

```yaml
packages:
  - git: "https://github.com/tuva-health/ahrq_quality_indicators.git"
    revision: v0.1.0
```

Then resolve dependencies and build the package from the root project:

```shell
dbt deps
dbt build --select package:ahrq_quality_indicators
```

The package selection includes its two seed nodes. Keep them in the first
build so the released measure and value-set assets are loaded before dependent
models run.

## Configuration

Configure package behavior in the root project's `dbt_project.yml`:

```yaml
vars:
  claims_enabled: true
  # Optional: produces <prefix>_ahrq_quality_indicators.
  tuva_schema_prefix: analytics
```

| Variable | Default | Purpose |
| --- | --- | --- |
| `claims_enabled` | `false` | Enables the package's claims-based models. Use an unquoted YAML boolean. |
| `tuva_schema_prefix` | unset | Prefixes the output schema when supplied. |
| `tuva_last_run` | dbt run start time in UTC | Timestamp written to public outputs. |
| `ahrq_quality_indicators_data_asset_version` | `1.0.0` | Selects the independently versioned public PQI asset snapshot. |

## Data assets

The checked-in files under `seeds/` are header-only dbt loader contracts. At
build time, Tuva Core's shared seed loader retrieves the released contents
from:

```text
s3://tuva-public-resources/data-marts/ahrq-quality-indicators/1.0.0/
```

The same snapshot is mirrored to GCS and Azure. Package code and data assets
have independent version numbers; the `0.1.0` package release intentionally
uses the `1.0.0` data-asset snapshot. Change the asset-version variable only
when testing another published, compatible snapshot.

## Supported platforms

The package targets the same warehouse families as Tuva Core: Snowflake,
BigQuery, Databricks, Microsoft Fabric, Redshift, and DuckDB. Use a dbt adapter
and Tuva connector version supported by Tuva Core. Cross-database parsing and
the package's focused unit coverage are part of release validation; downstream
connector behavior should still be validated in your own environment.

## Documentation and support

- Review column-level contracts in
  [`models/ahrq_quality_indicators/pqi/pqi_models.yml`](models/ahrq_quality_indicators/pqi/pqi_models.yml).
- Learn more about Tuva at [The Tuva Project](https://thetuvaproject.com/).
- Report bugs or request enhancements in [GitHub
  Issues](https://github.com/tuva-health/ahrq_quality_indicators/issues).
- Join the [Tuva community on
  Slack](https://join.slack.com/t/thetuvaproject/shared_invite/zt-4663yf7du-MUIbAJPxHD65byDtAHwSyg).

Contributions are welcome through pull requests. Please describe the behavior
being changed and include the narrowest dbt unit or data test that demonstrates
it. Validate changes from a consuming root project that installs Tuva Core.

This project is licensed under the [Apache License 2.0](LICENSE).
