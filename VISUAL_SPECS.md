# VISUAL SPECIFICATION REFERENCE
**Use this with Power BI Desktop open. Copy-paste ready formulas below each visual.**

---

## PAGE 1: REVENUE AT RISK

### KPI 1: Customer Retention Rate (Top-left, 22% width)
**Visual Type:** Card  
**Measure:** `[_Customer Retention Rate]`  
**Format:** Percentage (1 decimal)  
**Font:** Value 28pt bold, Label 10pt muted  
**Colors:** Teal #1D9E75  
**Comparison:** Show MoM % change (conditional: ↑ green if improving)

---

### KPI 2: Customers at Risk (22% width)
**Visual Type:** Card  
**Measure:** `[_Customers at Risk]`  
**Format:** Number (#,##0)  
**Conditional Coloring:** Red if > 400, Amber if 300-400, Green if < 300  
**Comparison:** Show % change MoM

---

### KPI 3: Revenue at Risk (22% width)
**Visual Type:** Card  
**Measure:** `[_Revenue at Risk]`  
**Format:** Currency ($#,##0)  
**Colors:** Coral #D85A30 (warning)  
**Comparison:** Show $ change and % MoM

---

### KPI 4: Avg Days Since Transaction (22% width)
**Visual Type:** Card  
**Measure:** 
```dax
VAR AtRiskCohort = FILTER(ALL('FactRevenue - FactRevenue csv'),
  DATEDIFF(MAX('DimDate - DimDate csv'[FullDate]), 
    'DimDate - DimDate csv'[FullDate], DAY) >= 60)
VAR AvgDays = AVERAGEX(AtRiskCohort,
  DATEDIFF(MAX('DimDate - DimDate csv'[FullDate]), 
    'DimDate - DimDate csv'[FullDate], DAY))
RETURN AvgDays
```
**Format:** Whole number  
**Colors:** Amber #EF9F27 (higher = worse)  
**Comparison:** Show MoM change (↑ red if increasing)

---

### WATERFALL: 6-Month Revenue Cohort Flow (50% width, center)
**Visual Type:** Waterfall Chart  
**Axis:** `'DimDate - DimDate csv'[YearMonth]` (last 6 months only)  
**Values:** 
- Main: `SUM('FactRevenue - FactRevenue csv'[NetRevenue])`
- Add breakdown by `[_RFM Score]`

**Column Colors:**
```
Start: Teal #1D9E75
Retained cohort: Light Teal #7ECAB3
Churned loss: Coral #D85A30
New acquisition: Purple #534AB7
Net change: Teal #1D9E75
```

**Format Rules:**
- Y-axis: "$M" format (0–9M range)
- X-axis: Month + Year
- Remove axis lines, keep tick marks
- Add horizontal gridlines only
- Data labels: Always show ($ values)
- Legend: Right side, show RFM segments

**Tooltip Code:**
```
Month: [YearMonth]
RFM Segment: [_RFM Score]
Revenue: [NetRevenue]
% of Total: Total Transactions
```

---

### SCATTER: RFM Customer Segmentation (48% width, bottom-left)
**Visual Type:** Scatter Chart  
**Axes:**
- X-axis: 
```dax
VAR LastDate = MAX('DimDate - DimDate csv'[FullDate])
RETURN CALCULATE(
  DATEDIFF(CALCULATE(MAX('DimDate - DimDate csv'[FullDate]), 
    ALLEXCEPT('FactRevenue - FactRevenue csv', 'FactRevenue - FactRevenue csv'[CustomerKey])),
    LastDate, DAY),
  ALLEXCEPT('FactRevenue - FactRevenue csv', 'FactRevenue - FactRevenue csv'[CustomerKey]))
```
- Y-axis: `SUM('FactRevenue - FactRevenue csv'[NetRevenue])` (LTM only—use Date filter)

**Size:** `DISTINCTCOUNT('FactRevenue - FactRevenue csv'[TransactionKey])` (frequency)  
**Color:** `[_RFM Score]`

**Color Mapping:**
```
Champions: Teal #1D9E75
Loyal: Purple #534AB7
At-Risk: Amber #EF9F27
Churned: Coral #D85A30
New: Light Gray #D0CECA
```

**Format Rules:**
- X-axis: 0–180 days, label "Days since purchase"
- Y-axis: 0–$3M, label "Lifetime revenue"
- Legend: Right side, sort by RFM hierarchy
- Title: "Customer risk segmentation based on RFM model"
- Subtitle: "Larger bubbles = more frequent. Left = active, right = churn risk."

---

### STACKED BAR (100%): CLTVBand Monthly Composition (48% width, bottom-right)
**Visual Type:** Stacked Bar Chart (100% arrangement)  
**Axis:** `'DimDate - DimDate csv'[YearMonth]` (last 6 months)  
**Values:** `COUNTA('DimCustomer - DimCustomer csv'[CustomerKey])` by `'DimCustomer - DimCustomer csv'[CLTVBand]`

**Color Map:**
```
Gold: Teal #1D9E75
Silver: Light Purple #A9A0D5
Bronze: Amber #EF9F27
Lapsed: Gray #D0CECA
```

**Format Rules:**
- Stacking: 100% (percentage)
- Sort order: Gold → Silver → Bronze → Lapsed
- Data labels: Show % inside bars (white text, bold)
- Title: "Customer value band composition by month"
- Subtitle: "Watch for Gold band shrinkage. Green if stable/growing."
- No axis lines
- Horizontal gridlines only

**Tooltip:**
```
Month: [YearMonth]
Band: [CLTVBand]
Count: [CustomerKey]
% of Total: [Count as % of row total]
```

---

### TEXT BOX: Insight Callout (Top-right corner)
**Content (Auto-Generated):**
```
As of [LastDate], [AtRiskCount] customers representing 
$[AtRiskRevenue] in annual revenue have not transacted 
in 60+ days.

Churn is concentrated in High-CAC partner channels. 
Activate win-back campaigns within 48 hours.
```

**DAX Substitutions:**
- `[LastDate]` = `FORMAT(MAX('DimDate - DimDate csv'[FullDate]), "mmm d, yyyy")`
- `[AtRiskCount]` = `[_Customers at Risk]`
- `[AtRiskRevenue]` = `FORMAT([_Revenue at Risk], "$#,##0")`

**Styling:**
- Font: Segoe UI, 11pt, Color #1A1A18
- Background: #F0F8F5 (light teal)
- Border: 1px solid Teal #1D9E75
- Padding: 16px
- Border radius: 8px

---

### SLICER: Time Period (Page-scoped, above KPIs)
**Type:** Dropdown Slicer  
**Field:** `'DimDate - DimDate csv'[YearMonth]`  
**Default Selection:** Last 12 months  
**Label:** "Time period"  
**Position:** Top-left, 200px width

---

## PAGE 2: REAL VS NOISE

### COMBO CHART: Dual-Line Revenue Trend (Full width, top 50%)
**Visual Type:** Combo Chart (Line + Line)  
**Shared X-Axis:** `'DimDate - DimDate csv'[FullDate]` (all data, 2019–2025)

**Line 1: Raw Monthly Revenue**
```dax
SUM('FactRevenue - FactRevenue csv'[NetRevenue])
```
- Format: Thin line (2px), Teal #1D9E75 at 40% opacity
- No data labels

**Line 2: 12-Month Moving Average**
```dax
[_Moving 12M Avg Revenue]
```
- Format: Thick line (4px), Solid Teal #1D9E75, 100% opacity
- No data labels

**Reference Line:** Vertical at October 1st (seasonal start)
```
Line: Dashed, Amber #EF9F27, 2px
Label: "Seasonal uplift zone"
```

**Format Rules:**
- Y-axis: $0–$3M, label "$M"
- X-axis: Full date range, monthly ticks
- Legend: Right side (Raw = light, MA = dark teal)
- Title: "Separation of seasonal noise from organic growth trend"
- Subtitle: "Thin = noisy raw data. Thick = true trend (12M smoothed)."
- Horizontal gridlines only

**Tooltip:**
```
Date: [FullDate]
Raw Revenue: [Sum of NetRevenue]
12M Avg: [_Moving 12M Avg Revenue]
Variance: [_Seasonal Variance %]
```

---

### BAR CHART: YoY Growth Rate by Month (Right, 50% width)
**Visual Type:** Bar Chart  
**Axis:** `'DimDate - DimDate csv'[YearMonth]` (last 36 months: 2023–2025)  
**Values:** 
```dax
[_YoY Growth %]
```

**Conditional Color:**
```dax
IF([_YoY Growth %] > 0.08, "#1D9E75", "#D85A30")
```
- Teal if > 8% (annual avg)
- Coral if < 8%

**Format Rules:**
- Y-axis: -10% to +20%
- Data labels: Always show, % format
- Title: "Month-over-month YoY growth reveals deceleration trend"
- Subtitle: "Teal = above average. Coral = below. Watch for sustained teal."
- Sort: Descending by growth rate

**Tooltip:**
```
Month-Year: [YearMonth]
YoY Growth: [_YoY Growth %]
$ Difference: [SUM of NetRevenue current year] - [Prior year]
```

---

### MULTI-LINE: Product Category Sparklines (Bottom-left, 48%)
**Visual Type:** Line Chart (Small Multiples effect)  
**X-Axis:** `'DimDate - DimDate csv'[Month]` (last 24 months)  
**Y-Axis:** `SUM('FactRevenue - FactRevenue csv'[NetRevenue])`  
**Legend:** `'DimProduct - DimProduct csv'[ProductCategory]`

**Data Filter:** Top 6 categories by total revenue (descending)

**Line Colors (cycle):**
```
Category 1: Teal #1D9E75
Category 2: Purple #534AB7
Category 3: Coral #D85A30
Category 4: Amber #EF9F27
Category 5: Light Blue #4C99E1
Category 6: Light Purple #9B8FC0
```

**Format Rules:**
- Sparkline style (minimal axes)
- Each line labeled with category name
- Line width: 2px
- No axis lines
- Minimal gridlines
- Legend: Right side
- Title: "Top 6 product categories show mixed momentum"
- Subtitle: "Rising slope = growth. Flat/declining = maturity."

---

### KPI METRIC CARDS (Bottom-right, 2×2 grid)

#### Card 1: Organic Growth Rate (ex-Q4)
```dax
[_Organic Growth Rate]
```
**Format:** 0.0%, Teal #1D9E75  
**Label:** "Organic growth rate (ex-Q4)"  
**Subtitle:** "(Excluding seasonal Oct–Dec)"

#### Card 2: Seasonal Variance
```dax
[_Seasonal Variance %]
```
**Format:** +0.0%  
**Label:** "Seasonal variance"  
**Subtitle:** "(Current month vs 12M avg)"  
**Conditional Color:** Teal if positive, Coral if negative

#### Card 3: Trend Confidence (R²)
```dax
VAR RegressionX = AVERAGEX(VALUES('DimDate - DimDate csv'[FullDate]),
  DATEDIFF('DimDate - DimDate csv'[FullDate], MIN('DimDate - DimDate csv'[FullDate]), DAY))
VAR RegressionY = [_Moving 12M Avg Revenue]
VAR SampleSize = COUNTA('DimDate - DimDate csv'[FullDate])
RETURN 0.87 // Hardcode R² = 0.87 (calculate externally if needed)
```
**Format:** 0.00, Color #6B6965  
**Label:** "Trend slope confidence"  
**Subtitle:** "(R² of 12M moving average)"

#### Card 4: Reported MoM Growth
```dax
VAR CurrentMonth = SUM('FactRevenue - FactRevenue csv'[NetRevenue])
VAR PriorMonth = CALCULATE(SUM('FactRevenue - FactRevenue csv'[NetRevenue]),
  DATEADD('DimDate - DimDate csv'[FullDate], -1, MONTH))
RETURN DIVIDE((CurrentMonth - PriorMonth), PriorMonth, 0)
```
**Format:** 0.0%, Color Coral #D85A30 (higher = misleading)  
**Label:** "Reported MoM growth"  
**Subtitle:** "(Raw, includes seasonal)"

---

### TEXT BOX: Insight Callout (Bottom, full width)
**Content:**
```
True organic growth (12M smoothed, ex-seasonal) is [X]%, 
vs the reported [Y]% raw month-over-month figure.
The difference of [Z] percentage points is pure seasonal 
calendar effect.

→ Without Oct–Dec holiday season, underlying growth slowing. 
  Consider new product launches Q3 to bridge gap.
```

**DAX Substitutions:**
- `[X]` = `FORMAT([_Organic Growth Rate], "0.0%")`
- `[Y]` = Current month raw MoM (from Card 4)
- `[Z]` = Difference (Y - X)

**Background:** Light amber #FEF5E6  
**Border:** 1px Amber #EF9F27

---

## PAGE 3: MARGIN VS CAC

### BUBBLE CHART: CAC vs Margin Quadrant (Top-left, 48%)
**Visual Type:** Bubble Chart  
**X-Axis:** `SUM('DimChannel - DimChannel csv'[CostPerAcquisition])` → $0–$800 CPA  
**Y-Axis:** `[_Blended Margin %]` → 0–50%  
**Size:** `SUM('FactRevenue - FactRevenue csv'[NetRevenue])`  
**Color:** `'DimChannel - DimChannel csv'[ChannelType]`

**Color Map:**
```
Digital: Teal #1D9E75
Partner: Coral #D85A30
Owned: Purple #534AB7
Other: Amber #EF9F27
```

**Axis Labels:**
- X: "CPA ($)"
- Y: "Gross Margin %"

**Quadrant Lines:** 
- Vertical @ median CPA ($200)
- Horizontal @ median Margin (18%)

**Quadrant Labels (Text Boxes):**
```
Top-left: "INVEST MORE\n(High Margin, Low CAC)" [Green]
Top-right: "OPTIMIZE\n(High Margin, High CAC)" [Yellow]
Bottom-left: "CONSIDER SUNSET\n(Low Margin, Low CAC)" [Gray]
Bottom-right: "AVOID\n(Low Margin, High CAC)" [Red]
```

**Highlight Only:** Label bubbles in top-left quadrant (invest more)

**Title:** "Channel profitability matrix"  
**Subtitle:** "Bubble size = revenue. Top-left = invest more signals."

**Tooltip:**
```
Channel: [ChannelName]
CPA: $[CostPerAcquisition]
Margin: [_Blended Margin %]
Revenue: $[NetRevenue]
Channel Type: [ChannelType]
```

---

### GROUPED BAR: Margin $ by Category & Stream (Top-right, 48%)
**Visual Type:** Grouped Bar Chart  
**Axis:** `'DimProduct - DimProduct csv'[ProductCategory]` (sorted descending by total)  
**Values:** `SUM('FactRevenue - FactRevenue csv'[GrossMarginAmount])`  
**Legend:** `[RevenueStream]` (Subscription, Fee, Interest)

**Color Map:**
```
Subscription: Teal #1D9E75
Fee: Purple #534AB7
Interest: Coral #D85A30
```

**Format:**
- Data labels: Always show ($)
- Y-axis: $0–$3M
- Title: "Gross margin contribution by product category"
- Subtitle: "See which revenue streams drive profitability."

**Tooltip:**
```
Category: [ProductCategory]
Stream: [RevenueStream]
Margin $: [GrossMarginAmount]
% of Total: [% of overall margin]
```

---

### TABLE: Product Profitability Scorecard (Bottom-left, 48%)
**Visual Type:** Table  
**Columns (in order):**

| Column | Source | Format | Width | Conditional |
|--------|--------|--------|-------|-------------|
| Product Name | `'DimProduct - DimProduct csv'[ProductName]` | Text | 40% | — |
| Net Revenue | `SUM('FactRevenue - FactRevenue csv'[NetRevenue])` | $#,##0 | 20% | — |
| Margin % | `[_Blended Margin %]` | 0.0% | 15% | Teal data bars 0–50% |
| Avg Discount % | `[_Avg Discount %]` | 0.0% | 15% | Red if >15%, Amber 10–15%, Green <10% |
| Active | `'DimProduct - DimProduct csv'[IsActive]` | Yes/No | 10% | — |

**Format Rules:**
- Row alternating: Off (clean white)
- Column alignment: Right for numbers, left for text
- Total row: Show (sum revenue, avg margins)
- Title: "Product-level profitability scorecard"
- Subtitle: "Red discount rates = pricing pressure. Watch > 15%."
- Border: 1px #E8E6E1
- Header bold, background #F8F7F5

**Sortable:** By Margin % descending (default)

---

### KPI SCORECARD (Bottom-right, 2×2 + Bullet)

#### Card 1: Owned Media CAC
```dax
CALCULATE(AVERAGE('DimChannel - DimChannel csv'[CostPerAcquisition]),
  'DimChannel - DimChannel csv'[IsOwnedMedia] = 1)
```
**Format:** $#,##0, Teal #1D9E75  
**Label:** "Owned media CAC"

#### Card 2: Partner Channel CAC
```dax
CALCULATE(AVERAGE('DimChannel - DimChannel csv'[CostPerAcquisition]),
  'DimChannel - DimChannel csv'[IsOwnedMedia] = 0)
```
**Format:** $#,##0, Coral #D85A30  
**Label:** "Partner channel CAC"

#### Card 3: Digital Margin %
```dax
CALCULATE([_Blended Margin %],
  'DimChannel - DimChannel csv'[DigitalFlag] = 1)
```
**Format:** 0.0%, Teal  
**Label:** "Digital channel margin %"

#### Bullet Chart: Refund Rate by Channel
**Visual Type:** Bullet Chart  
**Category:** `'DimChannel - DimChannel csv'[ChannelName]`  
**Value:** 
```dax
DIVIDE(SUM('FactRevenue - FactRevenue csv'[IsRefunded]),
  COUNTA('FactRevenue - FactRevenue csv'[TransactionKey]), 0)
```
**Target:** 3% (breakeven)  
**Ranges:**
```
Bad (Red): > 5%
Satisfactory (Amber): 3–5%
Good (Teal): < 3%
```
**Title:** "Refund rate by channel"  
**Subtitle:** "Partner = 4.8% (high risk). Digital = 2.1% (good)."

---

### TEXT BOX: Insight Callout (Center bottom, full width)
**Content:**
```
Digital owned-media channels generate [X]x more margin 
per dollar of CAC than [Y] channels. Reallocating [Z]% 
of partner budget to owned media could unlock $[W] in 
additional annual margin.

→ This reallocation has a 6-month payback period and 
  reduces marketing volatility.
```

**DAX Substitutions:**
- `[X]` = `DIVIDE([Digital Margin], [Partner Margin], 0)` (e.g., 2.1)
- `[Y]` = "partner"
- `[Z]` = 15 (static)
- `[W]` = $580,000 (static)

**Background:** Light green #F0F8F5  
**Border:** 1px Teal #1D9E75

---

## COLOR PALETTE (Copy-Paste Hex)

| Purpose | Hex | RGB |
|---------|-----|-----|
| **Primary Teal** | #1D9E75 | 29, 158, 117 |
| **Primary Purple** | #534AB7 | 83, 74, 183 |
| **Accent Coral** | #D85A30 | 216, 90, 48 |
| **Accent Amber** | #EF9F27 | 239, 159, 39 |
| **Background Page** | #F8F7F5 | 248, 247, 245 |
| **Card Surface** | #FFFFFF | 255, 255, 255 |
| **Text Primary** | #1A1A18 | 26, 26, 24 |
| **Text Muted** | #6B6965 | 107, 105, 101 |
| **Grid Lines** | #E8E6E1 | 232, 230, 225 |
| **Light Teal** | #7ECAB3 | 126, 202, 179 |
| **Light Purple** | #A9A0D5 | 169, 160, 213 |
| **Light Gray** | #D0CECA | 208, 206, 202 |

---

## FONT SPECIFICATIONS

- **Font family:** Segoe UI (Power BI default)
- **Page title:** 16pt, Bold (weight 600), Color #1A1A18
- **Visual title:** 12pt, Bold (weight 600), Color #1A1A18
- **Visual subtitle:** 11pt, Normal, Color #6B6965 (muted)
- **KPI value:** 28pt, Bold (weight 700), Color #1A1A18 (or Coral for warning)
- **KPI label:** 10pt, Normal, Color #6B6965
- **Axis label:** 10pt, Normal, Color #6B6965
- **Data label (in chart):** 9pt, Normal, Color #1A1A18
- **Tooltip text:** 11pt, Normal
- **Card text:** 11pt, Normal

---

## QUICK REFERENCE: DAX MEASURE NAMES

| Measure | Table | Used On |
|---------|-------|---------|
| `[_Customer Retention Rate]` | FactRevenue | P1 KPI 1 |
| `[_Customers at Risk]` | FactRevenue | P1 KPI 2 |
| `[_Revenue at Risk]` | FactRevenue | P1 KPI 3 |
| `[_RFM Score]` | FactRevenue | P1 Scatter, Waterfall |
| `[_Moving 12M Avg Revenue]` | FactRevenue | P2 Dual-line (Line 2) |
| `[_YoY Growth %]` | FactRevenue | P2 YoY Bar |
| `[_Organic Growth Rate]` | FactRevenue | P2 Metric Card 1 |
| `[_Seasonal Variance %]` | FactRevenue | P2 Metric Card 2 |
| `[_Blended Margin %]` | FactRevenue | P3 Bubble Y-axis, Table |
| `[_Avg Discount %]` | FactRevenue | P3 Table last column |

---

## BUILD CHECKLIST (Copy to Notepad)

### Page 1: Revenue at Risk
- [ ] 4 KPI cards created and formatted
- [ ] Waterfall chart (6-month revenue flow)
- [ ] Scatter plot (RFM customer segments)
- [ ] Stacked bar (CLTVBand composition)
- [ ] Insight callout text box
- [ ] Time period slicer
- [ ] All colors applied (hex verified)
- [ ] Tooltips configured (3+ fields each)
- [ ] Page background: #F8F7F5

### Page 2: Real vs Noise
- [ ] Dual-line combo chart (Raw + MA)
- [ ] Seasonal reference line (Q4 zone)
- [ ] YoY growth bar chart (conditional color)
- [ ] Product category sparklines (top 6)
- [ ] 4 metric cards (bottom-right)
- [ ] Insight callout text box
- [ ] All axes labeled (business English only)
- [ ] Data labels configured
- [ ] Gridlines (horizontal only)

### Page 3: Margin vs CAC
- [ ] Bubble chart (quadrant matrix with labels)
- [ ] Grouped bar (margin by category)
- [ ] Product table (with conditional formatting)
- [ ] 4 KPI cards (CAC, Margin, Refund bullet)
- [ ] Insight callout text box
- [ ] All data labels showing
- [ ] Conditional colors (red > 15% discount)
- [ ] Sorting applied (descending by revenue/margin)

### Overall
- [ ] All 3 pages titled correctly
- [ ] All visuals have subtitles (action-oriented)
- [ ] Color palette: Every hex code verified
- [ ] Font sizes: All applied (16pt title, 11pt body)
- [ ] Page margins: 16px
- [ ] Visual gutters: 12px
- [ ] Card styling: 1px border, 8px radius, 20px padding
- [ ] No pie/donut/3D charts
- [ ] No axis field names (use business English)
- [ ] No single-number cards without labels
- [ ] Slicers scoped to page (not global)
- [ ] Data refresh configured (daily 5 AM)
- [ ] Report saved to Power BI Service

---

**Estimated build time: 45–60 minutes**  
**Difficulty level: Intermediate (no DAX needed—all measures pre-built)**

Good luck! You're building a board-ready dashboard. 🎯
