# 🛍️ RetailBusiness360 — Microsoft Fabric Data Engineering Platform

[![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://learn.microsoft.com/en-us/fabric/)
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org/)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-0A6EBD?style=for-the-badge)](https://delta.io/)
[![Azure Event Hub](https://img.shields.io/badge/Azure%20Event%20Hub-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/en-us/products/event-hubs/)
[![KQL](https://img.shields.io/badge/KQL-512BD4?style=for-the-badge)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
[![SQL](https://img.shields.io/badge/SQL%20Warehouse-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)](https://learn.microsoft.com/en-us/fabric/data-warehouse/)

**Microsoft Fabric Data Engineering Project | Medallion Architecture | Multi-Source Retail Analytics | Real-Time Streaming**

---

## Project Summary

RetailBusiness360 is an end-to-end retail analytics data platform built on Microsoft Fabric. The platform brings together data from four distinct source systems — an on-premises ERP SQL Server, a cloud CRM, a logistics REST API, and a real-time point-of-sale event stream — into a single, governed Lakehouse using the Medallion Architecture pattern. The goal is to give retail operations teams, data analysts, and supply chain managers a reliable, daily-refreshed single source of truth covering sales, customers, inventory, and logistics across an international store network.

The platform processes over 172,000 records spanning 30 stores across six countries — the UK, USA, Germany, India, Australia, and UAE — and delivers analytics-ready Gold layer tables that power business intelligence dashboards for sales performance, customer behaviour, inventory health, and shipment KPIs.

---

## Architecture & Data Flow

The platform is structured around the Medallion Architecture, a staged design where data quality and analytical readiness increase progressively through three lake layers: Bronze, Silver, and Gold, all hosted on Microsoft Fabric's OneLake.

**Bronze** is the raw ingestion layer. Data from each source system lands here exactly as received — no transformations applied. ERP files arrive as date-stamped CSVs under structured folder paths; CRM data lands as Delta tables; logistics data is ingested as CSV from a REST API response; and real-time POS transactions flow in via Azure Event Hub into a KQL EventHouse. Bronze serves as an immutable audit trail, ensuring every byte from every source is preserved and traceable.

**Silver** is the cleaning and standardisation layer. A PySpark notebook reads all seven entities from Bronze and applies a systematic quality framework — type casting, date parsing, string normalisation, null-key removal, deduplication on primary keys, and business rule filtering (e.g. rejecting negative prices or zero-quantity transactions). Derived columns are added where needed, such as recalculating delivery days from ship and delivery dates when the source value is missing, or computing net stock movement from received and dispatched quantities. Every Silver table carries a `_silver_load_ts` audit timestamp for lineage tracking, and all tables are written as Delta Lake format for ACID compliance and time-travel support.

**Gold** is the business aggregation layer. A second PySpark notebook reads Silver tables as Delta and produces three dimensional models and six aggregated analytical tables, all written back as Delta Lake. The fact_sales table is built by enriching each transaction with product, customer, and store attributes, then deriving discount amounts and time partitions (year, month, quarter, week). From there, aggregations are computed across six analytical views: daily sales trends, product performance with revenue ranking, customer-level lifetime metrics, regional breakdowns by quarter, inventory health snapshots with reorder alerts, and shipment KPIs by delivery partner and status. A customer RFM segmentation (Champion, Loyal, Potential, New) is derived automatically from lifetime revenue, and a reorder flag is applied to any product-warehouse combination where stock on hand falls below 10% of stock received.

---

## Data Sources

The platform integrates four source system types, each representing a different enterprise integration pattern. The on-premises ERP SQL Server contributes product and inventory data, ingested daily at 6 PM using a dynamic pipeline that queries INFORMATION_SCHEMA to discover tables automatically — meaning no table names are hardcoded and new ERP tables are picked up without any pipeline changes. The cloud CRM system contributes customer profiles and interaction records, ingested daily at 6 AM from Lakehouse CSV files into the Fabric SQL Warehouse, followed by an automatic Dataflow Gen2 refresh. The logistics REST API delivers shipment data weekly via an HTTP GET call with RFC5988 pagination support and explicit 13-field column mapping to flatten the JSON response. Finally, real-time point-of-sale transactions flow through Azure Event Hub into a KQL EventHouse database, with a separate pipeline bridging the KQL table directly into the SQL Warehouse — providing near-real-time transaction visibility with auto-create table support at the sink.

---

## Pipelines & Orchestration

All four ingestion pipelines are defined as parameterised ARM JSON templates, making them portable and deployable to any Fabric workspace by swapping workspace and artifact IDs. Each pipeline is purpose-built for its source pattern: the ERP pipeline uses a Lookup activity to dynamically iterate all tables; the CRM pipeline uses a SetVariable and ForEach pattern over a fixed table list before triggering a dataflow refresh; the logistics pipeline maps REST response fields to Lakehouse columns; and the streaming pipeline copies directly from a KQL database table into a SQL Warehouse table with full schema mapping.

The event streaming path is complemented by a Python notebook (`EventStream_Trigger.ipynb`) that reads the sales transactions CSV from OneLake and replays it row by row into Azure Event Hub using the `azure-eventhub` SDK, simulating a live POS feed with a one-second delay between events — enabling end-to-end testing of the streaming pipeline without a live POS system.

---

## Key Engineering Outcomes

RetailBusiness360 demonstrates a complete, production-style data engineering solution on Microsoft Fabric. It covers all four major integration patterns found in enterprise retail — batch SQL extraction, file-based lakehouse ingestion, API polling, and real-time event streaming — within a single unified platform. The Medallion architecture enforces a clear separation between raw, cleaned, and analytical data, with each layer independently evolvable. The data quality framework applied at Silver ensures that every downstream analytical table rests on a foundation of type-safe, deduplicated, null-checked data. The Gold layer delivers six pre-aggregated tables and three dimensional models that are immediately consumable by BI tools via the Fabric SQL Warehouse endpoint, covering the full retail analytics surface — from daily revenue trends and product revenue rankings to customer RFM segments, regional quarterly performance, inventory reorder alerts, and carrier-level logistics SLAs.
