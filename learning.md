“I built an end-to-end data pipeline using a Medallion Architecture on Databricks with Delta Lake.

Raw e-commerce event data is ingested into a Bronze layer as immutable Delta tables, where no filtering or cleansing happens to preserve auditability and support reprocessing.

From Bronze, data flows into a Silver layer where I enforce schema, standardize categorical fields, apply data quality rules, and deduplicate events using business keys. Invalid records are written to a separate reject table with explicit rejection reasons so there’s no silent data loss. Silver processing is incremental and uses event-time watermarks with Delta MERGE to ensure idempotency and handle late-arriving data.

The Gold layer consists of analytics-ready data marts aligned with business use cases like daily revenue, user funnels, product performance, and growth metrics. Gold tables are incrementally refreshed using event-date partition overwrites and written using saveAsTable to keep Unity Catalog metadata in sync.

Overall, the design mirrors how production analytics platforms are built: raw data preservation, trusted intermediate datasets, and optimized business-facing tables.”
