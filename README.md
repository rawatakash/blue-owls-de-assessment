**Blue Owls Data Engineering Assessment**
**Overview**

This project implements a resilient data ingestion pipeline for e-commerce data exposed via internal APIs. The pipeline is designed to handle real-world challenges such as intermittent API failures, authentication issues, rate limiting, and data quality inconsistencies.

The solution follows a medallion architecture approach, with a focus on building a robust Bronze layer that ingests raw data reliably and idempotently. Than Silver layer handled null values and datatypes, Star schema will be created in gold layer having fact dimension tables.

**Architecture**

The pipeline ingests data from the following API endpoints:

orders
order_items
customers
products
sellers
payments

Data is extracted incrementally (from 2018-07-01 onward), validated, and stored in the Bronze layer as CSV files.

**Bronze Layer Structure**
bronze/
  orders/
  order_items/
  customers/
  products/
  sellers/
  payments/
  quarantine/
manifest.json
Each record includes:

_ingested_at: timestamp of ingestion
_source_endpoint: source API endpoint

****Key Technical Decisions
1. Centralized API Client****

A reusable API client was implemented to handle:

Authentication (Bearer token)
Token refresh on expiration (401 responses)
Retry logic for transient failures

This avoids duplication and ensures consistent handling of API interactions across all endpoints.

****2. Retry and Resilience Strategy**

The API is intentionally unreliable, so resilience is critical.**

500 (server errors) → retried using exponential backoff
429 (rate limiting) → retried with delay
401 (unauthorized) → token refreshed automatically

**3. Idempotency Approach**

The pipeline is designed to be idempotent using a manifest file.

Each successfully ingested page is recorded in manifest.json
On re-run, previously processed pages are skipped
This ensures no duplicate data is written to Bronze

This approach was chosen over deduplication at write time because:

It avoids expensive file scans
It ensures deterministic ingestion behavior

**4. Pagination Handling**

All endpoints support pagination. The pipeline:

Iterates through pages sequentially
Stops when an empty response is returned
Tracks processed pages in the manifest

**Production Considerations (Azure / Microsoft Fabric)**

If deployed in a production environment, the following improvements would be implemented:

**Orchestration**
Azure Data Factory or Fabric Pipelines for scheduling and dependency management
**Storage**
Bronze layer stored in Data Lake using Parquet/Delta format
Partitioning based on ingestion date for performance
**Security**
Credentials stored in Azure Key Vault
Managed identities for secure access
**Monitoring**
Logging integrated with Azure Monitor / Log Analytics
Alerts for pipeline failures and retry exhaustion
**CI/CD**
GitHub Actions or Azure DevOps pipelines for automated deployment
Environment-based configurations (dev, test, prod)
**Cost Optimization**
Incremental ingestion to reduce API calls
Efficient file formats and partitioning
Controlled retry limits to avoid excessive compute usage

  
