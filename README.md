# AHRQ Quality Indicators

dbt package for the Tuva Project AHRQ Quality Indicators data mart.

## Data assets

Seed contents are stored under
`s3://tuva-public-resources/data-marts/ahrq-quality-indicators/<asset-version>/`
and mirrored to GCS and Azure. The checked-in CSV files contain only the
headers required by dbt.

`ahrq_quality_indicators_data_asset_version` selects the folder and defaults to
`1.0.0`. Package code and data assets are versioned independently and are
coordinated manually. Cloud manifests record the asset inventory, provenance,
and release status; dbt loads the configured path without reading them.
