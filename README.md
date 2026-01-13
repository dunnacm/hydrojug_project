# HydroJug Phase 2 — Demand vs Inventory (5-Month Forecasting Horizon)

This repository contains my solution for a supply planning/data analysis exercise. The goal is to compare **combined demand** (forecast + open orders, without double counting) against **available supply** (on-hand + scheduled incoming) across the next **5 months**, and to identify where we can fulfill demand vs where we must either **order** or **move demand**.

## View the report (recommended)
- **HTML report (GitHub Pages):** https://dunnacm.github.io/hydrojug_project/

The report is exported from the notebook and is intended to be readable by both technical and non-technical audiences.

## What’s in this repo
- `analysis.ipynb` — the full notebook: methodology + code + outputs
- `docs/index.html` — the self-contained exported HTML report (used by GitHub Pages)
- `.gitignore` — excludes raw data files from being published

## Problem framing (high level)
I answer three questions over the next 5 months:
1. **What can be shipped** with current and incoming inventory?
2. **Which UPCs require ordering now**, considering a standard **90-day lead time**?
3. **Where might demand need to move** to later months due to timing/backlog risk?

## Key definitions (given)
- **Combined demand** = Forecast + Open Orders, but when orders overlap forecast, we count the **order** (no double counting).
- **Available inventory** is inventory that can be committed immediately.
- **Committed** inventory is already reflected in **On Hand (OH)**.
- **Pending Approval** and **Unallocated** do not tap inventory until manually changed.
- **Backordered** will auto-commit once inventory becomes available.
- **Scope:** only analyze UPCs that have inventory data.
- **Lead time:** assume **90 days (~3 months)**.

## Method summary (how it works)
1. **Normalize UPCs** across tables for reliable joins (Excel formatting differences are common).
2. **Build time-phased forecast** by `(UPC, Channel)` across the next 5 months.
3. **Aggregate orders** (open + committed). Since orders are not month-dated in the provided data, I use a documented assumption:
   - **Forecast-shaped allocation:** distribute orders across months proportional to the forecast pattern for that `(UPC, Channel)`.
4. **Compute combined demand** using the “greater-of” rule:
   - `CombinedDemand_m = max(Forecast_m, AllocatedOrders_m)`
5. **Build supply**:
   - start with `On Hand`
   - bucket `Incoming` by `Next Expected Date` into monthly receipts
6. **Simulate inventory position** month-by-month per UPC:
   - `Position_m = StartOnHand + cumulative_receipts_m - cumulative_demand_m`
7. **Separate true shortages vs timing risks**:
   - **End-of-horizon shortage** (still negative at the end)
   - **Peak backlog** (negative at any point, even if it recovers)
8. **Lead time**:
   - estimate order quantities only for months where a PO could arrive given ~90 days.

## Data confidentiality note
Because I could not confirm whether the provided dataset is proprietary, I intentionally did **not** publish the raw Excel file to this repository. The analysis is still fully reviewable via the HTML report and notebook outputs.

## How to reproduce locally (optional)
> If you have access to the Excel file, place it in `data/` and run the notebook.

### Setup (Windows / PowerShell)
```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install --upgrade pip
py -m pip install pandas openpyxl matplotlib jupyter ipykernel nbconvert
