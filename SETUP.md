# Setup & Installation Guide

Complete step-by-step instructions to set up the Fake News Detection Pipeline locally.

---

## 📋 Prerequisites

### System Requirements
- **OS:** Windows 10+, macOS 10.13+, or Linux (Ubuntu 18.04+)
- **RAM:** Minimum 8GB (16GB recommended)
- **Disk Space:** 5GB free space

### Software Requirements

#### 1. PostgreSQL 12+ ✅
**Windows:**
- Download: https://www.postgresql.org/download/windows/
- Version: 12 or higher
- Installation: Keep default settings
- Port: 5432 (default)
- Password: Remember your `postgres` password!

**macOS:**
```bash
brew install postgresql@14
brew services start postgresql@14
```

**Linux (Ubuntu):**
```bash
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
sudo service postgresql start
```

**Verify installation:**
```bash
psql --version
psql -U postgres -c "SELECT version();"
```

---

#### 2. Talend Open Studio ✅
- Download: https://www.talend.com/products/talend-open-studio
- Version: 2023.06 or higher (free)
- Java: Comes with installer
- Installation: Standard setup

**Verify installation:**
- Launch Talend Open Studio
- Check Java version: Help → About

---

#### 3. Git ✅
**Windows:**
- Download: https://git-scm.com/download/win
- Installation: Use defaults
- Verify: `git --version`

**macOS:**
```bash
brew install git
git --version
```

**Linux:**
```bash
sudo apt-get install git
git --version
```

---

#### 4. NewsAPI Key ✅
1. Go to https://newsapi.org
2. Sign up (free tier available)
3. Copy your API key
4. **Keep it safe!** You'll need it later

---

## 🗂️ Step 1: Clone the Repository

```bash
# Navigate to your projects folder
cd ~/projects  # or Desktop, Documents, etc.

# Clone the repository
git clone https://github.com/citoyenu/fake-news-detection-pipeline.git

# Navigate into the project
cd fake-news-detection-pipeline

# Verify structure
ls -la
```

**Expected output:**
README.md

ARCHITECTURE.md

SETUP.md

LICENSE

.gitignore

SQL/

Talend/

docs/
---

## 🗄️ Step 2: Create PostgreSQL Database & Tables

### 2.1 Connect to PostgreSQL

**Windows/macOS/Linux:**
```bash
psql -U postgres
```

You should see:
postgres=#

---

### 2.2 Create Database

```sql
CREATE DATABASE fake_news_db
  WITH ENCODING 'UTF8'
       LC_COLLATE 'C'
       LC_CTYPE 'C';
```

**Verify:**
```sql
\l
```

You should see `fake_news_db` in the list.

---

### 2.3 Create Schemas

```sql
-- Connect to the database
\c fake_news_db

-- Create schemas
CREATE SCHEMA bronze;
CREATE SCHEMA silver;
CREATE SCHEMA gold;

-- Verify
\dn
```

Expected output:
Name   | Owner

---------+----------

bronze  | postgres

gold    | postgres

silver  | postgres
---

### 2.4 Create BRONZE Table

```sql
CREATE TABLE bronze.raw_articles (
    id SERIAL PRIMARY KEY,
    source VARCHAR(255),
    author VARCHAR(255),
    title TEXT,
    description TEXT,
    content TEXT,
    url TEXT,
    "publishedAt" VARCHAR(50),
    category VARCHAR(100),
    ingestion_timestamp VARCHAR(50),
    raw_response TEXT
);

-- Verify
\dt bronze.*
```

---

### 2.5 Create SILVER Table

```sql
CREATE TABLE silver.clean_articles (
    id SERIAL PRIMARY KEY,
    source VARCHAR(255),
    author VARCHAR(255),
    title TEXT,
    description TEXT,
    content TEXT,
    url TEXT UNIQUE,
    "publishedAt" VARCHAR(50),
    category VARCHAR(100),
    word_count INTEGER,
    processed_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Verify
\dt silver.*
```

---

### 2.6 Create GOLD Table

```sql
CREATE TABLE gold.fake_news_results (
    id SERIAL PRIMARY KEY,
    silver_id TEXT,
    title TEXT,
    source VARCHAR(255),
    category VARCHAR(100),
    published_at VARCHAR(50),
    url TEXT,
    fake_score NUMERIC(5,2),
    is_fake BOOLEAN,
    confidence_level VARCHAR(20),
    detection_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Verify
\dt gold.*
```

---

### 2.7 Exit PostgreSQL

```sql
\q
```

---

## 🔧 Step 3: Configure Talend Project

### 3.1 Open Talend Open Studio

1. Launch **Talend Open Studio**
2. Create a new workspace or use existing

---

### 3.2 Import the Project

1. File → Import Project
2. Select the project folder: `fake-news-detection-pipeline`
3. Click **Next** → **Finish**

Wait for import to complete (~1-2 minutes)

---

### 3.3 Create Context Variables

#### **Path in Talend:**
Left Panel → Contexts → Right-click → New Context Group

#### **Context Name:** `ctx_fakenews`

#### **Add these variables:**

| Name | Type | Value |
|---|---|---|
| `api_key` | String | `YOUR_NEWSAPI_KEY` |
| `api_url` | String | `https://newsapi.org/v2/top-headlines` |
| `db_host` | String | `localhost` |
| `db_port` | int | `5432` |
| `db_name` | String | `fake_news_db` |
| `db_user` | String | `postgres` |
| `db_password` | String | `your_postgres_password` |
| `page_size` | int | `100` |
| `categories` | String | `technology` |

**Replace:**
- `YOUR_NEWSAPI_KEY` → Your actual API key from newsapi.org
- `your_postgres_password` → Your PostgreSQL password

---

### 3.4 Create Database Connection

#### **Path in Talend:**
Left Panel → Metadata → Connections de bases de données → New DB Connection

#### **Settings:**

| Field | Value |
|---|---|
| **Name** | `conn_fakenews_pg` |
| **Type de base de données** | `PostgreSQL` |
| **Hôte** | `localhost` |
| **Port** | `5432` |
| **Base de données** | `fake_news_db` |
| **Utilisateur** | `postgres` |
| **Mot de passe** | `your_password` |

#### **Test connection:**
Click **"Tester la connexion"** → Should say **"Connexion réussie"** ✅

---

## ▶️ Step 4: Run the Pipeline

### 4.1 Run Individual Jobs (for testing)
Right-click JOB_BRONZE_NewsIngestion

→ Run as → Talend Run
Wait for completion (look for "Code de sortie = 0")
Repeat for JOB_SILVER_NewsProcessing

and JOB_GOLD_FakeNewsDetection
**Expected:** Each job should complete successfully ✅

---

### 4.2 Run Master Orchestrator (end-to-end)
Right-click JOB_MASTER_Orchestrator

→ Run as → Talend Run
**What you should see:**
[statistics] connected

[statistics] Job JOB_BRONZE_NewsIngestion started

[statistics] 70 rows processed

[statistics] Job JOB_SILVER_NewsProcessing started

[statistics] 60 rows processed

[statistics] Job JOB_GOLD_FakeNewsDetection started

[statistics] 60 rows processed

[statistics] Job JOB_MASTER_Orchestrator terminé à HH:MM

[statistics] Code de sortie = 0
**Code de sortie = 0** = SUCCESS ✅  
**Code de sortie = 1** = ERROR ❌

---

## 🔍 Step 5: Verify Data in PostgreSQL

### Check BRONZE Layer

```bash
psql -U postgres -d fake_news_db -c "SELECT COUNT(*) FROM bronze.raw_articles;"
```

**Expected:** 70+ rows

```bash
psql -U postgres -d fake_news_db -c "SELECT title, source FROM bronze.raw_articles LIMIT 3;"
```

---

### Check SILVER Layer

```bash
psql -U postgres -d fake_news_db -c "SELECT COUNT(*) FROM silver.clean_articles;"
```

**Expected:** ~60 rows (after deduplication)

```bash
psql -U postgres -d fake_news_db -c "SELECT title, word_count FROM silver.clean_articles LIMIT 3;"
```

---

### Check GOLD Layer

```bash
psql -U postgres -d fake_news_db -c "SELECT COUNT(*) FROM gold.fake_news_results;"
```

**Expected:** ~60 rows with scores

```bash
psql -U postgres -d fake_news_db -c "SELECT title, fake_score, is_fake, confidence_level FROM gold.fake_news_results LIMIT 3;"
```

---

## 🚨 Troubleshooting

### Error: "Connection refused"
**Problem:** PostgreSQL not running  
**Solution:**
```bash
# Windows (Command Prompt as Admin)
net start postgresql-x64-14

# macOS
brew services start postgresql@14

# Linux
sudo service postgresql start
```

---

### Error: "Cannot find database"
**Problem:** Database not created  
**Solution:** Re-run Step 2 (Create Database)

---

### Error: "API key invalid"
**Problem:** Wrong NewsAPI key  
**Solution:**
1. Go to https://newsapi.org
2. Verify your API key
3. Update context variable in Talend
4. Retry

---

### Error: "Cannot resolve variable"
**Problem:** Context variables not set  
**Solution:**
1. Check Contexts are created (Step 3.3)
2. Make sure job uses correct context
3. Restart Talend if necessary

---

### Error: "Code de sortie = 1"
**Problem:** Job failed  
**Solution:**
1. Check console output for error message
2. Verify database connection
3. Check context variables
4. Check PostgreSQL is running

---

## ✅ Verification Checklist

- [ ] PostgreSQL installed and running
- [ ] Talend Open Studio installed
- [ ] Database `fake_news_db` created
- [ ] Three schemas created (bronze, silver, gold)
- [ ] Three tables created
- [ ] Project cloned from GitHub
- [ ] Context variables configured
- [ ] Database connection tested
- [ ] JOB_MASTER_Orchestrator runs successfully
- [ ] Data visible in PostgreSQL

---

## 🎉 Success!

If all steps completed:
✅ Pipeline is fully operational  
✅ 70+ articles ingested  
✅ Data cleaned and scored  
✅ Ready for dashboard integration  

---

## 📞 Next Steps

1. **Run the pipeline regularly** to keep data fresh
2. **Integrate with BI tool** (Tableau, Power BI)
3. **Add real fake news detection** (ML model)
4. **Set up automated scheduling** (every 30 min)
5. **Create dashboards** for visualization

---

**Setup Version:** 1.0  
**Last Updated:** June 29, 2026  
**Status:** Ready for Production