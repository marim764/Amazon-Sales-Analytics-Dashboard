# Documentation

This document covers the key planning and design aspects of the **Amazon Sales Analytics Dashboard (Power BI)**:

- Project Planning  
- Stakeholder Analysis  
- Database Design  
- UI/UX Design  

---

## 1. Project Planning

### 1.1 Project Objective

Build an interactive **Power BI dashboard** to analyze Amazon product performance over a single month, helping users understand:

- Overall sales and units performance  
- Which categories and products drive revenue  
- How **discounts, coupons, Buy Box**, and **Sponsored** status impact sales  
- Detailed metrics for any specific product  

### 1.2 Scope

**In Scope:**

- Data modeling for a **single-month** dataset  
- Star schema with `FactAmazon`, `DimDate`, `DimCategory`  
- Key DAX measures (Total Sales, Total Units, Average Price, Average Rating, % Shares, Avg Discount)  
- **Three report pages**:
  1. Overview  
  2. Drivers  
  3. Product Deep Dive  
- Interactive slicers (Day, Category, Coupon, Buy Box, Sponsored)  
- Drillthrough / navigation from Top 10 products to Product Deep Dive  
- Basic documentation and GitHub repository

**Out of Scope (for now):**

- Multi-month / year-over-year analysis  
- Region / marketplace breakdown  
- Row-Level Security (RLS)  
- Automatic refresh from a live data source  

### 1.3 Data Sources & Assumptions

- Source: Amazon-like dataset with fields such as:
  - `product_title`, `product_rating`, `total_reviews`  
  - `purchased_last_month`, `discounted_price`, `original_price`, `discount_percentage`  
  - `has_coupon`, `is_sponsored`, `buy_box_availability`  
  - `product_image_url`  
  - `data_collected_at` (date/time)
- Assumptions:
  - `purchased_last_month` represents units sold in the analysis period.  
  - `discounted_price` is the effective selling price.  
  - `discount_percentage` = `(original_price - discounted_price) / original_price`.  
  - Boolean flags (`has_coupon`, `is_sponsored`, etc.) are reliable for segmentation.

### 1.4 Success Criteria

The project is considered successful if:

- The model loads without errors and relationships are valid.  
- All three pages respond to the **same slicer panel**.  
- KPI measures are correct and consistent across visuals.  
- A user can:
  - Understand overall performance from the **Overview** page.  
  - See the impact of discounts / coupons / Buy Box / Sponsored on the **Drivers** page.  
  - Select a product and view its metrics + image on the **Product Deep Dive** page.  
- The dashboard is visually clean, consistent, and fast to use.

---

## 2. Stakeholder Analysis

### 2.1 Main Stakeholders

| Stakeholder             | Role / Interest                                                | Key Questions                                                                 |
|-------------------------|----------------------------------------------------------------|-------------------------------------------------------------------------------|
| Amazon Seller / Owner   | Business decision maker                                        | Which products and categories generate most revenue? Are promotions working?  |
| Data Analyst / BI User  | Analyzes performance, prepares reports                         | How do discounts, coupons, Buy Box, and Sponsored affect sales and units?     |
| Marketing / Promotions  | Plans discounts/coupons and sponsored campaigns               | Which campaigns increase sales? Which categories respond best to discounts?   |
| Operations / Inventory  | Manages stock levels and supply                                | Which products need replenishment? Where are consistent top sellers?          |
| Developer / Student (You)| Builds and maintains the model and report                     | Is the model correct, performant, and easy to extend and present?             |

### 2.2 Stakeholder Needs

- **Owner / Management**
  - High-level KPIs (Total Sales, Units, Top Categories, Top Products)
  - Quick identification of best / worst performers

- **Analyst**
  - Ability to slice by **day, category, coupon, Buy Box, Sponsored**
  - Visibility into the **drivers** behind performance

- **Marketing**
  - Clear view of **Coupon Sales Share**, **Sponsored Sales Share**
  - Understanding of whether **higher discounts** really drive more units

- **Operations**
  - Granular product-level view (Deep Dive) to identify key products

- **Developer**
  - Clean model (star schema), easy to explain in documentation and portfolio

---

## 3. Database Design

### 3.1 Overall Architecture

The report uses a **star schema**:

- One central fact table: `FactAmazon`
- Supporting dimensions: `DimDate`, `DimCategory`
- Optional “flag” columns for Coupon / Sponsored / Buy Box status

This structure simplifies DAX and enables **fast filtering** and **clear relationships**.

### 3.2 Fact Table: FactAmazon

**FactAmazon** (one row per product listing / record):

- **Keys:**
  - `product_id` (or implicit key)
  - `date_key` (linked to DimDate)
  - `category_key` (linked to DimCategory)

- **Numeric measures:**
  - `purchased_last_month` (units)
  - `discounted_price`
  - `original_price`
  - `discount_percentage`
  - `total_reviews`
  - `product_rating`

- **Flags / attributes:**
  - `has_coupon_flag` (True/False)
  - `is_sponsored_flag` (True/False)
  - `has_buy_box_flag` or `buy_box_availability`
  - `product_image_url`
  - `data_collected_at` (date/time)

- **DAX measures built on top:**
  - `Total Sales = SUMX(FactAmazon, purchased_last_month * discounted_price)`
  - `Total Units = SUM(FactAmazon[purchased_last_month])`
  - `Avg Selling Price`, `Avg Rating`
  - `Sales With Coupon`, `Sales Without Coupon`
  - `Sales With Buy Box`, `Sales Without Buy Box`
  - `Sales Sponsored`, `Sales Non Sponsored`
  - `% Shares` for each of the above

### 3.3 Dimension: DimDate

- Contains all dates for the analysis month.
- Columns:
  - `Date`
  - `Day` (1–31)
  - `Month`
  - `Year`
  - `Year-Month` (for grouping if needed)
- Relationship:
  - `DimDate[Date]` → `FactAmazon[data_collected_at]` (or date_key)

Used in:

- Daily Sales Trend
- Time-based filtering (Day slicer)

### 3.4 Dimension: DimCategory

- Contains product categories.
- Columns:
  - `category_key`
  - `product_category` (e.g., Electronics, Home, etc.)
- Relationship:
  - `DimCategory[category_key]` → `FactAmazon[category_key]`

Used in:

- Sales by Category
- Drivers analysis by Category (Discount vs Units, Coupon impact, Buy Box impact)

---

## 4. UI/UX Design

### 4.1 Global Design Principles

- **Consistency:** Same header, left filter panel, fonts, and colors across all pages.
- **Simplicity:** Minimal clutter; each page answers a specific question:
  - Overview → “What is happening?”
  - Drivers → “Why is it happening?”
  - Deep Dive → “What’s happening for this product?”
- **Focus:** KPIs at the top, main visuals in the center, filters on the left.
- **Branding:** Amazon-inspired colors (dark background, orange and blue accents).

### 4.2 Color Palette & Style

- **Background:** Very dark grey / near black  
- **Accent colors:**
  - Amazon orange: `#ff9900`
  - Deep blue: `#146eb4`
  - Dark navy: `#232f3e`
- **Visuals:**
  - Rounded corners
  - Soft shadows on cards and charts
  - Clean white/light text (`#f2f2f2`) on dark background

### 4.3 Layout by Page

#### Overview Page

- **Top:** 4 KPI cards (Total Sales, Total Units, Avg Price, Avg Rating)
- **Left:** Vertical slicer panel:
  - Amazon logo
  - Day slicer
  - Category slicer
  - Coupon Status (Yes/No)
  - Buy Box Status (Yes/No)
  - Sponsored Status (Yes/No)
  - Clear Filters button
- **Center:**
  - Sales by Category (bar chart)
  - Daily Sales Trend (line chart)
- **Bottom:**
  - Top 10 Products by Sales (table)

#### Drivers Page

- **Top:** Driver KPIs:
  - Avg Discount %
  - Coupon Sales Share
  - Buy Box Sales Share
  - Sponsored Sales Share
- **Left panel:** Same slicers & Clear Filters button
- **Main visuals:**
  - Discount vs Units by Category (scatter, bubble size = sales)
  - Coupon Impact by Category (clustered bar)
  - Buy Box Impact by Category (clustered bar)
  - Sponsored vs Organic Sales (donut)

#### Product Deep Dive Page

Kept **intentionally simple** for clarity:

- **Top:** Same header + 4 per-product KPI cards:
  - Product Sales
  - Units Sold
  - Avg Selling Price
  - Avg Rating
- **Left center:** Large product image (from `product_image_url`)
- **Right center:** Product Details table (one row for the selected product)
- **Left panel:** Day and Category slicers (others can be hidden to reduce clutter)

### 4.4 Interactions & Navigation

- **Slicers** are **synced across pages**, so selected filters persist.
- **Clear Filters** button resets slicers to the default state.
- From the **Top 10 Products** table in Overview, users can:
  - Select a product and navigate to **Product Deep Dive**, which focuses on that product.
- Tooltips and visual headers are simplified to keep the UI clean and focused.

---

This documentation describes the reasoning behind the **planning, stakeholders, data model, and UI/UX** of the Amazon Sales Analytics Dashboard and can be used for academic reports, project submissions, or as part of a professional portfolio.
