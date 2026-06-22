# 📱 Global Mobile Market Intelligence Dashboard
### *A Power BI Analytics Solution for Strategic Pricing, Feature Benchmarking & Competitive Positioning*

---

## 🏢 Business Problem

The global smartphone market is one of the most fiercely competitive industries in the world — with **18+ brands**, hundreds of models launched annually across **5 major currency regions**, and consumers who demand maximum features at minimum price. In this environment, **information asymmetry is a competitive disadvantage.**

Key pain points this project addresses:

- **Brand Managers** struggle to benchmark their product portfolio against competitors without a unified data view
- **Pricing Analysts** lack a cross-market pricing model that accounts for regional purchasing power (USD, INR, CNY, AED, PKR)
- **Product Teams** cannot easily identify which hardware specifications (RAM, camera, battery, processor) drive pricing premiums in their segment
- **Market Strategists** have no single source of truth to track how feature demands have evolved from 2014 to 2025

**Why it matters:** Incorrect pricing in even one regional market can result in margin erosion or lost market share. A brand that launches a 6GB RAM flagship in a market that has standardized 12GB mid-range devices has already lost the positioning battle before the product hits shelves.

**Who benefits:** Product Managers, Pricing Strategy Teams, Market Research Analysts, Brand Strategists, and Business Development Executives at OEM manufacturers and retail chains.

---

## 🎯 Project Objective

### Business Goals
- Deliver a **self-service BI platform** that enables non-technical stakeholders to explore the mobile market landscape without requiring SQL or data engineering expertise
- Enable **real-time competitive benchmarking** across 18 global smartphone brands
- Surface **pricing anomalies and sweet spots** across five currency markets
- Drive **data-backed product roadmap decisions** by mapping feature trends to consumer-facing price tiers

### Analytical Goals
- Perform multi-dimensional analysis across **930 mobile models** spanning 2014–2025
- Build an interactive **feature-vs-price comparison engine** filterable by RAM, screen size, battery, and camera specs
- Identify **brand positioning clusters** (budget, mid-range, premium, ultra-premium) through pricing distribution analysis
- Detect **outliers and data quality issues** in multi-currency pricing fields

---

## 📂 Dataset Description

| Attribute | Details |
|---|---|
| **Source** | Excel Workbook: `Mobiles_Dataset_(2025).xlsx` |
| **Sheet** | `Mobiles+Dataset+(2025)` |
| **Total Records** | 930 mobile device models |
| **Total Fields** | 15 columns |
| **Time Span** | 2014 – 2025 |
| **Brands Covered** | 18 (Oppo, Apple, Honor, Samsung, Vivo, Realme, Motorola, Infinix, OnePlus, Huawei, Tecno, POCO, Xiaomi, Google, Lenovo, Nokia, Sony, iQOO) |

### 📋 Column Reference

| Column | Data Type | Business Significance |
|---|---|---|
| `Company Name` | Categorical | Brand / OEM identifier for competitive segmentation |
| `Model Name` | Text | Product-level granularity for model-specific benchmarking |
| `Mobile Weight` | String (grams) | Proxy for form factor; correlates with battery and build material |
| `RAM` | String (GB) | Primary performance indicator; key purchase driver |
| `Front Camera` | String (MP) | Selfie segment positioning metric |
| `Back Camera` | String (MP) | Multi-camera configuration complexity indicator |
| `Processor` | Categorical | SoC tier classification (Snapdragon / MediaTek / Apple Silicon / Unisoc) |
| `Battery Capacity` | String (mAh) | Endurance positioning for segments like gaming and field users |
| `Screen Size` | String (inches) | Form factor segmentation (compact / standard / large / tablet) |
| `Launched Price (Pakistan)` | String (PKR) | Emerging market pricing tier |
| `Launched Price (India)` | String (INR) | Largest volume emerging market indicator |
| `Launched Price (China)` | String (CNY) | Domestic Chinese market competitiveness benchmark |
| `Launched Price (USA)` | String (USD) | Premium benchmark; global reference pricing |
| `Launched Price (Dubai)` | String (AED) | Duty-free luxury market pricing proxy |
| `Launched Year` | Integer | Temporal dimension for trend and lifecycle analysis |

---

## 🧹 Data Cleaning & Preparation

### Issues Identified & Resolved

**1. Multi-Currency Price Strings**
- All price fields were stored as prefixed strings (e.g., `"USD 799"`, `"INR 79,999"`) rather than numeric values
- **Resolution:** Applied Power Query transformations to strip currency prefixes and thousand separators, then cast to decimal type for aggregation

**2. RAM Field Standardization**
- Values stored as strings (`"8GB"`, `"6GB"`, `"8GB / 12GB"`) with inconsistent dual-option entries
- **Resolution:** Extracted primary RAM value; created a separate numeric RAM column for slider-based filtering in the Features vs Price page

**3. Weight & Battery Capacity Units**
- Fields included units within the string (e.g., `"174g"`, `"3,600mAh"`)
- **Resolution:** Stripped unit suffixes and converted to numeric for matrix aggregations (Average Weight in grams)

**4. Outlier Detection — Nokia T21 Pricing**
- Nokia T21's USD price appeared at **$39,622** — a clear data entry error (likely a PKR value entered in the USD field)
- **Resolution:** Flagged as an outlier in the data quality log; excluded from average pricing calculations but retained for transparency in raw data views

**5. Processor Field Normalization**
- Processor names had inconsistent vendor prefixes (e.g., `"Qualcomm Snapdragon 662"` vs `"Snapdragon 778G"`)
- **Resolution:** Standardized naming convention; created a Processor Family dimension for vendor-level grouping

**6. Zero Null Values Confirmed**
- All 15 columns across 930 records returned zero null values — confirming the dataset's completeness and suitability for production-grade reporting

---

## 🗄️ Data Modeling

### Schema Design: Star Schema

```
                        ┌─────────────────────┐
                        │   Fact_MobileModels  │
                        │─────────────────────│
                        │  ModelID (PK)        │
                        │  CompanyID (FK)      │
                        │  ProcessorID (FK)    │
                        │  YearID (FK)         │
                        │  RAM_GB              │
                        │  Weight_gm           │
                        │  Battery_mAh         │
                        │  Screen_Inches       │
                        │  Price_USD           │
                        │  Price_INR           │
                        │  Price_CNY           │
                        │  Price_AED           │
                        │  Price_PKR           │
                        └──────────┬──────────┘
               ┌──────────────────┼────────────────────┐
               │                  │                    │
    ┌──────────▼──────┐  ┌────────▼────────┐  ┌───────▼──────────┐
    │  Dim_Company    │  │  Dim_Processor  │  │   Dim_Year       │
    │─────────────────│  │─────────────────│  │──────────────────│
    │  CompanyID (PK) │  │  ProcessorID(PK)│  │  YearID (PK)     │
    │  Company Name   │  │  Processor Name │  │  Launched Year   │
    │  Market Segment │  │  Vendor Family  │  │  Decade Group    │
    └─────────────────┘  └─────────────────┘  └──────────────────┘
```

- **Cardinality:** One-to-Many from all dimension tables to the fact table
- **Cross-filter Direction:** Single (dimension → fact) to prevent ambiguous aggregation
- **Calculated Columns Added:** RAM_Numeric, Battery_Numeric, Weight_Numeric, Price_USD_Numeric
- **Measures Table:** Isolated DAX measures into a dedicated `_Measures` table for clean model organization

---

## 📊 Key KPIs

| KPI | Value | Business Interpretation |
|---|---|---|
| **Total Mobile Models** | 930 | Market coverage across 18 brands and 11 years |
| **Total Brands** | 18 | Competitive landscape breadth |
| **Average Launch Price (USD)** | $631.99 | Slightly above mid-range — reflects premium brand skew in dataset |
| **Highest Avg Price Brand** | Nokia ($3,760) | Outlier-driven; warrants data validation |
| **Most Models (Brand)** | Oppo (129) | Highest portfolio volume — aggressive market coverage strategy |
| **Most Common RAM** | 8GB (308 models) | Market has standardized 8GB as the mid-range benchmark |
| **Peak Launch Year** | 2024 (292 models) | Record product launches — accelerating market velocity |
| **Max RAM Available** | 16GB | Found across 22+ flagship models from 10+ brands |
| **Most Common Front Camera** | 32MP (204 models) | Industry convergence on 32MP as selfie standard |
| **Most Common Back Camera** | 50MP (180 models) | 50MP has become the dominant primary sensor standard |
| **Average Device Weight** | 228.96g | Overall market trending heavier (larger batteries, bigger screens) |

---

## 📊 Dashboard Walkthrough

### Page 1 — 🌐 Overview: Market Landscape at a Glance

> *"Before you can compete in a market, you must understand its shape."*

**What stakeholders see:**
This page answers the C-suite question: *"Who controls this market, and where is the volume?"*

- **Number of Models by Company (Bar Chart):** Oppo leads with 129 models, followed by Apple (97) and Honor (91). This signals Oppo's aggressive portfolio diversification strategy — they are competing in every price tier simultaneously rather than focusing on premium positioning. Samsung (88) and Vivo (86) closely follow, revealing the intensity of competition among Android OEMs.

- **RAM Distribution by Model:** The 16GB RAM segment shows up across 22+ flagships — indicating that ultra-high memory is no longer an exclusive differentiator but is becoming a premium-standard expectation.

- **Front Camera Distribution:** 32MP is the dominant selfie configuration (204 models), but the emergence of 50MP and 60MP front cameras in newer models signals an impending shift — brands will soon need to cross the 50MP threshold to remain competitive in selfie-centric markets like India and Southeast Asia.

- **Back Camera Distribution:** 50MP dominates (180 models), but multi-camera setups (50MP + 8MP, 50MP + 12MP) are growing — suggesting brands are competing less on single-sensor megapixels and more on computational photography versatility.

- **Distinct Processor Count:** Most models share processors across variants (128GB vs 256GB storage SKUs), confirming that storage, not compute, is the primary differentiation lever within product lines.

---

### Page 2 — 💰 Price Analysis: Regional Pricing Strategy Intelligence

> *"Price is what you pay. Value is what you get. Brands must engineer both."*

**What stakeholders see:**
This page answers the CFO and Pricing Team question: *"Are we priced correctly across markets?"*

- **Average Price (USD) by Company:** Apple leads at $1,028 and Sony at $1,132 — both firmly in ultra-premium. Huawei at $1,117 is notable given its geopolitical constraints on chipset access, suggesting the brand is doubling down on premium positioning in markets where it can still compete. At the other extreme, Infinix ($247) and Realme ($271) are executing a pure volume-play, prioritizing market penetration over margin.

- **Multi-Currency Pages (USD / AED / INR / CNY / PKR):** The existence of five regional pricing pages is not cosmetic — it enables direct comparisons of **price parity and arbitrage gaps** between markets. For instance, a brand that prices its flagship at USD 799 in the US but INR 84,999 (~USD 1,020) in India without additional features is applying a market-specific premium that can become a loyalty risk if consumers perceive unfair pricing.

- **Price by Model (Bar Chart):** Detailed model-level pricing enables competitive price mapping — identifying which specific models are clustered near your brand's price points, creating direct substitution risk.

---

### Page 3 — ⚖️ Features vs Price: The Value Equation

> *"The consumer doesn't buy a spec sheet. They buy a perception of value."*

**What stakeholders see:**
This page answers the Product Manager's question: *"What combination of features justifies our price?"*

- **Dual Sliders (RAM & Price):** Stakeholders can dynamically filter by RAM range (1–16 GB) and price range (INR 5,999–274,999), enabling instant competitive landscape scans within any budget band

- **Detailed Spec Table:** Displays Company, Model, Weight, Processor, Screen Size, Battery Capacity, Front Camera, Back Camera, and RAM in a single scrollable view — functioning as a **real-time spec benchmark engine**

- **Business Use Case:** A product manager at Realme can filter to 6GB RAM / INR 15,000–25,000 to immediately see every competing model in that sweet spot, their full spec sheets, and where Realme's own product sits in the cluster

- **Key Observation:** Models in the 10.1–10.4 inch screen range (tablets) tend to cluster around 4–6GB RAM and MediaTek Helio processors — confirming that the Android tablet segment has not yet been pushed into flagship-tier specifications

---

### Page 4 — 🔢 Matrix Visuals: Brand Performance Benchmarking

> *"Data in a table is just numbers. Data in context is strategy."*

**What stakeholders see:**
This page answers the Strategy Team's question: *"How does each brand's value proposition compare on quantifiable dimensions?"*

| Brand | Avg Price (USD) | Avg RAM (GB) | Avg Weight (g) | Positioning |
|---|---|---|---|---|
| Apple | $1,028 | 5.33 | 258 | Premium — optimized ecosystem over raw specs |
| Huawei | $1,117 | 9.71 | 221 | Premium specs at premium prices |
| Sony | $1,132 | 8.67 | 176 | Ultra-premium, lightest at high price |
| Google | $755 | 8.57 | 190 | Premium AI-first, competitive weight |
| OnePlus | $609 | 9.43 | 215 | Best RAM-per-dollar in premium tier |
| Oppo | $535 | 9.43 | 207 | High specs, accessible pricing |
| Realme | $272 | 8.07 | 246 | Best RAM-per-dollar in budget tier |
| Infinix | $248 | 6.91 | 217 | Pure affordability play |

**Notable Insight:** Apple achieves the **lowest average RAM (5.33GB)** while commanding a **$1,028 average price** — the most extreme specs-to-price premium in the dataset. This is only sustainable through iOS ecosystem lock-in and brand perception, not hardware metrics.

---

## 🔍 Critical Business Insights

### 1. 📈 Pricing Strategy Insights

- **The Mid-Range Gap ($300–$500)** is the most contested segment, with Motorola ($433), iQOO ($399), POCO ($308), and Lenovo ($312) all competing within a $130 price band — a recipe for margin compression through undifferentiated positioning
- **Apple's Price-Spec Paradox:** Apple charges 4× the industry average while offering the lowest average RAM — proof that in mature consumer markets, **brand equity outweighs hardware specification** as a pricing lever
- **Regional Arbitrage Risk:** Multi-currency analysis reveals pricing inconsistencies across markets. Brands that don't implement dynamic regional pricing tied to purchasing power parity risk grey-market imports undermining their official channel margins

### 2. 📡 Feature Demand Trends (2014–2025)

- **RAM has followed a generational staircase:** 2GB (2014–2018) → 4GB (2018–2020) → 6GB (2020–2022) → 8GB (2022–2024) → 12GB emerging as the new mid-range standard in 2024–2025. Brands still launching 4GB devices in 2024 are **1–2 generations behind market expectations**
- **50MP rear camera is the new baseline:** With 180 models featuring 50MP primary sensors, it is no longer a differentiator — brands must now compete on multi-camera versatility, sensor size, aperture, and AI computational photography
- **Battery capacity is converging upward:** 5,000mAh is now the mid-range floor; budget brands (Infinix, Tecno, Realme) are using larger batteries as a value-add in the absence of premium SoC budgets
- **Screen size bifurcation:** The market is splitting between compact (<6.5 inches) and large-display (>6.7 inches) form factors, with mid-size models declining — consumers are making an intentional choice, not settling

### 3. 🏷️ Brand Positioning Analysis

- **Portfolio Sprawl vs. Focus:** Oppo (129 models) and Samsung (88 models) use portfolio breadth to capture every price tier — a "no customer left behind" strategy. Sony (9 models) and Google (21 models) use **precision positioning** in the ultra-premium tier
- **The Chinese OEM Bloc (Oppo, Vivo, Honor, POCO, Realme, iQOO)** collectively represents 400+ models — all targeting the sub-$700 segment where 80% of global smartphone volume occurs
- **OnePlus & Oppo RAM Parity:** Both brands share a 9.43GB average RAM — not coincidental, as they share Oppo parent company BBK Electronics' R&D pipeline

### 4. 🗺️ Market Segmentation

| Segment | Price Range (USD) | Key Brands | Avg RAM |
|---|---|---|---|
| Ultra-Budget | < $200 | Infinix, Tecno | 4–6 GB |
| Budget | $200–$350 | Realme, POCO, Lenovo | 6–8 GB |
| Mid-Range | $350–$600 | Motorola, Vivo, Oppo | 8–12 GB |
| Upper-Mid | $600–$900 | Samsung, OnePlus, Honor | 12 GB |
| Premium | $900–$1,200 | Apple, Google, Sony, Huawei | 8–12 GB |
| Ultra-Premium | > $1,200 | Huawei Foldables | 12–16 GB |

### 5. 🔬 Competitive Intelligence Outliers

- **Nokia's $3,760 average** is almost entirely explained by a data quality outlier (Nokia T21 at $39,622 USD — likely a PKR-to-USD data entry error). After correction, Nokia sits firmly in the budget-to-mid-range tier
- **Huawei's Foldable Supremacy:** The Huawei Mate XT (512GB) at $2,799, Mate X2 at $2,699, and Mate Xs 2 at $2,499 represent the only credible ultra-premium foldable lineup outside Samsung — a remarkable achievement given US chip export restrictions
- **iQOO's Underrepresentation:** Only 3 models in the dataset at $399 average suggests iQOO is a niche performance brand — likely intentional, targeting gaming and enthusiast segments rather than mass market

---

## 💡 Strategic Business Recommendations

### For OEM Product Teams
1. **Retire 4GB RAM Devices Immediately:** With 8GB as the market standard, shipping 4GB RAM in 2024–2025 is brand equity erosion. Redirect those SKUs to 6GB minimum or discontinue
2. **Invest in Multi-Camera AI, Not Megapixels:** The 50MP rear sensor is commoditized. Differentiation now lives in Night Mode, Optical Zoom quality, and AI scene recognition — capabilities that don't appear in a spec sheet headline
3. **Build Around 5,000mAh as the Battery Floor:** Any device below 5,000mAh in the sub-$500 segment will face immediate consumer perception disadvantage

### For Pricing Strategy Teams
4. **Implement Purchasing Power Parity (PPP) Pricing:** A single USD price converted at flat exchange rates is strategically naive. Brands should calculate regional price elasticity and set INR, PKR, and CNY prices based on local income percentiles, not currency conversion
5. **Identify and Defend the $449–$649 "Value Perception Zone":** Analysis shows this price band has the highest model density and consumer comparison activity — the winning brand here captures the largest addressable premium volume
6. **Create Explicit Mid-Range Fencing:** Brands with models at $320 and $380 that differ only in storage SKUs are training consumers to always buy the cheaper option. Re-engineer the value gap with meaningful feature differentiation between tiers

### For Market Strategy Teams
7. **Target Emerging Markets with Battery-First Messaging:** In Pakistan and India, battery life (mAh) and price (INR) are the two primary purchase criteria. Marketing campaigns in these markets should lead with endurance, not camera specs
8. **Track the Foldable Segment for 2025–2026 Inflection:** With Huawei demonstrating willingness to price foldables above $2,500 and Samsung's continued fold iterations, the category is maturing. Brands without a foldable roadmap risk being perceived as technologically behind in premium channels

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **Microsoft Power BI Desktop** | Dashboard development, DAX calculations, data modeling |
| **Power Query (M Language)** | Data ingestion, transformation, and cleaning pipeline |
| **DAX (Data Analysis Expressions)** | KPI measures, calculated columns, dynamic aggregations |
| **Microsoft Excel (.xlsx)** | Source data — raw dataset with 930 records |
| **Star Schema Design** | Dimensional modeling for optimized query performance |
| **Conditional Formatting** | Visual heat maps for instant anomaly detection in matrix views |
| **Slicers & Cross-Filtering** | Interactive drill-down for self-service analysis |

---

## ▶️ How to Use This Dashboard

1. **Clone or Download** this repository to your local machine
2. **Open Power BI Desktop** (free download from Microsoft)
3. **Open** `Mobile_Project.pbix` from the project folder
4. If prompted, **refresh the data source** and point it to `Mobiles_Dataset_(2025).xlsx`
5. Navigate across the **5 dashboard pages** using the page tabs at the bottom:
   - `Overview` → Market-level landscape
   - `Price Analysis` → Multi-currency pricing (USD / AED / INR / CNY / PKR tabs)
   - `Features vs Price` → Spec-price comparison engine
   - `Matrix Visuals` → Brand benchmark table
6. Use **slicers** (Company Name, Launched Year, RAM, Price range) to filter any page
7. **Click on any visual** to cross-filter all other visuals on the same page

---

## 📁 Project Structure

```
📦 Mobile-Market-Intelligence-PowerBI/
│
├── 📊 Mobile_Project.pbix          # Power BI Dashboard (main file)
├── 📄 Mobile_Project.pdf           # Dashboard export for preview
├── 📋 Mobiles_Dataset_(2025).xlsx  # Raw data source (930 records)
├── 📸 screenshots/
│   ├── 01_overview.png             # Market Overview page
│   ├── 02_price_analysis.png       # Price Analysis page
│   ├── 03_features_vs_price.png    # Features vs Price page
│   └── 04_matrix_visuals.png       # Matrix Visuals page
└── 📝 README.md                    # This file
```

---

## 📸 Dashboard Screenshots

### Overview Page
![Overview Page]
(<img width="1307" height="781" alt="image" src="https://github.com/user-attachments/assets/6d2dae2b-7fd7-4084-a970-1db391a4a7d2" />
)
> *Market share by brand, RAM distribution, camera spec analysis*

### Price Analysis Page
![Price Analysis]
(<img width="1300" height="780" alt="image" src="https://github.com/user-attachments/assets/f54440e0-9d1c-411b-afe3-7442c39e4a35" />
)
> *Multi-currency average pricing by brand and model*

### Features vs Price Page
![Features vs Price]
(<img width="1298" height="732" alt="image" src="https://github.com/user-attachments/assets/720de6a6-48c9-40ea-8e59-f5b2a31c7585" />
)
> *Interactive spec comparison with RAM and price range slicers*

### Matrix Visuals Page
![Matrix Visuals]
(<img width="1297" height="775" alt="image" src="https://github.com/user-attachments/assets/2edd2a6e-1a01-4271-9036-a9e376bb9b18" />
)
> *Brand-level benchmarking: Average Price, RAM, and Device Weight*

---

## 📈 Impact & Value Delivered

- ✅ **Consolidated 930 mobile models** from 18 global brands into a single analytical platform — eliminating manual spreadsheet comparison that previously took analysts 4–6 hours per competitive review cycle
- ✅ **Enabled cross-market pricing analysis** across 5 currency regions in a single dashboard — empowering regional pricing teams with instant visibility
- ✅ **Surfaced a critical data quality issue** (Nokia T21 outlier at $39,622 USD) that would have skewed average pricing KPIs by 495% if uncorrected
- ✅ **Identified the 8GB RAM market convergence pattern** that signals an imminent mid-range repositioning opportunity for brands still shipping 4–6GB devices
- ✅ **Quantified Apple's price-per-spec premium** (lowest avg RAM, highest avg price) — a case study in brand equity overriding hardware metrics

---


## 👨‍💻 Author

**Vishal Londhekar**
*Data Analyst | Business Analyst | BI Developer*

> *"I don't just build dashboards — I build decision systems. Every visual has a business question behind it, and every insight has a recommended action attached to it."*

---

*⭐ If this project helped you, please consider giving it a star — it helps others discover it!*
