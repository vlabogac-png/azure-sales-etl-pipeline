# 🚀 Azure Sales ETL Pipeline

ETL-Pipeline für Sales & Products Daten mit Azure Data Factory

## 📊 Architektur
Raw Data (Blob Storage)
├── sales_data.csv
└── products_api_response.json
↓
[Azure Data Factory]
├── DF_Clean_Sales
└── DF_Clean_Products
↓
Staging (Blob Storage)
├── sales_cleaned.parquet
└── products_active.parquet


## 🔄 Pipelines

- **PL_Sales_ETL_Demo**: Hauptpipeline für Sales & Products ETL

## 📦 Data Flows

### DF_Clean_Sales
- Duplikate entfernen
- Null-Werte filtern
- LoadDate hinzufügen

### DF_Clean_Products
- JSON flattening
- Nur aktive Produkte

## 🛠️ Technologien

- Azure Data Factory
- Azure Blob Storage
- Parquet Format
Commit changes
