# Azure Sales ETL Pipeline

## Project Overview

Automated ETL pipeline for processing sales and product data in Azure Data Factory.

---

## Architecture

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│   Raw Data      │      │   Data Flows     │      │    Staging      │
│   (Blob)        │─────>│   (Transform)    │─────>│   (Parquet)     │
└─────────────────┘      └──────────────────┘      └─────────────────┘
  • CSV/JSON              • Cleaning                 • Optimized
  • Unstructured          • Filtering                • Compressed
                          • Enrichment               • Partitioned
```

---

## Data Sources

### Input (Blob Storage: `raw/`)

| File                         | Format | Description                    |
| ---------------------------- | ------ | ------------------------------ |
| `sales_data.csv`             | CSV    | Sales transactions             |
| `products_api_response.json` | JSON   | Product catalog (API response) |
| `customers_dimension.csv`    | CSV    | Customer master data           |

### Output (Blob Storage: `staging/`)

| File                      | Format  | Description                |
| ------------------------- | ------- | -------------------------- |
| `sales_cleaned.parquet`   | Parquet | Cleaned sales data         |
| `products_active.parquet` | Parquet | Active products (filtered) |

---

## Components

### Linked Services

* **LS_BlobStorage_Sales**: Azure Blob Storage connection

### Datasets

| Name                     | Type          | Path                            |
| ------------------------ | ------------- | ------------------------------- |
| DS_Sales_CSV             | DelimitedText | raw/sales_data.csv              |
| DS_Products_JSON         | JSON          | raw/products_api_response.json  |
| DS_Sales_Cleaned_Parquet | Parquet       | staging/sales_cleaned.parquet   |
| Parquet1                 | Parquet       | staging/products_active.parquet |

### Data Flows

#### 1. DF_Clean_Sales

```
Source (CSV)
    ↓
RemoveDuplicates (Aggregate)
    ↓
FilterNulls (Filter: !isNull(order_id))
    ↓
AddLoadDate (Derived Column: currentTimestamp())
    ↓
Sink (Parquet)
```

**Transformations:**

* Remove duplicates (based on order_id)
* Filter null values in critical fields
* Add load timestamp

#### 2. DF_Clean_Products

```
Source (JSON)
    ↓
FlattenData (Flatten: products array)
    ↓
FilterActive (Filter: status == 'active')
    ↓
Sink (Parquet)
```

**Transformations:**

* Flatten JSON structure
* Keep only active products
* Parquet optimization

### Pipeline

**PL_Sales_ETL_Demo**

```
┌──────────────────────────┐
│ Run Sales Cleaning       │  (parallel)
└──────────────────────────┘

┌──────────────────────────┐
│ Run Products Cleaning    │  (parallel)
└──────────────────────────┘
```

**Execution:** Both data flows run in parallel for maximum performance

---

## Triggers

### 1. Schedule Trigger

* **Name:** TR_Daily_Sales_ETL
* **Frequency:** Daily at 02:00
* **Purpose:** Regular data processing

### 2. Event Trigger (optional)

* **Name:** TR_OnNewFile_Sales
* **Event:** New file in `raw/` folder
* **Purpose:** Real-time processing on upload

---

## Monitoring

### Pipeline Runs

* **Location:** ADF Studio → Monitor → Pipeline runs
* **Metrics:**

  * Success rate
  * Average duration
  * Failed runs

### Alerts

* **Email notification** on pipeline failure
* **Azure Monitor** integration
* **Logic App** for advanced notifications

---

## Deployment

### Prerequisites

* Azure subscription
* Azure Data Factory instance
* Azure Blob Storage account
* Git repository (GitHub)

### Setup Steps

1. **Clone repository:**

   ```bash
   git clone https://github.com/[username]/azure-sales-etl-pipeline.git
   ```

2. **Connect ADF with Git:**

   * ADF Studio → Manage → Git configuration
   * Repository: azure-sales-etl-pipeline
   * Branch: main

3. **Configure linked service:**

   * Update `LS_BlobStorage_Sales` with your storage account
   * Adjust connection string

4. **Upload test data:**

   ```bash
   az storage blob upload-batch      --account-name <storage-account>      --destination <container>/raw      --source ./sample-data
   ```

5. **Test pipeline:**

   * Run debug
   * Check output
   * Validate staging files

6. **Publish:**

   * Validate all
   * Publish all

---

## Testing

### Debug Mode

```bash
# In ADF Studio
1. Open pipeline: PL_Sales_ETL_Demo
2. Click "Debug"
3. Wait for completion (~3–5 min)
4. Check Output tab
```

### Validation

* Both activities: Status = Succeeded
* Parquet files present in `staging/`
* No errors in output log

---

## Performance

| Metric          | Value                       |
| --------------- | --------------------------- |
| Average runtime | 3–5 minutes                 |
| Parallelization | 2 data flows simultaneously |
| Compute size    | Small (4 vCores)            |
| Data volume     | ~1–10 MB (test), scalable   |

---

## Security

* **Managed identity** for storage access
* **Key Vault** for secrets (recommended)
* **RBAC** for ADF access
* **Private endpoints** (optional, for prod)

---

## Troubleshooting

### Issue: Pipeline fails

**Solution:**

1. Monitor → Pipeline runs → Click on failed run
2. Check error message
3. Validate input data
4. Check linked service connection

### Issue: Parquet files missing

**Solution:**

1. Check sink configuration in data flow
2. Validate storage account permissions
3. Check container & path

### Issue: Data flow is slow

**Solution:**

1. Increase compute size (Medium/Large)
2. Enable partitioning
3. Optimize transformations

---

## Resources

* [Azure Data Factory Documentation](https://docs.microsoft.com/azure/data-factory/)
* [Data Flow Best Practices](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-performance)
* [Parquet Format Guide](https://parquet.apache.org/docs/)
