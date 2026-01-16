# 🏭 Azure Sales ETL Pipeline

## 📋 Projekt-Übersicht

Automatisierte ETL-Pipeline zur Verarbeitung von Sales- und Product-Daten in Azure Data Factory.

---

## 🏗️ Architektur

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│   Raw Data      │      │   Data Flows     │      │    Staging      │
│   (Blob)        │─────>│   (Transform)    │─────>│   (Parquet)     │
└─────────────────┘      └──────────────────┘      └─────────────────┘
  • CSV/JSON              • Cleaning                 • Optimiert
  • Unstrukturiert        • Filtering                • Komprimiert
                          • Enrichment               • Partitioniert
```

---

## 📁 Datenquellen

### Input (Blob Storage: `raw/`)
| Datei | Format | Beschreibung |
|-------|--------|--------------|
| `sales_data.csv` | CSV | Verkaufstransaktionen |
| `products_api_response.json` | JSON | Produktkatalog (API Response) |
| `customers_dimension.csv` | CSV | Kundenstammdaten |

### Output (Blob Storage: `staging/`)
| Datei | Format | Beschreibung |
|-------|--------|--------------|
| `sales_cleaned.parquet` | Parquet | Bereinigte Sales-Daten |
| `products_active.parquet` | Parquet | Aktive Produkte (gefiltert) |

---

## 🔧 Komponenten

### Linked Services
- **LS_BlobStorage_Sales**: Azure Blob Storage Verbindung

### Datasets
| Name | Typ | Pfad |
|------|-----|------|
| DS_Sales_CSV | DelimitedText | raw/sales_data.csv |
| DS_Products_JSON | JSON | raw/products_api_response.json |
| DS_Sales_Cleaned_Parquet | Parquet | staging/sales_cleaned.parquet |
| Parquet1 | Parquet | staging/products_active.parquet |

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

**Transformationen:**
- Duplikate entfernen (basierend auf order_id)
- Null-Werte in kritischen Feldern filtern
- Load-Timestamp hinzufügen

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

**Transformationen:**
- JSON-Struktur flatten
- Nur aktive Produkte behalten
- Parquet-Optimierung

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

**Execution:** Beide Data Flows laufen parallel für maximale Performance

---

## ⚙️ Trigger

### 1. Schedule Trigger
- **Name:** TR_Daily_Sales_ETL
- **Frequenz:** Täglich um 02:00 Uhr
- **Zweck:** Regelmäßige Datenverarbeitung

### 2. Event Trigger (optional)
- **Name:** TR_OnNewFile_Sales
- **Event:** Neue Datei in `raw/` Ordner
- **Zweck:** Echtzeit-Verarbeitung bei Upload

---

## 📊 Monitoring

### Pipeline Runs
- **Location:** ADF Studio → Monitor → Pipeline runs
- **Metriken:**
  - Success Rate
  - Average Duration
  - Failed Runs

### Alerts
- **Email-Benachrichtigung** bei Pipeline-Fehler
- **Azure Monitor** Integration
- **Logic App** für erweiterte Notifications

---

## 🚀 Deployment

### Voraussetzungen
- Azure Subscription
- Azure Data Factory Instance
- Azure Blob Storage Account
- Git Repository (GitHub)

### Setup-Schritte

1. **Repository klonen:**
   ```bash
   git clone https://github.com/[username]/azure-sales-etl-pipeline.git
   ```

2. **ADF mit Git verbinden:**
   - ADF Studio → Manage → Git configuration
   - Repository: azure-sales-etl-pipeline
   - Branch: main

3. **Linked Service konfigurieren:**
   - Update `LS_BlobStorage_Sales` mit deinem Storage Account
   - Connection String anpassen

4. **Testdaten hochladen:**
   ```bash
   az storage blob upload-batch      --account-name <storage-account>      --destination <container>/raw      --source ./sample-data
   ```

5. **Pipeline testen:**
   - Debug ausführen
   - Output prüfen
   - Staging-Dateien validieren

6. **Publish:**
   - Validate all
   - Publish all

---

## 🧪 Testing

### Debug Mode
```bash
# In ADF Studio
1. Öffne Pipeline: PL_Sales_ETL_Demo
2. Klicke "Debug"
3. Warte auf Completion (~3-5 Min)
4. Prüfe Output Tab
```

### Validierung
- ✅ Beide Activities: Status = Succeeded
- ✅ Parquet-Dateien in `staging/` vorhanden
- ✅ Keine Fehler im Output Log

---

## 📈 Performance

| Metrik | Wert |
|--------|------|
| Durchschnittliche Laufzeit | 3-5 Minuten |
| Parallelisierung | 2 Data Flows gleichzeitig |
| Compute Size | Small (4 vCores) |
| Datenvolumen | ~1-10 MB (Test), skalierbar |

---

## 🔐 Security

- **Managed Identity** für Storage-Zugriff
- **Key Vault** für Secrets (empfohlen)
- **RBAC** für ADF-Zugriff
- **Private Endpoints** (optional, für Prod)

---

## 🛠️ Troubleshooting

### Problem: Pipeline schlägt fehl
**Lösung:**
1. Monitor → Pipeline runs → Klick auf Failed run
2. Prüfe Error Message
3. Validiere Input-Daten
4. Prüfe Linked Service Connection

### Problem: Parquet-Dateien fehlen
**Lösung:**
1. Prüfe Sink-Konfiguration im Data Flow
2. Validiere Storage Account Permissions
3. Prüfe Container & Pfad

### Problem: Data Flow langsam
**Lösung:**
1. Erhöhe Compute Size (Medium/Large)
2. Aktiviere Partitioning
3. Optimiere Transformationen

---

## 📚 Ressourcen

- [Azure Data Factory Dokumentation](https://docs.microsoft.com/azure/data-factory/)
- [Data Flow Best Practices](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-performance)
- [Parquet Format Guide](https://parquet.apache.org/docs/)

---

## 👥 Team

- **Entwickler:** Vladislav Bogac
- **Projekt:** Azure Sales ETL Pipeline
- **Datum:** Januar 2026

---

## 📝 Changelog

### v1.0.0 (2026-01-09)
- ✅ Initial Release
- ✅ Sales & Products Data Flows
- ✅ Pipeline mit Parallel Execution
- ✅ Schedule Trigger
- ✅ Monitoring & Alerts
- ✅ Git Integration

---

## 📄 Lizenz

MIT License - siehe LICENSE Datei

