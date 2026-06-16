---
name: variance-analysis
description: Produce a structured budget-vs-actual variance analysis
---

# Variance Analysis

Produce a structured variance analysis comparing budget to
actual figures.

## Required inputs

Ask the user for:
- **Period** (e.g., Q2 2026, June 2026)
- **Line items** or categories to analyze
- **Budget figures** and **actual figures**
- **Materiality threshold** for flagging (default: 5% or $10K)

## Analysis structure

### 1. Summary table

| Line Item | Budget | Actual | Var ($) | Var (%) | Flag |
|-----------|--------|--------|---------|---------|------|

Flag items exceeding the materiality threshold with a marker.

### 2. Favorable / Unfavorable classification

Group variances by:
- **Favorable**: actuals better than budget
- **Unfavorable**: actuals worse than budget

### 3. Root cause commentary

For each flagged variance, provide:
- Likely cause (volume, price, timing, one-time)
- Whether the variance is expected to recur
- Suggested action (investigate, adjust forecast, none)

### 4. Net impact

Summarize the total favorable and unfavorable impact on the
bottom line for the period.
