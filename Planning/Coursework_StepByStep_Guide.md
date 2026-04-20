# Visual Analytics Coursework — Complete Step-by-Step Guide
**Target: 100/100 — Spring 2026**
**Deadline: 13:00, Tuesday 5th May 2026**

---

## 0. Mark Allocation (Plan Your Effort Accordingly)

- **Problem understanding — 10 marks** → introduction + target-user framing.
- **Data preparation & task analysis — 15 marks** → cleaning, joining, Munzner tasks.
- **Data visualisation — 50 marks** → (32 dashboard) + (10 dimensionality reduction) + (8 justification).
- **Conclusions — 15 marks** → what you learned about the *problem* AND about *InfoVis*.
- **Presentation — 10 marks** → prose quality, figures, references.

> **Rule of thumb:** every section in your report must map to one of the five criteria above. If a paragraph doesn't earn marks, cut it.

---

## 1. Choose Your Socio-Economic Angle (Day 1)

You must pick ONE well-defined question. Two strong options — pick whichever interests you more. Both are defensible, both have rich 2011 + 2021 data, both support Bayesian modelling.

### Option A — Health, Ethnicity & Deprivation

- **Research question:** "How does self-reported general health vary across ethnic groups and local authorities in England & Wales, and how has this relationship shifted between 2011 and 2021?"
- **Why it scores well:** health inequality is a front-page policy issue; COVID makes 2011→2021 comparison compelling; ethnicity + health gives a rich multivariate space ideal for PCA/t-SNE.
- **Tables to pull (2011):** KS201EW (Ethnic group), LC3206EW (General health by ethnic group), QS303EW (Long-term health problem / disability), KS611EW (Occupation) as a socio-economic control.
- **Tables to pull (2021):** TS021 (Ethnic group) + TS037 (General health) for direct comparison.
- **Target users:** NHS regional commissioners, local-authority public-health analysts, Department of Health policy leads.

### Option B — Housing Affordability & Employment

- **Research question:** "Which local authorities in England & Wales show the biggest mismatch between housing tenure, occupation mix and employment, and how has that mismatch evolved 2011 → 2021?"
- **Why it scores well:** housing affordability is highly topical (2030 forecasting ties directly to the Bayesian requirement); the data is clean and complete.
- **Tables to pull (2011):** KS402EW (Tenure), KS601EW (Economic activity), KS611EW (Occupation), QS411EW (Hours worked).
- **Tables to pull (2021):** TS054 (Tenure) + TS066 (Economic activity).
- **Target users:** local-authority housing officers, Homes England, DLUHC analysts, mortgage lenders.

> **Pick one. Don't combine them.** The rubric rewards *depth* over breadth.

---

## 2. Set Up Your Working Environment (Day 1-2)

- Create a project folder with this structure:
  - `/data/raw/2011/` — original .xlsx/.csv from the 2011 index.
  - `/data/raw/2021/` — bulk downloads from nomisweb.
  - `/data/clean/` — tidy CSVs you will load into Tableau.
  - `/data/projections/` — CSV matrices from PCA / t-SNE / UMAP.
  - `/notebooks/` — Jupyter notebooks for EDA + modelling.
  - `/scripts/` — the single `projections.py` you will submit.
  - `/report/` — LaTeX or Word draft + figures.
  - `/tableau/` — `.twbx` packaged workbook.
- Install Python libraries: `pandas`, `numpy`, `scikit-learn`, `umap-learn`, `pymc` (Bayesian), `arviz`, `matplotlib`, `seaborn`.
- Version your work with git from Day 1 — examiners sometimes ask for provenance; you protect yourself from data loss.

---

## 3. Data Acquisition (Day 2-3)

### 2011 data

- Open `2011CensusIndexofTablesandTopics_v11_4_2.xlsx`, tab **"All Tables"**.
- For each table in your short-list, follow the hyperlink and download at **Local Authority District (LAD)** level — this is the unit of analysis that matches best to 2021.
- Save with descriptive filenames: `KS201EW_ethnic_2011_LAD.csv` etc.

### 2021 data — two routes, pick one

- **Route 1 — Bulk download (recommended):** https://www.nomisweb.co.uk/sources/census_2021_bulk → download the `.zip` for each TS-table you need → unzip → each folder contains a LAD-level CSV.
- **Route 2 — ONS map download:** https://www.ons.gov.uk/census/maps → select variable → "Download data". Slower but sometimes cleaner.

### Geography reconciliation (critical)

- 2011 uses pre-2019/2021 LAD boundaries; 2021 uses current boundaries (several mergers in Somerset, Dorset, North Yorkshire, Buckinghamshire etc.).
- Download the ONS **LAD lookup / changeover file** from https://www.ons.gov.uk/releases/censusmapsupdatechangeovertime.
- Two options to reconcile:
  - **Option A — roll 2021 back to 2011 boundaries:** aggregate merged LADs by summing counts; preserves the larger LAD set (331 units).
  - **Option B — roll 2011 forward to 2021 boundaries:** simpler (fewer rows), but you lose granularity.
- Pick **Option A** if you want more points in your projection (better clustering); pick **Option B** if you want simpler maps and narrative.

---

## 4. Data Preparation & Abstraction (Day 3-5)

This section is worth 15 marks — document everything in your report.

- **Load & inspect** each raw file in a Jupyter notebook. Check row counts, header rows, ONS footnotes.
- **Standardise column names** — use lowercase snake_case, e.g. `lad_code`, `lad_name`, `eth_white`, `health_very_good`.
- **Pivot to tidy form** if tables come as matrices — one row per LAD, one column per variable.
- **Compute proportions** as well as counts (the rubric explicitly rewards this — "Having the option of switching between absolute values and proportions is often a useful feature").
- **Join tables on `lad_code`** using pandas `merge`.
- **Handle missing values** — flag them, do NOT drop rows yet. Bayesian imputation happens next.
- **Record the data abstraction** using Munzner's vocabulary:
  - Dataset type: **tables** (one per census year) + **geometric/spatial** (LAD polygons).
  - Attribute types: **categorical** (ethnicity, tenure), **ordered / ordinal** (general-health 5-point scale), **quantitative** (counts, proportions).
  - Dataset semantics: one **item** per LAD; attributes are **distributions** over categories.

### Two Bayesian modelling options (you must use at least one)

- **Option A — Bayesian Linear Regression for missing-value imputation.**
  - Use `pymc` to regress the missing attribute on correlated predictors (e.g. impute missing health proportions from ethnicity + occupation mix).
  - Report the **posterior mean** as the imputed value and the **95% credible interval** as uncertainty.
  - Easier to justify; gives you credible intervals to put on your dashboard.
- **Option B — Bayesian forecasting to 2030.**
  - Fit a hierarchical model of your key variable across 2011 → 2021 and sample the posterior predictive at 2030.
  - Richer narrative (you can show a "2030 forecast" map in Tableau) but more work; requires a hierarchical prior over LADs.

> **Recommendation for 100/100:** implement **both** — use A to clean data, use B to add a forward-looking map. This is the single biggest differentiator between a 2:1 and a first.

---

## 5. Task Definition (Day 5) — Munzner Taxonomy

You must frame your tasks using Munzner's **Why–What–How** framework. List 3-5 tasks; each gets one paragraph in the report.

- **Action-Target pairs** to choose from:
  - *Discover → distribution* (e.g. how is poor health distributed geographically?).
  - *Compare → trends* (2011 vs 2021).
  - *Locate → outliers* (which LADs are unusual on the PCA projection?).
  - *Identify → clusters* (do t-SNE groupings align with geographic regions?).
  - *Derive → new attributes* (compute a composite deprivation index).

- For each task state:
  - **Why** the user performs it.
  - **What** data is consumed / produced.
  - **How** (which encoding) it is supported in your dashboard.

---

## 6. Dimensionality Reduction (Day 6) — Worth 10 Marks

You MUST submit at least two projections using different algorithms, both saved as CSV matrices, both visualised in Tableau.

### Two algorithm-pair options — pick one

- **Pair A — PCA + t-SNE (safest).**
  - PCA is **linear, interpretable, global** → use to show overall variance structure and report loadings for the justification.
  - t-SNE is **non-linear, local** → reveals clusters that PCA hides.
  - Together they give complementary global + local views — easy to justify in the report.

- **Pair B — PCA + UMAP (more modern).**
  - UMAP preserves more global structure than t-SNE and runs faster.
  - Gives you a modern talking point in the justification section.

### Implementation checklist (same for either pair)

- Stack your cleaned, **proportion-scaled** table of per-LAD attributes (e.g. one row per LAD, columns = all ethnicity/health/tenure proportions).
- Standardise with `StandardScaler` BEFORE projecting.
- Run the two algorithms; for each, output a CSV with columns: `lad_code, lad_name, dim1, dim2, original_label_for_colour_coding`.
- Save: `/data/projections/pca_2011_2021.csv` and `/data/projections/tsne_2011_2021.csv`.
- In the report explicitly **list the input variables** — the brief says *"it is important to communicate to the user which variables were used in the original data space as otherwise, it is hard to interpret the plots."*
- For PCA, also report the **first two principal-component loadings** — this lets the marker see you understand the projection.

---

## 7. Python Script to Submit (Day 6-7)

Consolidate all the above into one reproducible script `projections.py` (plus the `.ipynb` for your own exploration). Structure it as:

- Section 1: imports & config.
- Section 2: load cleaned CSVs.
- Section 3: Bayesian imputation (pymc) → writes imputed CSV + credible-interval CSV.
- Section 4: feature matrix assembly + standardisation.
- Section 5: PCA → saves `pca_*.csv` + prints loadings.
- Section 6: t-SNE or UMAP → saves `tsne_*.csv` (set `random_state=42` for reproducibility).
- Section 7 (optional): Bayesian 2030 forecast → saves `forecast_2030.csv`.
- Include a `README.md` with a one-line `python projections.py` reproduction instruction.

---

## 8. Build the Tableau Dashboard (Day 7-10) — Worth 32 Marks

### Two dashboard-architecture options

- **Option A — single multi-view dashboard.**
  - One screen, four linked views: (1) choropleth map, (2) PCA scatter, (3) t-SNE scatter, (4) small-multiples bar chart for 2011 vs 2021.
  - Pros: unified story; easier for the reviewer.
  - Cons: can feel cramped.

- **Option B — three-dashboard storybook.**
  - Dashboard 1: *Overview* — choropleth map + KPI tiles.
  - Dashboard 2: *Structure* — PCA + t-SNE side-by-side with brushing-and-linking.
  - Dashboard 3: *Change & Forecast* — 2011 vs 2021 slope chart + 2030 Bayesian forecast map with credible-interval toggle.
  - Pros: cleaner, supports narrative; the brief explicitly says more than one dashboard is allowed.
  - Cons: more work.

> **Recommendation for 100/100: Option B.** The brief says "Do not feel you have to squeeze everything onto a single dashboard"; examiners reward a clean story.

### Encoding choices to state & justify in the report

- **Choropleth** — sequential (ColorBrewer YlOrRd) for quantitative; diverging (RdBu) for change-from-mean. Justify with Munzner Chapter 10 (sequential vs diverging colour maps).
- **Scatter for projections** — position on x/y encodes the two projected dimensions (strongest channel per Mackinlay/Cleveland-McGill); colour encodes a meaningful label (e.g. region); size encodes population.
- **Bar charts for categorical comparisons** — length is the most accurate channel for quantitative comparison.
- **Slope charts / small multiples** for 2011→2021 change — exploits the parallel-coordinates principle.

### Interactivity requirements (must have all of these)

- **Filters:** LAD, region, year.
- **Toggle** between absolute counts and proportions (the brief explicitly rewards this).
- **Tooltip** on every mark showing LAD name + all displayed values + (if Option B Bayesian) the credible interval.
- **Brush/linked highlighting** between the map and the projection scatter so the user can see where a t-SNE cluster sits geographically — this is your strongest visual-analytics move.
- **Parameter control** to switch the PCA scatter colour between ethnicity, region, and the health index.

### Geocoding

- Build a **custom geocoding folder** mapping `lad_code` → centroid lat/long (CSV from ONS Open Geography Portal).
- If Tableau errors, consult https://kb.tableau.com/articles/issue/error-the-custom-geocoding-folder-has-errors-when-creating-map.

### Package correctly

- **File → Export Packaged Workbook** → produces `.twbx`.
- Open the `.twbx` on a **different machine** to verify it renders without missing data sources. *(The brief threatens a zero mark if it doesn't open.)*

---

## 9. Write the Report (Day 10-12) — 6 to 10 pages

Use this exact section order — matches the rubric 1-to-1.

- **Abstract** (~150 words) — problem, approach, headline finding.
- **Introduction** (~1 page) — define the socio-economic issue, cite 2-3 recent sources (ONS bulletins, Marmot Review, Resolution Foundation, Shelter etc.), state the research question, name the target users.
- **Data Preparation and Abstraction** (~1.5 pages) — list tables, describe cleaning & joining, reconcile 2011↔2021 geography, state Munzner dataset type + attribute semantics, justify your Bayesian imputation choice with posterior diagnostics (Rhat, effective sample size).
- **Task Definition** (~1 page) — 3-5 Why-What-How tasks.
- **Visualisation Justification** (~2 pages — this is where the 8 justification marks live) — defend every encoding choice citing Munzner, Mackinlay, Cleveland-McGill, Few. Justify PCA *and* the non-linear method. State the inputs to the projections.
- **Evaluation** (~1 page) — run a small user study with your **discussion group** (per brief): 3-5 participants, give them 3 analytic tasks using your dashboard, record time-on-task and success. Frame against Munzner Chapter 4's four nested levels (domain, abstraction, encoding, algorithm) — state which you validated and how.
- **Conclusion** (~1 page) — split into two subsections: **(a)** what you learned about the socio-economic issue; **(b)** what you learned about InfoVis. Both are mandatory per the brief.
- **References** — ~10 refs in IEEE or Harvard style.

### Two styling options

- **Option A — Word + Zotero** — fastest, fine for a first.
- **Option B — LaTeX (IEEEtran template)** — looks more polished; recommended if aiming 100/100.

---

## 10. Evaluation / User Study (Day 11)

Don't skip this — the brief explicitly requires "measurements and observations of the other students in your discussion group".

- Recruit 3-5 classmates from your Blackboard discussion group.
- Prepare a one-page briefing sheet + 3 analytic tasks, e.g.:
  - "Find the LAD with the highest proportion of very-poor-health residents in 2021."
  - "Name one cluster visible in the t-SNE that is NOT obvious on the map."
  - "Predict whether LAD X is likely to be above or below the national health mean in 2030 and justify."
- Record **time-to-completion** and **success/fail** for each task.
- Ask a short **post-test questionnaire** (5-point Likert: usefulness, clarity, trust in the projections).
- Report results as a small table + a paragraph tying observations to Munzner's four validation levels.

---

## 11. Final Quality-Control Checklist (Day 12)

Do not submit until every box is ticked.

- [ ] Tableau workbook opens on a fresh machine as `.twbx`.
- [ ] Both PCA and t-SNE (or UMAP) projection CSVs are in the submission.
- [ ] `projections.py` runs end-to-end on a clean environment; produces identical outputs to those used in the dashboard.
- [ ] Report is 6-10 pages, every rubric section present, every figure captioned, every claim cited.
- [ ] Bayesian method is described AND its output is visible on the dashboard (e.g. credible-interval tooltips).
- [ ] "Absolute vs proportion" toggle works.
- [ ] Brushing works between map and projection scatter.
- [ ] All sources for each projection are listed in the report.
- [ ] Geographic reconciliation between 2011 and 2021 is documented.
- [ ] Discussion-group evaluation is reported with numbers.
- [ ] All files zipped into a single file named `<studentID>_VA_Coursework.zip`.
- [ ] Uploaded to Blackboard **before** 13:00 on Tuesday 5th May — aim to upload Monday evening.

---

## 12. Suggested Day-by-Day Timeline (15 working days remain to the deadline from 20 Apr 2026)

- **Mon 20 – Tue 21 Apr:** pick topic (§1), set up env (§2), start downloading data.
- **Wed 22 – Fri 24 Apr:** finish data acquisition (§3), clean & reconcile geography (§4).
- **Sat 25 – Sun 26 Apr:** Bayesian imputation + optional 2030 forecast.
- **Mon 27 Apr:** task definitions (§5).
- **Tue 28 Apr:** run both projections, save CSVs (§6-7).
- **Wed 29 – Fri 1 May:** build Tableau dashboards (§8).
- **Fri 1 May (evening):** user study with discussion group (§10).
- **Sat 2 – Mon 4 May:** write the report (§9).
- **Mon 4 May (evening):** QC checklist (§11) + package zip + upload.
- **Tue 5 May morning:** buffer — fix any Blackboard upload glitches.

---

## 13. Common Pitfalls That Cost Marks

- Choosing a vague topic ("general trends in England") → loses problem-understanding marks.
- Not reconciling LAD boundaries → your 2011 and 2021 numbers look wrong → caught by marker.
- Submitting a `.twb` instead of `.twbx` → **zero marks** on the dashboard.
- Using default PCA / t-SNE settings with no justification → loses the 8 justification marks.
- Not listing the input variables to the projection → loses projection marks.
- Skipping the user study → loses evaluation marks.
- Bayesian method that isn't actually Bayesian (e.g. just a scikit-learn model) → fails a rubric requirement.
- Too many screenshots in the report → wastes page budget the brief explicitly warns against.

---

## 14. "100/100" Optional Extras

If you have time after the core is done, these lift a high-first into a perfect mark:

- **Animated slope chart** showing 2011 → 2021 → 2030 (with Bayesian CI band).
- **Posterior-predictive checks** plotted alongside the forecast to demonstrate model validity.
- **Accessibility pass** — colour-blind-safe palette, explicit keyboard focus order (cite WAI-ARIA / viz-accessibility literature).
- **Reproducibility statement** — conda env file + git hash + random seeds in the report appendix.
- **Second non-linear projection** (e.g. both t-SNE *and* UMAP) with a paragraph comparing them.

---

**Good luck, Priyanshu. Follow the timeline, tick every box in §11, and you'll be in the 90s. The extras in §14 are what push it to 100.**
