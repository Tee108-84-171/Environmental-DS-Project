# Environmental-DS-Project

# Data-Driven Design Basis for Municipal Solid Waste Management in Mueang Rayong

## 1. Environmental problem
Rayong is an industrial, coastal province. Municipal solid waste (MSW) grew from **1,425 to 1,598 t/day** between 2563 and 2567 B.E. (2.9 %/yr). The urban cluster of Mueang Rayong (6 municipalities) generates **254 t/day**, the whole district (15 local administrative organisations, LAOs) **490 t/day**. Before designing a system we need to know *how much waste to design for, how uncertain that is, and whether the proposed system (S3) is robustly better than landfilling (S1).*

**Research questions:** (1) How reliable is the data? (2) Where is the waste generated? (3) What drove provincial growth and where did it go? (4) What design flows follow from composition, and how uncertain are they? (5) How robust is S3 over S1?

## 2. Dataset source
| File (`data/raw/`) | Content |
|---|---|
| `rayong_msw_raw_data.xlsx` | Monthly 2568 waste for 15 LAOs + population; province series 2563–2568; disposal projects and S1/S3 design rows; source list |
| `rayong_msw_summary.pdf` | Summary narrative, LAO table and design composition table |
| `composition_from_pdf.csv`, `design_alternatives_from_xlsx.csv` | Hand-transcribed machine-readable copies of the PDF composition table and the S1/S3 text (validated in the notebook) |

Primary sources cited in the workbook: PCD Region 13 *Eastern Environmental Situation Report 2567*; Rayong PAO integrated disposal centre; PCD-13 Map Ta Phut transfer-station report; Rayong provincial waste committee minutes; Taphong municipality price document; Kasetsart Univ. × Rayong PAO composition study. **These primary documents were not re-downloaded** — please verify the headline figures (1,598.11 t/d; 983.66 t/d) against the PCD-13 report before submission.

## 3. Data cleaning and transformation
Column standardisation (Thai→English), removal of spreadsheet total rows, **independent re-computation of all formulas**, long-format monthly table, LAO metadata (zone, cluster membership, *tonnage basis*), and **13 validation checks against the PDF and internal mass balances — 13/13 pass** (`data/processed/audit_checks.csv`). Outputs are in `data/processed/`.

## 4. Exploratory data analysis
* Per-capita generation is **1.30 kg/cap/day for all 15 LAOs**, and the monthly series equals *daily rate × days in month* (CV of the daily rate = 2e-05): **there is no real seasonality, and population is not independent of tonnage** (`fig01`).
* **26 % of district tonnage (33 % in the urban cluster) is simply population × 1.30**; the rest is reported/contract data, often as ranges (`fig02`).
* Two LAOs (Map Ta Phut, Rayong City) generate **54 %** of district waste; the six-municipality cluster is **52 %** (`fig03`).
* Province 2563–2567: recovery rose from 31 % to 38 %, backlog fell from 12,500 t to 0 (`fig04`).

## 5. Data analysis and insights extraction
1. **Data audit (Insight 1).** Numbers are internally consistent but largely modelled — regression of waste on population would be circular, and seasonal effects cannot be learnt from this data.
2. **Growth went to recovery (Insight 2).** Of the +173 t/d added 2563→2567, **94 %** was recovered (+163 t/d) while properly-disposed tonnage grew only +20 t/d (`fig05`).
3. **Population base matters (Insight 3).** Per-capita is 2.07 kg/cap/d on registered residents but 1.30 with floating population — and the floating population reproduces tonnage ÷ 1.30 to within 1 person in 2567–68, strongly suggesting it is back-calculated, not independently measured.
4. **Design flows (`fig06`).** Cluster MRF feed = **309.03 t/working-day** (254 × 365/300 — every line is sized 21.7 % above the calendar-day average); district = 596.17.
5. **S3 mass balance (Insight 4, `fig07`).** Landfill drops from 254 to 51.82 t/d (**−80 %**), but outlets sum to only 70.2 %: **29.8 % of mass is unassigned** (likely moisture/biodegradation loss — to confirm in the design calculations).
6. **Budgets (Insight 5, `fig08`).** 4 of 6 listed budgets equal tonnes × 500 baht (the tipping fee) — derived, not independent cost evidence.

## 6. Data visualization
13 figures in `figures/` (one message per figure; titles state the finding; teal = recovery/S3, red = landfill/S1, grey = derived data). `fig09_key_insights_panel.png` summarises the four findings that matter most for the design.

## 7. Basic machine learning / simulation
Supervised prediction from population was rejected as circular. Methods used instead:
* **K-means (k=3, silhouette 0.70)** segments the 15 LAOs into logistics tiers: A = 2 LAOs (115–150 t/d, 54 % of waste), B = 5 (18–45 t/d), C = 8 (<11 t/d) (`fig10`).
* **Trend regression (n=5) + 3 scenarios** for the 20-year / 2-phase horizon (`fig11`): linear trend +43.6 t/d/yr (leave-one-out RMSE 2.2 t/d; predicts 1643 vs plan 1614 for 2568). Because the series looks modelled, no confidence interval is claimed; growth scenarios come from the data instead: low 1.0 %/yr (plan), base linear (2.7 %/yr of the 2567 level), high 2.9 %/yr (historical CAGR). Urban-cluster MRF feed: **341–412 t/working-day in year 10** and **376–548 in year 20**.
* **Monte-Carlo on composition (20,000 runs, `fig12`)** using the PDF's reported ranges (food 46.21–55.33 %, plastic 20.45–32.21 %; triangular, mode = design value): food P95 = 165 vs design 148 t/wd (×1.11); plastic P95 = 93 vs 74 (×1.25). Food flow exceeds design in 80 % of runs, plastic in 70 %.
* **S1 vs S3 robustness (`fig13`)**: the notebook reproduces the reported unit costs (S1 +297.76, S3 -586.40 baht/t). Annualised cost is almost identical (S1 74.0 vs S3 74.0 M baht/y), so the decision hinges on S3's product revenue (82.02 M baht/y) and land use (92.6 vs 375 rai, −75 %). With a +30 % / +50 % S3 cost over-run, S3 still wins if product revenue stays above 27 % / 45 % of the assumed value. (Monte-Carlo cost ranges are *assumed* for illustration.)

## 8. Environmental interpretation
* **Landfill burden:** S3 cuts landfilled mass by ~80 % and land take by ~75 %; over 20 years the base scenario sends 2.38 Mt to landfill under S1 vs 0.49 Mt under S3. About 66 % of the stream is degradable organic material (food, wood/garden, paper, textile), the main source of landfill methane and leachate.
* **Not yet a climate claim:** net GHG benefit is not quantified — RDF combustion emits fossil CO₂ from plastics and composting has CH₄/N₂O emissions; this needs an IPCC first-order-decay / LCA calculation (natural next step).
* **Resilience:** sizing on point-estimate composition under-provides capacity in most simulated cases — add the P95 margin above.
* **Priority data to collect:** weighbridge tonnage per LAO per month (Nam Khok centre / transfer stations), measured composition by season (tourism at Ban Phe / Ko Samet), floating-population estimates.

## 9. Limitations
~26 % of district tonnage derived from population × 1.30; no real seasonality; 2568 values are plan figures; S3 mass balance leaves 30 % unassigned; S1/S3 CAPEX build-ups were not provided (costs are the group's undiscounted totals); regression n=5; Monte-Carlo cost ranges are assumptions; primary sources not re-verified.
