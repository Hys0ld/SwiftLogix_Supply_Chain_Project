# SwiftLogix Weather Data Pipeline

This project builds a simple **Medallion Architecture** in Databricks using weather alerts from the **National Weather Service API**.

The pipeline processes active weather alerts and transforms them into analytics-ready Delta tables.

**API → Bronze → Silver → Gold**

---

## Architecture

```text
National Weather Service API
            │
            ▼
        Bronze
     Raw API JSON
            │
            ▼
        Silver
 Cleaned & transformed data
            │
            ▼
         Gold
 Fact & dimension tables
```

---

## 1. Data Source

Weather data is retrieved from the National Weather Service API:

```text
https://api.weather.gov/alerts/active
```

The API provides active weather alerts including:

* Event
* Severity
* Status
* Headline
* Location
* Start/end times
* Geographic codes

---

## 2. Bronze Layer

The raw API response is stored in:

```text
bronze.swiftlogix.bronze_weather
```

The raw JSON is saved along with a timestamp.

```text
bronze_weather
├── raw_data
└── timestamp
```

The raw response is preserved so it can be processed later.

---

## 3. Silver Layer

The Bronze JSON is parsed and transformed using PySpark.

The Silver table is:

```text
silver.swiftlogix.silver_weather
```

Main transformations include:

* Parse the API JSON
* Flatten nested properties
* Extract weather alert information
* Process geographic information
* Filter weather alerts
* Handle missing end times
* Create hazard types
* Add operational status

### Weather Filtering

Only alerts with:

```text
status = Actual
```

and severity of:

```text
Extreme
Severe
Moderate
```

are included.

### Operational Status

Weather severity is converted into an operational status:

| Severity | Operational Status |
| -------- | ------------------ |
| Extreme  | Suspended          |
| Severe   | Delayed            |
| Moderate | Normal             |

---

## 4. Location Enrichment

Weather alerts contain geographic codes such as:

```text
SAME
UGC
```

These codes are used to connect the weather alerts with location data.

Additional location data is loaded from Databricks Volumes and used to provide geographic information such as:

* City
* State
* Latitude
* Longitude
* FIPS

---

## 5. Gold Layer

The Gold layer contains analytics-ready Delta tables.

Schema:

```text
gold.swiftlogix
```

### Fact Table

```text
gold.swiftlogix.fact_active_hazards
```

Contains the active weather hazards and their operational impact.

### Dimension Tables

```text
gold.swiftlogix.dim_hazard_type
gold.swiftlogix.dim_warehouse_hub
gold.swiftlogix.dim_same_code_location
```

These tables provide additional information about:

* Hazard types
* Warehouse/city locations
* Geographic codes

---

## 6. Data Model

```text
                    ┌─────────────────────┐
                    │  dim_hazard_type    │
                    └──────────┬──────────┘
                               │
                               ▼
┌──────────────────┐    ┌─────────────────────┐
│ dim_warehouse_hub│───►│ fact_active_hazards  │
└──────────────────┘    └──────────┬──────────┘
                                   │
                                   ▼
                         ┌─────────────────────┐
                         │dim_same_code_location│
                         └─────────────────────┘
```

---

## 7. API Fault Tolerance

The API request includes basic fault tolerance.

The pipeline uses:

* 30-second request timeout
* Up to 3 attempts
* Retry delays
* Logging
* HTTP status validation

If the API continues to fail, the Databricks notebook exits with a failure message.

---

## 8. Technologies

* Databricks
* PySpark
* Python
* Delta Lake
* REST API
* JSON
* Spark SQL
* Medallion Architecture
* Dimensional Modeling

---

## 9. Final Databricks Structure

```text
bronze
└── swiftlogix
    └── bronze_weather

silver
└── swiftlogix
    └── silver_weather

gold
└── swiftlogix
    ├── fact_active_hazards
    ├── dim_hazard_type
    ├── dim_warehouse_hub
    └── dim_same_code_location
```

---

## 10. Project Goal

The goal of this project is to demonstrate how an external API can be integrated into a Databricks data pipeline and transformed into structured data for analytics.

The project demonstrates:

* API ingestion
* API fault tolerance
* Raw data storage
* JSON processing
* Data cleaning
* Geographic enrichment
* Fact and dimension tables
* Delta Lake
* Medallion Architecture
