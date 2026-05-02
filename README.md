# SP26_IDS701_UDS_Project

## Research question

**Among players who could plausibly leave after freshman year, what is the association between staying two years in college (`num_years_college == 2`) versus one-and-done (`num_years_college == 1`) on early NBA production, measured by Player Factor in NBA seasons 1–5 (`nba_season_1`–`nba_season_5`)?**

The primary analysis for this question is in **`matching.ipynb`**.

**Estimand:** ATT (average treatment effect on the **treated**) — for **two-year** players, contrast observed outcomes with those of **matched one-and-done** peers who had similar **freshman** Player Factor (`college_1`).

## Target audience / use

Results are framed for basketball operations stakeholders (e.g., draft strategy): the focus is whether an **additional sophomore year**, holding **freshman impact** comparable, shows up in **early-career NBA** metrics on the same composite scale used elsewhere in the project.

## Methodology (`matching.ipynb`)

1. **Sample:** Rows from `data/ncaa_nba_data.csv` with exactly **1 or 2** years of college and non-missing **`college_1`** (~402 players: ~220 one-and-done, ~182 two-year).
2. **Overlap (central trimming):** Restrict to the **central 90%** of `college_1` (5th–95th percentiles, band approximately **3.22–16.13**) so both treatment groups appear across freshman performance levels (**360** players in the analytic window).
3. **Treatment:** `treat == 1` ⇔ two years; `treat == 0` ⇔ one-and-done.
4. **Pre-treatment covariates for propensity:** **`college_1` only**, expanded as **linear + quadratic** (`PolynomialFeatures(degree=2)`). The notebook **does not** match on draft pick (`Pk`), draft year, or **`college_2`**, on the grounds that those are **downstream** of the stay-vs-leave decision.
5. **Propensity score:** `P(T=1 | X)` estimated by **logistic regression** on polynomial features of `college_1`; scores used for diagnostics and matching.
6. **Matching:** **1:1 nearest neighbor** on **logit(propensity score)** with a **Rubin caliper** of **0.25 × SD(logit PS)** (~**0.168**), **without** reusing controls. **76** of **166** treated (two-year) players in the overlap sample receive an acceptable match.
7. **Balance:** Standardized mean differences **before → after match**: `college_1` roughly **0.46 → 0.04**; propensity score **0.57 → ~0**.
8. **Outcome effects:** ATT = mean paired difference (two-year minus matched one-and-done) for each **`nba_season_k`** and for the **mean across NBA seasons 1–5**. **Uncertainty:** **Bootstrap** resampling of matched pairs (**2000** replicates); **95% intervals** reported.
9. **Supplementary estimand:** **Hájek** inverse-probability-weighted contrast on the **full** overlap-trimmed sample (closer to an **ATE**-style weighted comparison; differs from matched ATT).

## Main conclusions (`matching.ipynb`)

- After **propensity-based 1:1 matching on freshman Player Factor**, **none** of the ATT **bootstrap 95% CIs** exclude **zero** for individual NBA seasons 1–5 or for their **average** (point estimates oscillate by season; intervals are **wide**, especially in later years where **missing** seasons reduce effective information).
- **Interpretation:** In this matched subsample there is **no statistically distinguishable** difference in early NBA Player Factor between **two-year** players and **comparable one-and-dones**. That should be read as **“no clear evidence in this design and sample”**, **not** as proof of a causal null.
- **Caveats:** A large share of **two-year** athletes **cannot** be matched within the caliper; identification assumes **`college_1`** plus the propensity functional form absorbs the main confounders of **returning vs declaring** (**injury**, **advisory feedback**, academics, etc. are **not** in the model); **NBA** outcomes still bundle **minutes**, role, and selection into the measured impact.

Sensitivity to overlap definition and caliper can be explored by toggling **`USE_OVERLAP_RESTRICTION`** and **`caliper_mult`** in `matching.ipynb`.

## Other artifacts

- **`nba_did.ipynb`** explores panel / difference-in-differences style formulations on reshaped longitudinal data — a **distinct** causal framing from the **`matching.ipynb`** ATT exercise above.
- Pipeline and preprocessing for wider NCAA/NBA merges appear in **`ncaa_nba_data.Rmd`**, **`figures.Rmd`**, and **`UDS_Final_Project_NBA.Rmd`**.
- **`DARKO.ipynb`**: retrieve DARKO data into `data/` (optional / separate data source).

---

#### Clone

```bash
git clone https://github.com/tina-ytyuan/SP26_IDS701_UDS_Project.git
```

#### Create virtual environment + install dependencies

```bash
python -m venv venv
source venv/bin/activate

pip install -r requirements.txt

```

### Retrieve DARKO data

To retrieve the DARKO data and store in the data folder (on local), run `DARKO.ipynb`.
