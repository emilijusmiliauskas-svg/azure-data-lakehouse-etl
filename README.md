# Bee Haven: Azure Data Lakehouse for Environmental Analytics

## Project Overview
This project involves the design and implementation of an end-to-end **Azure Data Lakehouse** designed to process large-scale environmental and biological sensor data. The objective was to build an automated pipeline that ingests raw hive activity records and enriches them with historical climate data to enable a "Single Source of Truth" for ecological research.

The architecture focuses on high-performance storage, cost-effective computation, and production-grade orchestration using the **Medallion (Bronze/Silver/Gold) Design Pattern**.

---

## Skills Demonstrated
* **Cloud Architecture:** Designing scalable environments using Azure Resource Groups and ADLS Gen2.
* **Data Orchestration:** Building complex, parameterized ETL pipelines in Azure Data Factory (ADF).
* **Big Data Transformation:** Utilizing Synapse Analytics for Python-based cleaning and Parquet optimization.
* **API Integration:** Dynamic data enrichment using the BrightSky REST API and JSON parsing.
* **Cloud Security:** Implementing Role-Based Access Control (RBAC) and Service Principals.

---

## Technical Infrastructure

### 1. Cloud Environment & Security
* **Storage:** **Azure Data Lake Storage (ADLS) Gen2** with a hierarchical namespace for optimized big data file operations.
* **Security:** Implemented **Azure RBAC** to secure the environment, specifically utilizing the *Storage Blob Data Contributor* role for Synapse and ADF access.
* **Networking:** Leveraged the **ABFSS (Azure Blob File System Secure)** protocol for high-throughput, secure data access within the notebooks.

### 2. Orchestration & Ingestion (Bronze Layer)
Raw data (CSV and JSON API responses) is managed via **Azure Data Factory (ADF)**.
* **Dynamic Control Flow:** Used **Get Metadata** and **ForEach** activities to create a flexible ingestion engine capable of processing variable batch sizes.
* **Automated Archiving:** Implemented logic to timestamp and version raw files upon ingestion, ensuring a permanent audit trail and preventing data loss.

### 3. Transformation & Enrichment (Silver Layer)
Processing is handled by **Azure Synapse Analytics** using Python/Pandas notebooks.
* **Timezone Normalization:** Standardized local timestamps to **UTC** (Europe/Berlin), programmatically handling Daylight Savings Time (DST) transitions.
* **Feature Engineering:** * **Directional Flow:** Split sequential sensor records into distinct `flow_in` and `flow_out` metrics using `.iloc` and `.merge` logic.
    * **Weather API Enrichment:** Dynamically calculated data date ranges to fetch precise historical weather context (Temp, Solar, Wind) from the **BrightSky API**.
* **Storage Optimization:** Converted row-based CSVs into compressed **Apache Parquet** files via the `pyarrow` engine, significantly reducing storage costs and scan times.

### 4. Analytical Modeling (Gold Layer)
The Gold layer provides a curated **Fact Table** for final analysis.
* **High-Dimensional Join:** Performed a comprehensive **Outer Join** strategy across 30+ features to align hive sensors with external weather metrics.
* **Semantic Resolution:** Programmatically resolved naming collisions between `temperature_hive` and `temperature_ambient`.
* **Memory Optimization:** Converted location strings to the `category` data type to optimize memory usage for large-scale queries.

---

## Automation & Scheduling
The entire lifecycle is orchestrated autonomously:
* **Chained Workflows:** Utilized **Execute Pipeline** activities to ensure the Silver-to-Gold transformation only triggers upon the success of the Bronze-to-Silver layer.
* **Batch Scheduling:** Configured a daily **Schedule Trigger (02:00 UTC)**. This specific timing ensures that all late-arriving data from the previous day is captured before the daily batch commences.

---

## Project Structure
- **notebooks/**: Core ETL logic.
  - `01_bronze_to_silver.ipynb`: Initial cleaning, UTC normalization, and Weather API integration.
  - `02_silver_to_gold.ipynb`: Multi-source joins, semantic resolution, and final fact table creation.
- **assets/**: Visual documentation of ADF orchestration, architecture diagrams, and trigger logic.
- **data_samples/**: Samples of raw vs. processed data structures.

---

## Tech Stack
* **Cloud:** Azure (ADLS Gen2, ADF, Synapse Analytics)
* **Languages:** Python (Pandas, NumPy, Requests, fsspec)
* **File Formats:** Parquet, JSON, CSV
* **API:** BrightSky Weather API
