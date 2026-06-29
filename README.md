# Fake News Detection Pipeline 🚨

**An enterprise-grade ETL pipeline for real-time fake news detection using the medallion architecture (Bronze/Silver/Gold layers)**

---

## 📋 Project Overview

This project demonstrates a **professional, production-ready data pipeline** that:

✅ Ingests articles from **NewsAPI** in real-time  
✅ Cleans and deduplicates data in the Silver layer  
✅ Detects fake news with ML-based scoring  
✅ Orchestrates jobs with error handling & logging  
✅ Implements **medallion architecture** (industry standard)  

**Perfect for:** Data engineers, ETL specialists, data architects

---

## 🏗️ Architecture

### Medallion Architecture (3 Layers)
┌─────────────────────────────────────────────────────────────┐

│                     NewsAPI (Source)                        │

└──────────────────────────────────┬──────────────────────────┘

│

┌──────────────▼──────────────┐

│  🥉 BRONZE LAYER            │

│  (Raw Ingestion)            │

│  - Raw articles from API    │

│  - No transformation        │

│  - 70+ articles stored      │

└──────────────┬──────────────┘

│

┌──────────────▼──────────────┐

│  🥈 SILVER LAYER            │

│  (Cleaning & Enrichment)    │

│  - Remove duplicates        │

│  - Handle nulls             │

│  - Add word count           │

│  - Normalize data           │

└──────────────┬──────────────┘

│

┌──────────────▼──────────────┐

│  🥇 GOLD LAYER              │

│  (Analytics Ready)          │

│  - Fake news scoring        │

│  - Confidence levels        │

│  - Ready for dashboard      │

└──────────────┬──────────────┘

│

┌──────────────▼──────────────┐

│  Dashboard / BI Tools       │

│  (Future Implementation)    │

└─────────────────────────────┘
---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| **Talend Open Studio** | ETL orchestration & data pipeline |
| **PostgreSQL** | Data warehouse (Bronze, Silver, Gold schemas) |
| **NewsAPI** | Real-time article ingestion |
| **SQL** | Data transformation & querying |
| **Git/GitHub** | Version control & collaboration |

---

## 📊 Jobs Overview

### 1. **JOB_BRONZE_NewsIngestion** 🥉
- Fetches articles from NewsAPI
- Stores raw data as-is
- **Output:** `bronze.raw_articles` (70+ rows)

**Components:** tRESTClient → tExtractJSONFields → tMap → tPostgresqlOutput

---

### 2. **JOB_SILVER_NewsProcessing** 🥈
- Reads from Bronze layer
- Removes duplicates by URL
- Filters null values
- Calculates word count
- **Output:** `silver.clean_articles` (deduplicated, cleaned)

**Components:** tDBInput → tMap → tFilterRow → tPostgresqlOutput

---

### 3. **JOB_GOLD_FakeNewsDetection** 🥇
- Reads from Silver layer
- Applies fake news scoring algorithm
- Calculates confidence levels
- **Output:** `gold.fake_news_results` (scored, ready for analysis)

**Scoring Logic:**
- Short articles (< 100 words) → Higher fake risk
- Sensational keywords ("shocking", "breaking") → Higher fake risk
- Unknown sources → Higher fake risk
- Score range: 0.0 to 1.0

**Components:** tDBInput → tMap (with scoring logic) → tPostgresqlOutput

---

### 4. **JOB_MASTER_Orchestrator** 🎯
- Master job that runs all 3 layers in sequence
- Error handling with tDie
- Logging with tLogCatcher
- Statistics capture with tStatCatcher

---

## 📦 Prerequisites

- **PostgreSQL 9+** installed and running
- **Talend Open Studio** (latest version)
- **Java** 8+ (for Talend)
- **Git** installed
- **NewsAPI key** (free from newsapi.org)

---

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/citoyenu/fake-news-detection-pipeline.git
cd fake-news-detection-pipeline
```

### 2. Set Up Database
```bash
psql -U postgres -f SQL/01_create_schemas.sql
psql -U postgres -f SQL/02_create_bronze_table.sql
psql -U postgres -f SQL/03_create_silver_table.sql
psql -U postgres -f SQL/04_create_gold_table.sql
```

### 3. Configure Talend
- Open Talend Open Studio
- Set context variables (api_key, db credentials, etc.)

### 4. Run the Pipeline
- Execute `JOB_MASTER_Orchestrator`
- Monitor console for messages
- Check PostgreSQL tables for results

---

## 📈 Results

**After running the pipeline:**

✅ **Bronze Layer:** 70+ raw articles ingested  
✅ **Silver Layer:** Deduplicated, cleaned articles with word counts  
✅ **Gold Layer:** Fake news scores with confidence levels  

---

## 📁 Project Structure
fake-news-detection-pipeline/

├── README.md

├── ARCHITECTURE.md

├── SETUP.md

├── LICENSE

├── .gitignore

├── SQL/

│   ├── 01_create_schemas.sql

│   ├── 02_create_bronze_table.sql

│   ├── 03_create_silver_table.sql

│   └── 04_create_gold_table.sql

├── Talend/

│   ├── JOB_BRONZE_NewsIngestion/

│   ├── JOB_SILVER_NewsProcessing/

│   ├── JOB_GOLD_FakeNewsDetection/

│   └── JOB_MASTER_Orchestrator/

└── docs/

└── screenshots/
---

## 💡 Key Features

✅ **Medallion Architecture** — Industry-standard data design  
✅ **Real-time Ingestion** — Fetch fresh articles from NewsAPI  
✅ **Data Quality** — Deduplication, null handling, validation  
✅ **ML-ready Data** — Scored and enriched for analysis  
✅ **Error Handling** — Professional error management  
✅ **Logging & Monitoring** — Comprehensive logging  
✅ **Statistics** — Performance tracking  
✅ **Professional Orchestration** — Master job coordinates everything  

---

## 🔐 Security

⚠️ **Important:** 
- Never commit passwords or API keys
- Use `.gitignore` to exclude sensitive files
- Store credentials in Talend context variables

---

## 🚦 Future Enhancements

- [ ] Real fake news detection API integration
- [ ] Python ML model for advanced scoring
- [ ] Tableau/Power BI dashboard
- [ ] Automated scheduling (every 30 minutes)
- [ ] Email alerts on high fake news detection
- [ ] Historical trend analysis
- [ ] Multi-language support

---

## 👤 Author

**Created by:** Iheb Citoyenu  
**GitHub:** [@citoyenu](https://github.com/citoyenu)  

---

## 📄 License

This project is licensed under the **MIT License**

---

**Status:** ✅ Production Ready  
**Last Updated:** June 29, 2026