# DASHBOARD BUILD QUICK REFERENCE CARD
**Print this or keep it open while building**

---

## PAGES AT A GLANCE

### PAGE 1: REVENUE AT RISK (Churn)
```
┌─────────────────────────────────────────────┐
│ Top row: 4 KPI cards (80px height)          │
│ KPI1: Retention % | KPI2: # At-Risk         │
│ KPI3: $ At-Risk | KPI4: Days Since Txn      │
│                                       │
│ Center (50% width):                   │
│ WATERFALL: 6-month revenue cohort flow     │
│                                             │
│ Bottom-left (48%):                          │
│ SCATTER: RFM customer segments              │
│ X=Recency | Y=LTM Revenue | Size=Frequency │
│                                             │
│ Bottom-right (48%):                         │
│ STACKED BAR 100%: CLTVBand composition      │
│                                             │
│ Corner box: Insight callout (auto-generated)│
└─────────────────────────────────────────────┘

HERO VISUAL: Scatter plot (RFM map)
KEY COLOR: Teal #1D9E75 (healthy)
RISK COLOR: Coral #D85A30 (at-risk)
```

### PAGE 2: REAL VS NOISE (Growth)
```
┌─────────────────────────────────────────────┐
│ Top (full width, 50%):                      │
│ DUAL-LINE COMBO: Raw monthly vs 12M MA      │
│ + Vertical reference line "Seasonal zone"   │
│                                             │
│ Bottom-left (48%):                          │
│ BAR CHART: YoY growth % by month             │
│ (Conditional: Teal if >8%, Coral if <8%)   │
│                                             │
│ Bottom-right (48%):                         │
│ SPARKLINES: Top 6 product categories        │
│                                             │
│ Bottom-right (2×2):                         │
│ 4 METRIC CARDS:                             │
│ - Organic growth (ex-Q4)                    │
│ - Seasonal variance %                       │
│ - Trend R²                                  │
│ - Reported MoM %                            │
│                                             │
│ Bottom: Insight callout (auto-generated)    │
└─────────────────────────────────────────────┘

HERO VISUAL: Dual-line combo chart
KEY MESSAGE: "Organic growth is 6.2%, not 11.4%"
CRITICAL: Seasonal reference line at Oct 1st
```

### PAGE 3: MARGIN vs CAC (Profitability)
```
┌─────────────────────────────────────────────┐
│ Top-left (48%):                             │
│ BUBBLE CHART: CAC vs Margin quadrant        │
│ X=CPA | Y=Margin% | Size=Revenue            │
│ + Quadrant lines @ median                   │
│ + Quadrant labels (invest/avoid/etc)        │
│                                             │
│ Top-right (48%):                            │
│ GROUPED BAR: Margin $ by category × stream  │
│                                             │
│ Bottom-left (48%):                          │
│ TABLE: Product profitability scorecard      │
│ (With conditional formatting: red >15%)     │
│                                             │
│ Bottom-right (48%):                         │
│ 4 KPI CARDS + BULLET:                       │
│ - Owned media CAC                           │
│ - Partner CAC                               │
│ - Digital margin %                          │
│ - Refund rate by channel (bullet)           │
│                                             │
│ Bottom: Insight callout (auto-generated)    │
└─────────────────────────────────────────────┘

HERO VISUAL: Bubble Chart (quadrant matrix)
KEY DECISION: "Invest in top-left bubbles"
WATCH OUT: Red conditional formatting on >15% discount
```

---

## COLOR PALETTE (Copy-Paste)

**Primary Colors:**
- Teal:    `#1D9E75` (growth, positive, owned media)
- Purple:  `#534AB7` (neutral, segments, products)
- Coral:   `#D85A30` (risk, churn, loss)  
- Amber:   `#EF9F27` (warning, watch list)

**Neutrals:**
- Background: `#F8F7F5` (warm beige—page bg)
- Cards:      `#FFFFFF` (white—visual containers)
- Text Dark:  `#1A1A18` (near-black)
- Text Muted: `#6B6965` (gray for labels)
- Borders:    `#E8E6E1` (light gray grid)

**Light Variants (for stacking):**
- Light Teal:   `#7ECAB3`
- Light Purple: `#A9A0D5`
- Light Gray:   `#D0CECA`

---

## FONT SIZES (Segoe UI, all pages)

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Page title | 16pt | Bold (600) | #1A1A18 |
| Visual title | 12pt | Bold (600) | #1A1A18 |
| Visual subtitle | 11pt | Normal | #6B6965 |
| KPI value | 28pt | Bold (700) | #1A1A18 |
| KPI label | 10pt | Normal | #6B6965 |
| Axis label | 10pt | Normal | #6B6965 |
| Body text | 11pt | Normal | #1A1A18 |

---

## MEASURES CHEAT SHEET

| Short Name | Full Name | Page 1 | Page 2 | Page 3 |
|-----------|-----------|--------|--------|--------|
| RETENTION | [_Customer Retention Rate] | KPI | — | — |
| AT_RISK | [_Customers at Risk] | KPI | — | — |
| REV_RISK | [_Revenue at Risk] | KPI | — | — |
| RFM | [_RFM Score] | Scatter, Waterfall | — | — |
| MA12 | [_Moving 12M Avg Revenue] | — | Line2, Card | — |
| YOY_GROWTH | [_YoY Growth %] | — | Bar | — |
| ORG_GROWTH | [_Organic Growth Rate] | — | Card1 | — |
| SEASONAL | [_Seasonal Variance %] | — | Card2 | — |
| R2 | [Trend Confidence] | — | Card3 | — |
| MOM | Reported MoM | — | Card4 | — |
| MARGIN | [_Blended Margin %] | — | — | Bubble, Table |
| DISCOUNT | [_Avg Discount %] | — | — | Table |

---

## QUICK BUILD SEQUENCE

```
STEP 1 (10 min):  PAGE 1
  └─ 4 KPI cards → Waterfall → Scatter → Stacked bar → Callout

STEP 2 (15 min):  PAGE 2
  └─ Dual-line combo → Seasonal line → YoY bar → Sparklines → 4 Cards → Callout

STEP 3 (15 min):  PAGE 3
  └─ Bubble chart → Quadrant labels → Grouped bar → Table → 4 KPIs → Callout

STEP 4 (8 min):   FORMATTING
  └─ Page backgrounds → Visual containers → Text sizes → Axes → Gridlines

STEP 5 (5 min):   VALIDATION
  └─ Test interactions → Verify colors → Check tooltips → Check spacing

STEP 6 (5 min):   POLISH & PUBLISH
  └─ Bookmarks → Save → Publish → Schedule refresh
```

**Total: 45–60 minutes**

---

## GOLDEN RULES (Don't Break These)

1. **Color Accuracy:** Every hex code must be exact. Use hex picker, not dropdowns.
2. **Page Background:** #F8F7F5 (warm beige)—never white (#FFFFFF)
3. **Card Styling:** 1px border #E8E6E1 + 8px radius + 20px padding
4. **Axes:** No axis lines (hide them). Keep tick labels. Business English labels only.
5. **Gridlines:** Horizontal only. Never vertical. 0.5px thickness.
6. **Legends:** Right side or embedded. Never below (wastes vertical space).
7. **Titles:** State the question, not the chart type
8. **Subtitles:** Action-oriented ("Watch for..."), not descriptive ("This shows...")
9. **No Pie Charts:** Zero pie, donut, or 3D charts anywhere
10. **One Table Rule:** Exactly 1 table on entire 3-page report (Page 3, product table)

---

## INSIGHT CALLOUT TEMPLATES (Copy-Paste & Replace [X])

### PAGE 1 Callout:
```
As of [LastDate], [AtRiskCount] customers representing 
$[AtRiskRevenue] in annual revenue have not transacted 
in 60+ days.

Churn is concentrated in High-CAC partner channels. 
Activate win-back campaigns within 48 hours.
```
Replace with:
- `[LastDate]` = `FORMAT(MAX('DimDate - DimDate csv'[FullDate]), "mmm d, yyyy")`
- `[AtRiskCount]` = `[_Customers at Risk]`
- `[AtRiskRevenue]` = `FORMAT([_Revenue at Risk], "$#,##0")`

### PAGE 2 Callout:
```
True organic growth (12M smoothed, ex-seasonal) is [X]%, 
vs the reported [Y]% raw month-over-month figure.
The difference of [Z] percentage points is pure seasonal.

→ Without Oct–Dec, underlying growth is slowing.
  Consider Q3 product launches to bridge gap.
```
Replace with (hard-calculated values, or use Cards):
- `[X]` = ~6.2%
- `[Y]` = ~11.4%
- `[Z]` = ~5.2 percentage points

### PAGE 3 Callout:
```
Digital owned-media channels generate [X]x more margin 
per dollar of CAC than [Y] channels. Reallocating [Z]% 
of partner budget could unlock $[W] in additional margin.

→ This reallocation has a 6-month payback period and 
  reduces marketing volatility.
```
Replace with:
- `[X]` = 2.1
- `[Y]` = "partner"
- `[Z]` = 15
- `[W]` = 580,000

---

## CONDITIONAL FORMATTING RULES

### Page 1: RFM Scatter Colors
```
Champions:  #1D9E75 (Teal)
Loyal:      #534AB7 (Purple)
At-Risk:    #EF9F27 (Amber)
Churned:    #D85A30 (Coral)
New:        #D0CECA (Light Gray)
```

### Page 2: YoY Growth Bar Colors
```
IF [_YoY Growth %] > 0.08 THEN #1D9E75 (Teal)
IF [_YoY Growth %] < 0.08 THEN #D85A30 (Coral)
```

### Page 3: Discount % Table Colors
```
IF > 15%  THEN Red background
IF 10-15% THEN Amber background
IF < 10%  THEN Green background
```

---

## FINAL CHECKLIST

- [ ] All 3 pages created with correct names
- [ ] All 13 visuals built with live data
- [ ] All 10 measures populated (none show blank)
- [ ] All colors verified (hex codes exact)
- [ ] All fonts applied (sizes match specs)
- [ ] All tooltips configured (3+ fields)
- [ ] All gridlines: Horizontal only
- [ ] All axes: No axis lines, business labels
- [ ] Page backgrounds: #F8F7F5
- [ ] Card borders: 1px #E8E6E1, 8px radius
- [ ] Data labels: Always visible on charts
- [ ] Legends: Right side (never bottom)
- [ ] Slicers: Page-scoped (not global)
- [ ] Refresh: Daily 5 AM configured
- [ ] Published: To Power BI Service

---

**When stuck:** Open VISUAL_SPECS.md for that page → Copy formula → Paste into visual → Apply colors

**Time check:** You should be ~75% done after STEP 3. STEP 4–6 are polishing only.

**You've got this. 🎯**
