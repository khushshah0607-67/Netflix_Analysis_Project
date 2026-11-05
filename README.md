# Netflix Data Analysis Project Using MySQL

This repository contains a comprehensive **MySQL data analysis project focused on exploring and extracting insights from a Netflix titles dataset**. The project covers the entire pipeline, from setting up the database environment and importing raw data using `LOAD DATA INFILE`, to executing advanced SQL queries for content distribution, trend discovery, and text-based category tagging.

## Table of Contents
1. [Database Setup & Data Ingestion](#database-setup--data-ingestion)
2. [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)
3. [Business Insights & Query Categorization](#business-insights--query-categorization)
4. [Advanced Analytics & Schema Engineering](#advanced-analytics--schema-engineering)

---

## Database Setup & Data Ingestion

The dataset is ingested into a local MySQL instance using optimized server variables. Below is the ingestion schema and loading script:

```sql
-- Create the target database
CREATE DATABASE IF NOT EXISTS netflix_db;
USE netflix_db;

-- Load raw dataset safely from the MySQL secure uploads directory
LOAD DATA INFILE "C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/netflix_titles.csv" 
INTO TABLE NET 
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES;
```

---

## Exploratory Data Analysis (EDA)

Initial baseline audits to confirm the total rows, column integrity, and schema architecture:

```sql
-- Total record count audit
SELECT COUNT(*) FROM NET; 

-- Total metadata column count
SELECT COUNT(*) AS TOTAL_COLUMNS 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_SCHEMA = 'netflix_db' AND TABLE_NAME = 'NET'; 

-- Isolate structural metadata column names
SELECT COLUMN_NAME AS COL_NAMES 
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_SCHEMA = 'NETFLIX_DB' AND TABLE_NAME = 'NET'; 

-- System table profiling
DESCRIBE NET; 

-- Preliminary data preview
SELECT * FROM NET LIMIT 20; 

-- Distinct product types available
SELECT DISTINCT TYPE FROM NET; 
```

---

## Business Insights & Query Categorization

### 1. Content Segmentation
Quantifies the split between Movies and TV Shows across the global catalog.
```sql
SELECT TYPE, COUNT(*) AS TOTAL 
FROM NET 
GROUP BY TYPE;
```

### 2. Rating Dominance (Window Functions)
Finds the top (most common) rating category separately for both Movies and TV Shows using `RANK() OVER()`.
```sql
SELECT TYPE, RATING FROM (
    SELECT TYPE, RATING, COUNT(*), 
           RANK() OVER(PARTITION BY TYPE ORDER BY COUNT(*) DESC) AS RANKING 
    FROM NET 
    GROUP BY 1, 2
) AS T1 
WHERE RANKING = 1;
```

### 3. Chronological Filters & String Parsing
Extracts content specific to release parameters or filters out fresh content based on calculated date intervals.
```sql
-- Target year profiling (2020)
SELECT * FROM NET 
WHERE TYPE = 'Movie' AND RELEASE_YEAR = 2020;

-- Content added within historical 5-year boundaries
SELECT * FROM NET 
WHERE current_date() - STR_TO_DATE(DATE_ADDED, '%M %d, %Y') > 5;
```

### 4. Geographic & Creator Deep Dives
Identifies regional production density and tracks specific high-output creators or genres.
```sql
-- Top production countries
SELECT COUNTRY, COUNT(*) 
FROM NET 
GROUP BY COUNTRY 
ORDER BY 2 DESC 
LIMIT 6;

-- Filmography search by specific creator
SELECT * FROM NET 
WHERE DIRECTOR LIKE '%Rajiv Chilaka%';

-- Genre-specific querying
SELECT * FROM NET 
WHERE LISTED_IN LIKE '%Documentaries%';
```

---

## Advanced Analytics & Schema Engineering

### Data Parsing & Long-tail Analysis
Converts runtime metric strings into raw integer components for aggregations:
```sql
-- Isolate longest feature film metrics
SELECT type, MAX(CAST(SUBSTR(DURATION, 1, 3) AS UNSIGNED)) 
FROM NET 
WHERE TYPE='Movie';

-- TV Shows scaling beyond traditional seasonal lengths (>5 Seasons)
SELECT * FROM NET 
WHERE TYPE = 'TV Show' AND CAST(SUBSTR(DURATION, 1, 2) AS UNSIGNED) > 5;
```

### Weighted Historical Distributions
Calculates chronological density mapping for specific target markets (e.g., India) to assess annual velocity relative to total global footprints:
```sql
SELECT 
    EXTRACT(YEAR FROM STR_TO_DATE(DATE_ADDED, "%M %d, %Y")) AS YEAR, 
    COUNT(*) AS COUNT, 
    COUNT(*)/(SELECT COUNT(*) FROM NET WHERE COUNTRY = 'India') * 100 AS AVG_CONTENT_PER_YEAR 
FROM NET 
WHERE COUNTRY = 'India' 
GROUP BY 1 
ORDER BY 3 DESC;
```

### Data Cleansing & Conditional Sentiment Flagging
Utilizes Common Table Expressions (CTEs) alongside conditional strings to run risk metrics or content classification profiles on text descriptions:
```sql
-- Audit data completeness (Missing directorial attribution)
SELECT * FROM NET WHERE director IS NULL OR director = '';

-- Multi-variable time/cast constraints
SELECT * FROM NET 
WHERE cast LIKE '%Salman Khan%' 
  AND EXTRACT(YEAR FROM CURRENT_DATE) - release_year < 10;

-- Contextual text analysis and metric mapping via CTE
WITH CONTENT_CATEGORY AS (
    SELECT DESCRIPTION, 
           CASE WHEN DESCRIPTION LIKE '%Kill%' OR DESCRIPTION LIKE '%Violence%' THEN 'Bad' 
                ELSE 'Good' 
           END category 
    FROM NET
) 
SELECT category, COUNT(*) 
FROM CONTENT_CATEGORY 
GROUP BY 1;
```
