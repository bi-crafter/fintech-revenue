# Technical Architecture Documentation

## Data Model Overview

### Star Schema Design

```
                    DimCustomer
                         │
                         │ (1:M)
                         │
FactRevenue ◄────────────┴─────────────► DimProduct
    │
    ├──────────────► DimDate
    │
    └──────────────► DimChannel
```

### Fact Table: FactRevenue
**Grain**: One row per transaction  
**Row Count**: 80,000+ transactions  
**Storage**: ~28 MB (compressed columnar)

#### Measures (Numeric Columns)
```
Amount              DECIMAL(18,4)     Revenue amount in base currency
Quantity            INTEGER           Units sold
MarketingSpend      DECIMAL(18,2)     Acquisition cost attribution
Discount            DECIMAL(18,4)     Promotional discount applied
```

#### Keys (Foreign Keys)
```
CustomerId          Foreign → DimCustomer
ProductId           Foreign → DimProduct
DateKey             Foreign → DimDate (YYYYMMDD integer)
ChannelId           Foreign → DimChannel
```

#### Attributes (Dimension-like Data)
```
TransactionStatus   "Completed", "Refund", "Pending"
PaymentMethod       "Card", "Bank Transfer", "Digital Wallet"
```

---

### Dimension Table: DimCustomer
**Row Count**: ~12,000 unique customers  
**Type**: Slowly Changing Dimension (SCD Type 1)

```
CustomerId              (PK)
CustomerName            VARCHAR
Email                   VARCHAR
AcquisitionDate         DATE (YYYYMMDD)
AcquisitionSource       "Organic", "Paid Search", "Social", "Referral", "Email"
Segment                 "Enterprise", "SMB", "Mid-Market"
Industry                VARCHAR
IsActive                BIT (1 = active, 0 = churned)
MonthsSinceCohort       CALCULATED COLUMN (DATEDIFF logic)
```

**Calculated Column: MonthsSinceCohort**
```dax
DATEDIFF(
    MIN(RELATEDTABLE(FactRevenue)[TransactionDate]),
    TODAY(),
    MONTH
)
```
Value Range: 0-24 months (supports 24-month retention analysis)

---

### Dimension Table: DimProduct
**Row Count**: 8-12 products  
**Type**: SCD Type 1 (Slowly Changing)

```
ProductId               (PK)
ProductName             VARCHAR     "Lending Platform", "FX Marketplace", etc.
ProductCategory         VARCHAR     "Fintech Core", "Ancillary"
ProductLine             VARCHAR     "Core Revenue", "Experimental"
IsActive                BIT
LaunchDate              DATE
```

---

### Dimension Table: DimDate
**Row Count**: ~3,700 dates (10-year span)  
**Type**: Conformed Dimension

```
DateKey                 (PK)        YYYYMMDD format (e.g., 20240515)
Date                                Calendar date
Year                    NUMBER      2020-2025
Quarter                 VARCHAR     "Q1", "Q2", etc.
Month                   VARCHAR     "January", "February"
MonthNumber             NUMBER      1-12
DayOfWeek               VARCHAR     "Monday", "Tuesday"
WeekOfYear              NUMBER      1-52
FiscalYear              NUMBER      Aligned to business fiscal calendar
FiscalQuarter           VARCHAR     "FY24 Q1", etc.
IsWeekend               BIT         1 if Sat/Sun
```

**Rationale**: Pre-calculated date dimension allows:
- Fast integer joins (DateKey vs DATE datatype)
- Fiscal calendar alignment (important for Q4-heavy business)
- Granular filtering without DAX calculation overhead

---

### Dimension Table: DimChannel
**Row Count**: 5-6 channels  
**Type**: SCD Type 1

```
ChannelId               (PK)
ChannelName             VARCHAR     "Direct Sales", "Marketplace", "API", etc.
ChannelType             VARCHAR     "B2B", "B2C", "B2B2C"
GeographicFocus         VARCHAR     "EMEA", "APAC", "Americas"
IsActive                BIT
```

---

## Relationship Configuration

### Join Keys & Cardinality

| Relationship | Cardinality | Direction | Active | Business Rule |
|-------------|-------------|-----------|--------|---------------|
| FactRevenue → DimCustomer | (M:1) | Both | ✓ Yes | Each transaction belongs to one customer |
| FactRevenue → DimProduct | (M:1) | Both | ✓ Yes | Each transaction is for one product |
| FactRevenue → DimDate | (M:1) | Both | ✓ Yes | DateKey INTEGER join for performance |
| FactRevenue → DimChannel | (M:1) | Both | ✓ Yes | Each transaction used one channel |

### Cross-Filter Direction
All relationships use **BOTH** directions to allow:
- Filtering fact by dimension attributes
- Propagating dimension filters across related tables

---

## DAX Formulas & Calculations

### Category: Unit Economics

#### Average Customer Acquisition Cost (CAC)
```dax
[Average CAC] = 
    DIVIDE(
        SUM(FactRevenue[MarketingSpend]),
        DISTINCTCOUNT(FactRevenue[CustomerId]),
        0
    )
```
**Usage**: Identifies most efficient acquisition channels  
**Example Output**: $38.50 per customer acquired

#### CAC Payback Period
```dax
[CAC Payback Period (Months)] = 
    DIVIDE(
        [Average CAC],
        [Average Revenue Per Customer] / 12,
        0
    )
```
**Usage**: Determines how many months to break even on acquisition spend  
**Insight**: If > 6 months, channel is unprofitable

#### Customer Acquisition Efficiency
```dax
[CAC Efficiency Ratio] = 
    DIVIDE(
        [Total Revenue],
        [Total Marketing Spend],
        0
    )
```
**Interpretation**: 
- Ratio = 1.0: Each $1 of spend → $1 of revenue (break-even)
- Ratio > 1.5: Efficient channel (scale aggressively)
- Ratio < 0.5: Inefficient channel (deprioritize)

---

### Category: Time Intelligence

#### Year-over-Year Revenue Growth
```dax
[YoY Revenue Growth %] = 
    VAR CurrentYear = [Total Revenue]
    VAR PriorYear = CALCULATE(
        [Total Revenue],
        SAMEPERIODLASTYEAR(DimDate[Date])
    )
    RETURN
        DIVIDE(
            CurrentYear - PriorYear,
            PriorYear,
            0
        ) * 100
```
**Context Awareness**: Automatically adjusts based on selected date range  
**Example**: If user selects "Q2 2024", formula compares Q2 2024 vs Q2 2023

#### Rolling 12-Month Revenue
```dax
[Rolling 12M Revenue] = 
    CALCULATE(
        [Total Revenue],
        DATESBETWEEN(
            DimDate[Date],
            TODAY() - 365,
            TODAY()
        )
    )
```
**Purpose**: Smooths seasonality common in fintech (Q4 spikes)  
**Business Insight**: Reveals true growth trend vs. seasonal noise

#### Prior Period Revenue
```dax
[Prior Period Revenue] = 
    CALCULATE(
        [Total Revenue],
        DATEADD(DimDate[Date], -1, MONTH)
    )
```
**Usage**: Month-over-month comparisons for variance analysis

---

### Category: Growth Attribution & Variance

#### Revenue Variance (Bridge Chart Driver)
```dax
[Revenue Variance] = 
    [Total Revenue] - [Prior Period Revenue]
```
**Output**: Actual dollar difference ($850K from Lending, etc.)

#### Variance Percentage
```dax
[Variance %] = 
    DIVIDE(
        [Revenue Variance],
        [Prior Period Revenue],
        0
    ) * 100
```
**Output**: Percentage contribution to total growth

#### Product-Level Growth Attribution
```dax
[Product Growth Contribution %] = 
    DIVIDE(
        [Current Product Revenue] - [Prior Product Revenue],
        SUM(
            CALCULATE(
                [Current Product Revenue] - [Prior Product Revenue],
                ALLEXCEPT(FactRevenue, FactRevenue[ProductId])
            )
        ),
        0
    ) * 100
```
**Insight**: "Which product drove 65% of FY24 growth?" → SMB Lending

---

### Category: Retention & Cohort Analysis

#### Cohort Retention Percentage
```dax
[Cohort Retention %] = 
    VAR CohortMonth = 
        SELECTEDVALUE(DimCustomer[CohortMonth])
    VAR CurrentMonthUsers = 
        DISTINCTCOUNT(FactRevenue[CustomerId])
    VAR CohortSizeAtStart = 
        CALCULATE(
            DISTINCTCOUNT(FactRevenue[CustomerId]),
            ALLEXCEPT(
                FactRevenue,
                FactRevenue[CohortMonth]
            )
        )
    RETURN
        DIVIDE(
            CurrentMonthUsers,
            CohortSizeAtStart,
            0
        ) * 100
```

**Critical Design**: `ALLEXCEPT` ensures denominator = initial cohort size (not current)
- Without ALLEXCEPT: Denominator shrinks each month (misleading decline)
- With ALLEXCEPT: True retention ratio showing actual churn

**Example Output**:
```
Cohort: Jan 2023
  Month 0: 100% (1,200 customers)
  Month 1: 92%  (1,104 customers)
  Month 3: 77%  (924 customers) ← 15% churn spike detected
  Month 6: 65%  (780 customers)
  Month 12: 58% (696 customers)
```

#### Active Customer Count
```dax
[Active Customers] = 
    CALCULATE(
        DISTINCTCOUNT(FactRevenue[CustomerId]),
        DimCustomer[IsActive] = 1
    )
```

---

## Aggregation & Performance Optimization

### Materialized Aggregations
Power BI applies automatic aggregations at:
- **Daily grain**: Pre-calculated daily revenue by Product/Channel
- **Monthly grain**: Pre-calculated monthly cohort retention

### Query Optimization Techniques
1. **DateKey Integer Join**: ~2x faster than DATE datatype joins
2. **Column Encoding**: Categorical columns (Channel, Segment) use dictionary encoding (~70% compression)
3. **Aggregation Folding**: Power Query transformations pushed to data engine where possible
4. **Calculated Columns Sparing**: Only 2 calculated columns (DayOfWeek, MonthsSinceCohort)

### Storage Footprint
```
Fact Table (80k rows):       ~18 MB
Dimensions (20k rows total):  ~8 MB
Calculations & Metadata:      ~2 MB
─────────────────────────────────
Total Semantic Model:        ~28 MB
```

Compressed at ~8:1 ratio vs. original CSV (~240 MB)

---

## Data Refresh & Maintenance

### Refresh Schedule
- **Fact Table**: Daily at 12:00 AM UTC (after close of trading)
- **Dimensions**: Daily at 12:15 AM UTC
- **Incremental Refresh**: Last 90 days of data (initial load 2 years historical)

### Data Quality Checks
```
Power Query Rules:
1. NULL → Replace with "Unknown" (Segment, Channel)
2. Currency Conversion → All amounts in USD base currency
3. Duplicate Detection → Transaction ID uniqueness validated
4. Date Validation → TransactionDate must be ≤ TODAY()
5. Referential Integrity → All FK values exist in dimension
```

### Error Handling
- **Soft Errors**: Logged to Power Query diagnostic log (trace in Power BI Service)
- **Hard Errors**: Refresh fails; PBI Service alerts admin
- **Quarantine Process**: Failed rows moved to `FailedTransactions` auxiliary table for investigation

---

## Visualization Mapping

### Report Page 1: Revenue Overview
**Visuals**:
- KPI Card: [Total Revenue]
- KPI Card: [YoY Revenue Growth %]
- Area Chart: Revenue trend (by Month)
- Pie Chart: Revenue by Product

**Filters**: Date Range, Product Filter, Segment Slicer

### Report Page 2: Revenue Bridge (Growth Attribution)
**Main Visual**: Stacked Column Chart (Waterfall effect)
- X-Axis: [Product]
- Y-Axis: [Product Growth Contribution %]
- Color: Positive (teal) vs. Negative (red)

**Insight**: Drills into "which product drove 65% of growth"

### Report Page 3: Unit Economics (4-Quadrant)
**Main Visual**: Scatter Plot
- X-Axis: [Average CAC]
- Y-Axis: [YoY Revenue Growth %]
- Size: [Total Revenue]
- Color: [Channel Name]

**Insight**: "Social Ads have high CAC; Email has low CAC—reallocate 10% budget"

### Report Page 4: Cohort Retention Heatmap
**Main Visual**: Matrix (Rows × Columns)
- Rows: [CohortMonth] (Jan 2023 - Dec 2024)
- Columns: [MonthsSinceCohort] (0-24)
- Values: [Cohort Retention %]
- Color Scale: 0% red → 100% dark green

**Insight**: "Month 3 cohorts show 15% churn spike—UX issue?"

---

## Next Steps for Enhancement

### Suggested Improvements (Post-MVP)
1. **Predictive Analytics**: Add Python integration to forecast churn risk profiles
2. **Incrementalization**: Implement incremental refresh for FactRevenue (last 30 days only)
3. **RLS (Row-Level Security)**: Restrict customer visibility by Account Manager
4. **Decomposition Tree**: Drill into variance drivers automatically
5. **Real-Time Refresh**: Shift to Premium capacity for sub-hourly refreshes

---

## Troubleshooting Guide

### Issue: Cohort Retention % shows declining trend across all months
**Cause**: Missing `ALLEXCEPT` in denominator; denominator shrinking  
**Fix**: Ensure denominator locks to initial cohort size

### Issue: YoY Growth showing 0% or error
**Cause**: Prior year data missing (e.g., selected date range is < 1 year)  
**Fix**: Use COALESCE or provide MIN date >= 1 year prior

### Issue: CAC calculation showing NULL
**Cause**: DIVIDE by zero (no marketing spend in selected filter context)  
**Fix**: Ensure denominator checks are in place; use DIVIDE(..., 0) to return 0 vs. error

### Issue: Report takes >5 seconds to load
**Cause**: Multiple DISTINCTCOUNT functions in complex filters  
**Fix**: Enable aggregation folding; limit matrix dimensions (max 2k cells)

---

**Document Version**: 1.0 | Last Updated: May 2026
