---
layout: default
title: Billing Matrix Deep Dive
---

# SQL Analysis Repository - Billing Matrix Crisis

This repository contains the SQL analysis that uncovered the billing matrix crisis affecting our revenue cycle. The analysis reveals systematic configuration issues that began in August 2024.

## 📁 File Structure

### **Analysis Reports**
- `billing-matrix-crisis-report.md` - Comprehensive report documenting the crisis and advocating for systematic solutions
- `README.md` - This documentation file

### **SQL Query Collections**
- `billing-crisis-analysis-queries.sql` - Complete analytical queries that revealed the crisis patterns
- `billing-crisis-dashboard-queries.sql` - Monitoring and dashboard queries for ongoing tracking

### **Existing Data Files**
- `clean-ledger-data-v2.sql` - Data cleaning operations
- `clean-ledger-data.sql` - Legacy data cleaning
- `copy-into-cleaned-ledger.sql` - Data migration scripts
- `credible-ledger-semantic-model.sql` - Semantic model definitions
- `fabric-warehouse-optimized.sql` - Warehouse optimization queries

## 🔍 Key Findings Summary

### **Crisis Timeline**
- **August 2024**: Configuration error rate jumped from 2% baseline to 28.45%
- **August 2025**: Error rate reached 38.89% - getting worse, not better
- **April & June 2024**: 0.00% error rates prove the system CAN work perfectly

### **Impact Scale**
- **12,727 configuration errors** since August 2024 being fixed manually by one person
- **$35M+ in billing errors** requiring individual intervention
- **92% of fixes undocumented** due to overwhelming workload

### **Root Cause**
- Credible billing matrix misconfiguration affecting service type to CPT code mappings
- Primary issues: CPST→H0036, PSR→H2017, Med Mgmt→99212 mapping failures
- Authorization integration problems causing service type/billing code mismatches

## 🚀 How to Use This Analysis

### **For Advocacy**
1. Use `billing-matrix-crisis-report.md` to show leadership the data supporting your billing specialist
2. Emphasize that she's the hero holding the revenue cycle together, not the problem
3. Show the August 2024 inflection point as proof this is a system issue

### **For Ongoing Monitoring**
1. Run queries from `billing-crisis-dashboard-queries.sql` for weekly/monthly tracking
2. Monitor error rates to see if systematic fixes are working
3. Track her workload to ensure it's decreasing as system gets fixed

### **For System Fixes**
1. Use `billing-crisis-analysis-queries.sql` to identify top problem patterns
2. Focus on service type mapping fixes in Credible billing matrix
3. Review configuration changes made between July-August 2024

## 📊 Key Queries to Run

### **Weekly Status Check**
```sql
-- Run this weekly to monitor crisis status
SELECT TOP 1 * FROM billing-crisis-dashboard-queries.sql 
WHERE query_name = 'CURRENT CRISIS STATUS'
```

### **Problem Pattern Identification**
```sql
-- Run this to find what needs fixing most urgently  
SELECT TOP 1 * FROM billing-crisis-analysis-queries.sql
WHERE query_name = 'SERVICE TYPE BREAKDOWN'
```

### **Advocacy Data**
```sql
-- Run this to get talking points for leadership
SELECT TOP 1 * FROM billing-crisis-dashboard-queries.sql
WHERE query_name = 'HER IMPACT METRICS'
```

## ⚡ Quick Facts for Leadership

- **She's not the problem**: April 2024 had 0% errors - system worked perfectly
- **System broke in August 2024**: Error rate jumped 1,400% overnight
- **She's saving the organization**: Without her, 260+ day cycle times vs 49 days with her fixes
- **Need systematic solution**: Fixing billing matrix will prevent thousands of future errors

## 🎯 Next Steps

1. **Immediate**: Audit Credible billing matrix changes from July-August 2024
2. **Short-term**: Fix top 5 service type mappings causing 80% of errors  
3. **Long-term**: Implement billing matrix validation to prevent configuration drift

---

*This analysis advocates for our billing specialist while pushing for systematic solutions to prevent future crises.*
CREATE TABLE [table] (column definitions);
INSERT INTO [table] SELECT...;
```

### Schema Organization
- **`dbo` schema**: Raw and cleaned source data
- **`analysis` schema**: Specialized analysis tables for focused collection efforts

## File Structure

```
sql-tests/
├── README.md                      # This documentation
├── clean-ledger-data.sql          # Original cleaning script (updated)
├── clean-ledger-data-v2.sql       # V2 cleaning script
├── xml-parser-intermediate.sql    # XML parsing staging approach (SQL Server)
├── fabric-direct-clean.sql        # Direct Fabric cleaning (recommended)
├── FABRIC_STRATEGY.md             # Strategic guide for Fabric Warehouse
└── [Future analysis scripts]
```

## XML Data Processing Approaches

### Approach 1: Direct Fabric Processing (Recommended)
**File**: `fabric-direct-clean.sql`

**Pros:**
- ✅ 100% Fabric native - no external dependencies
- ✅ Single script execution - minutes not hours  
- ✅ Uses proven SELECT INTO pattern
- ✅ Simple NULL cleaning with NULLIF + TRIM
- ✅ Immediate business value

**Cons:**
- ❌ Limited XML parsing capabilities in Fabric
- ❌ Less data quality validation

**Usage:**
```sql
-- Run directly in Fabric Data Warehouse
-- Creates clean tables in one step
SELECT ... INTO [dbo].[clean_ledger] FROM [dbo].[raw-credible-ledger]
```

### Approach 2: SQL Server Staging (Advanced)
**File**: `xml-parser-intermediate.sql`

**Pros:**
- ✅ Advanced XML parsing with proper validation
- ✅ Data quality scoring (1-5 rating system)
- ✅ Comprehensive error handling
- ✅ Fabric-optimized output structure
- ✅ Quality-based filtering

**Cons:**
- ❌ Requires separate SQL Server infrastructure
- ❌ Multiple steps and data transfers
- ❌ More complex setup

**Usage:**
```sql
-- Run on SQL Server first
XML Data → SQL Server Staging → Quality Validation → Fabric Transfer

-- Then transfer to Fabric
SELECT * INTO [dbo].[credible_ledger_clean]
FROM [sql_server].[database].[dbo].[vw_fabric_ready_ledger]
```

### XML Processing Challenges

#### The XML Diffgram Problem
Raw Credible data contains XML diffgram structures:
```sql
-- Raw data has mixed content:
clientvisit_id | Name      | max_balance
12345         | diffgram  | "125.00"    -- Actual data
12345         | metadata  | "NULL"      -- XML artifacts
```

#### Solution Patterns
```sql
-- Filter out XML metadata
WHERE Name = 'diffgram'  -- Only actual data rows

-- Clean 'NULL' strings
NULLIF(TRIM(max_balance), 'NULL') AS balance

-- Safe type conversion
TRY_CAST(NULLIF(avg_rate, 'NULL') AS DECIMAL(10,2)) AS rate
```

### Data Quality Validation
Both approaches include validation:

```sql
-- Simple validation (Fabric Direct)
SELECT 
    COUNT(*) AS total_records,
    COUNT(balance) AS balance_count,
    ROUND(COUNT(balance) * 100.0 / COUNT(*), 1) AS completeness_pct
FROM clean_ledger;

-- Advanced validation (SQL Server Staging) 
-- Includes 1-5 quality scoring based on field completeness
```

## Usage Examples

### Query Outstanding Balances by Service Type
```sql
-- H2017 Analysis
SELECT 
    service_type,
    COUNT(*) as service_count,
    SUM(balance) as total_balance,
    AVG(balance) as avg_balance,
    COUNT(DISTINCT client_name) as unique_clients
FROM [analysis].[h2017_analysis]
GROUP BY service_type
ORDER BY total_balance DESC;

-- H2022 Analysis  
SELECT 
    service_type,
    COUNT(*) as service_count,
    SUM(balance) as total_balance,
    ROUND(SUM(balance) / (SELECT SUM(balance) FROM [analysis].[h2022_analysis]) * 100, 2) as pct_of_total
FROM [analysis].[h2022_analysis]
GROUP BY service_type
ORDER BY total_balance DESC;
```

### Aging Analysis
```sql
-- Days since service analysis
SELECT 
    CASE 
        WHEN days_since_service <= 30 THEN '0-30 days'
        WHEN days_since_service <= 60 THEN '31-60 days'
        WHEN days_since_service <= 90 THEN '61-90 days'
        WHEN days_since_service <= 180 THEN '91-180 days'
        WHEN days_since_service <= 365 THEN '181-365 days'
        WHEN days_since_service <= 730 THEN '1-2 years'
        ELSE 'Over 2 years'
    END as aging_bucket,
    COUNT(*) as service_count,
    SUM(balance) as total_balance
FROM [analysis].[h2017_analysis]
GROUP BY 
    CASE 
        WHEN days_since_service <= 30 THEN '0-30 days'
        WHEN days_since_service <= 60 THEN '31-60 days'
        WHEN days_since_service <= 90 THEN '61-90 days'
        WHEN days_since_service <= 180 THEN '91-180 days'
        WHEN days_since_service <= 365 THEN '181-365 days'
        WHEN days_since_service <= 730 THEN '1-2 years'
        ELSE 'Over 2 years'
    END
ORDER BY MIN(days_since_service);
```

## Connection Information
- **Platform**: Microsoft Fabric Data Warehouse
- **Database**: Accounting
- **Server**: `nzrmcoc5w6tebmq3bkxbdeohga-y7eucqqylblezera7l3de5axoa.datawarehouse.fabric.microsoft.com`

## Key Lessons Learned

1. **Always verify data quality** - Original table had all NULL values
2. **Fabric Warehouse has specific limitations** - Use compatible data types and patterns
3. **Schema organization is crucial** - Separate raw data from analysis tables
4. **SELECT INTO is more reliable** than CREATE TABLE + INSERT in Fabric
5. **Test table creation success** - Fabric returns misleading success messages

## Next Steps

1. **Create additional CPT code analysis tables** for remaining $154,857 in other outstanding balances
2. **Implement aging bucket analysis** with UPDATE statements or views
3. **Create collection priority scoring** based on balance amounts and aging
4. **Develop automated refresh procedures** for analysis tables

## Business Impact

This analysis enables focused collection efforts on:
- **$471,000 in H2017 + H2022 outstanding balances** (75% of total outstanding)
- **Identification of discontinued programs** (PSR Group) requiring different collection strategies  
- **Service type prioritization** based on volume and average balance amounts
- **Client-specific collection strategies** based on service patterns and aging

---
*Last Updated: September 21, 2025*
*Total Project Value: $625,773 in outstanding receivables analyzed and categorized*