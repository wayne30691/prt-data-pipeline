
# PRT LS E2E Master Data Pipeline Guidebook

## Purpose
This guide outlines the structured steps and naming conventions for building a **clean and unified LS E2E master dataset**, used to analyze actual depletion volume by grouped outlets and matched to contract/target SKUs.  

---

## Table Naming Conventions  

| Prefix | Meaning              | Example                  |
|--------|----------------------|--------------------------|
| `f_`   | Fact Table           | `f_actual_depletions`    |
| `d_`   | Dimension Table      | `d_grouped_outlets`      |
| `v_`   | View or Intermediate | `v_depletions_enriched`  |
| `tmp_` | Temporary Table      | `tmp_replace_skus_raw`   |

*No rules for suffix naming.*

---

## Data Pipeline Steps

### 🔹 Step 1: `d_grouped_outlets`
**Description:**  
Dimension table that maps raw outlets into standardized groups (e.g., 買酒網系列, 洋酒城系列, 佳賀系列).

**Source:**  
- Internal outlet mapping reference maintained by Commercial team.

---

### 🔹 Step 2: `f_actual_depletions`
**Description:**  
Raw fact table showing depletion volume by SKU, outlet, and transaction date.

**Source:**  
- Current year: OBIEE daily export
- Historical: storage in SharePoint
- Refreshed daily

---

### 🔹 Step 3: `f_actual_depletions_grouped`
**Transformation:**  
Join `f_actual_depletions` with `d_grouped_outlets` on outlet code.

**Purpose:**  
Add parent outlet group information to the depletions data.

---

### 🔹 Step 4: `d_contract_skus`
**Description:**  
Dimension table containing SKUs defined as part of sales contracts and targets for a given period.

**Source:**  
- Manually maintained  
- Refreshed monthly/quarterly

---

### 🔹 Step 5: `d_replace_skus`
**Description:**  
SKU replacement mapping table combining:
- Replace list from SharePoint  
- *Replace rules from LS Contract結案 dataset*

**Transformation Steps:**
1. Load SharePoint Replace SKU list → `tmp_replace_skus_raw`
2. Merge with LS Contract rule logic → `d_replace_skus`

---

### 🔹 Step 6: `f_actual_depletions_replaced`
**Transformation:**  
Apply `d_replace_skus` mapping to `f_actual_depletions_grouped`.  
This replaces old/discontinued SKUs with their new equivalents for reporting consistency.

---

### 🔹 Step 7: `f_depletions_final`
**Transformation:**  
Final merge of:
- `f_actual_depletions_replaced`  
- `d_contract_skus`  
- `d_grouped_outlets`  

**Purpose:**  
This is the **final analytical dataset** used in BI dashboards and reports.

**Fields included:**  
- Region
- Grouped Outlet code  
- Grouped Outlet  
- ItemGroup code (Replaced & Contract Mapped)
- allocation%
- Brand
- ItemGroup description 
- Contract Type  
- Sales reps

---

## Refresh Schedule

| Table                     | Frequency  |
|---------------------------|------------|
| `f_actual_depletions`     | Daily      |
| `d_grouped_outlets`       | Monthly    |
| `d_contract_skus`         | Monthly    |
| `d_replace_skus`          | Monthly    |
| `f_depletions_final`      | Daily      |

---

## Notes & Best Practices

- Ensure all SKUs in `f_actual_depletions` are passed through `d_replace_skus` before contract matching.
- Maintain a changelog for `d_replace_skus` for transparency.
- Use surrogate keys for joins to avoid mismatches (e.g., Outlet code, ItemGroup code).
- Validate `f_depletions_final` against reported figures from commercial leads monthly.
