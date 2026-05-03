# Does Staying in College Pay Off? Estimating the Causal Effect of a Second Year on Early NBA Production
 
**IDS 701: Unifying Data Science | Spring 2026 | Duke MIDS**
**Team:** Bruce Chen, Gaurav Law, Arushi Singh, Tina Yuan

---

## Project Overview
 
Every spring, college basketball freshmen who are on the bubble for the NBA Draft face a high-stakes choice: declare for the draft now, or return to college for a second year. The presumed upside of staying is an additional season of development that pays off in the league. The downside is forgone NBA salary and possible erosion of draft stock. What players and their coaches actually want to know is whether the development benefit is real, holding the player's freshman ability fixed.
 
This project uses propensity-score matching on a panel of 393 NBA-drafted players from 2007-08 through 2024-25 to estimate the causal effect of staying for a second college year on early NBA production. We pair each two-year player with a one-and-done peer who looked statistically similar as a freshman, then compare their NBA performance over the first five professional seasons.
 
The final output is an Average Treatment Effect on the Treated (ATT) for each NBA season and for the five-season average, with bootstrap confidence intervals showing the precision of each estimate.
 
This project is designed for stakeholders such as:
 
- Division I men's basketball head coaches advising draft-eligible freshmen
- Player development directors at college programs
- Players themselves who are weighing the stay-or-leave decision
- Sports analytics researchers studying college-to-pro transitions

---

## Research question

**Among players who could plausibly leave after freshman year, what is the association between staying two years in college (`num_years_college == 2`) versus one-and-done (`num_years_college == 1`) on early NBA production, measured by Player Factor in NBA seasons 1–5 (`nba_season_1`–`nba_season_5`)?**

More specifically, we ask:
 
1. After matching on freshman performance, is there a statistically distinguishable difference in early NBA Player Factor between two-year players and similar one-and-dones?
2. Does the matched comparison support a development-investment narrative for the extra college year, or does it leave the question open?
3. How much of the matched-pair variance is explained by precision (sample size and missingness) versus genuine effect heterogeneity?
4. Does the answer depend on where in the freshman performance distribution the player sits?
5. Does the matching design hold up under reasonable changes to the overlap window and caliper?
The framing is causal at the individual-player level (would another year of college push this player's NBA output up?), not predictive at the team-evaluation level (given a prospect's number of college years, how good are they likely to be?). The two questions look similar but lead to different answers and different stakeholders.
 
---
 
## Data Sources
 
This project draws on four publicly available basketball data sources:
 
| Source | Description |
|---|---|
| Bart Torvik | College basketball season-level advanced metrics (PORPAG, offensive and defensive rating, usage percent) |
| DARKO | Daily Adjusted and Regressed Kalman Optimized projections for NBA players, using box scores and tracking data |
| Basketball Reference | NBA and NCAA player season tables; used for college-year reconciliation |
| NBA.com (via `hoopR` in R) | NBA player and draft data |
 
Data was scraped and merged at the player level. Fuzzy joins between college and NBA datasets were occasionally inaccurate, so name and college matching was reconciled manually for ambiguous cases.
 
### Accessing the Data
 
The merged analytic file is available in this repository at `data/ncaa_nba_data.csv`. Larger raw files (DARKO history, Torvik exports, draft tables) are also in `data/`.
 
To regenerate the DARKO portion from scratch:
 
```bash
jupyter notebook DARKO.ipynb
```
 
---
 
## Treatment, Outcome, and Estimand
 
**Treatment** is binary at the player level:
 
```text
treat = 1  if num_years_college == 2  (two-year players)
treat = 0  if num_years_college == 1  (one-and-done players)
```
 
**Outcome** is each player's DARKO Daily Plus-Minus (DPM) for NBA seasons 1 through 5, plus the five-season average. DPM is a regularized impact metric that uses box scores, player tracking, and game-level stats to project long-term value. Negative DPM indicates a poor career trajectory.
 
**Estimand** is the Average Treatment Effect on the Treated (ATT). For each two-year player, we contrast observed NBA outcomes against those of a matched one-and-done peer with similar freshman performance. The ATT answers the question: "for two-year players who have a comparable one-and-done peer on freshman performance, what is the average difference in early NBA production?"
 
A supplementary Hájek inverse-probability-weighted (IPW) estimator is reported on the full overlap-trimmed sample, closer to an ATE-style weighted comparison.
 
---
 
## Player Metric: College Impact Score
 
Comparing NCAA and NBA performance requires choosing measures that align across leagues. We use DARKO DPM for NBA outcomes and a composite **impact score** for college:
 
```text
impact = standardized( PORPAG, ORtg, DRtg, USG% ) / minutes_per_game
```
 
The impact score factors in Bart Torvik's Points Over Replacement Per Adjusted Game (PORPAG, for NCAA men), offensive and defensive ratings, and usage percent, all standardized by minutes per game. The metric is scaled to a [0, 25] range, with 25 representing the best college season-level performance.
 
### Imputation for Unobserved NBA Seasons
 
Players with no recorded data for a given NBA season are assigned a DPM of -10 (more conservative than the all-time single-season floor of approximately -5.9). This affects the descriptive averages but does not drive the matched-pair identification, since within-pair differences depend on each pair's actual values.
 
---
 
## Repository Structure
 
```text
SP26_IDS701_UDS_Project/
├── data/                                # Raw and processed data files
│   ├── ncaa_nba_data.csv                # Final merged analytic file
│   ├── darko_history.csv                # DARKO DPM history
│   ├── Torvik_26.csv                    # Bart Torvik college metrics
│   ├── draft.csv                        # NBA draft history (post-2008)
│   └── draft_pre_08.csv                 # NBA draft history (pre-2008)
├── matching.ipynb                       # Primary analysis: propensity matching + ATT
├── nba_did.ipynb                        # Exploratory: panel / staggered DiD framing
├── ncaa_nba_data.Rmd                    # NCAA/NBA merge pipeline
├── DARKO.ipynb                          # DARKO data retrieval
├── plots/                               # Output figures used in the memo
│   ├── Propensity_Scores.png
│   ├── Matched_ATT.png
│   └── DiD_plot.png
├── unused/                              # Earlier explorations not used in the final memo
├── requirements.txt                     # Python package requirements
├── renv.lock                            # R package lockfile
├── README.md                            # Project documentation
└── SP26_IDS701_UDS_Project.Rproj        # RStudio project file
```
 
---
 
## Environment Setup
 
Clone the repository:
 
```bash
git clone https://github.com/tina-ytyuan/SP26_IDS701_UDS_Project.git
cd SP26_IDS701_UDS_Project
```
 
Create and activate a Python virtual environment:
 
```bash
python -m venv venv
source venv/bin/activate
```
 
For Windows PowerShell:
 
```bash
python -m venv venv
venv\Scripts\Activate.ps1
```
 
Install Python dependencies:
 
```bash
pip install --upgrade pip
pip install -r requirements.txt
```
 
R dependencies (for the data merge and R-side scripts) are managed via `renv`. From R or RStudio:
 
```r
install.packages("renv")
renv::restore()
```
 
If Jupyter is not available after installation, run:
 
```bash
pip install notebook ipykernel
python -m ipykernel install --user --name nba-causal --display-name "Python (nba-causal)"
```
 
---
 
## How to Run the Pipeline
 
The project is organized around two notebooks: `matching.ipynb` (primary analysis) and `nba_did.ipynb` (exploratory robustness check).
 
### Step 1: Verify the Data
 
The merged file `data/ncaa_nba_data.csv` should already be in the repository. If you want to regenerate the DARKO portion or the full pipeline:
 
```bash
jupyter notebook DARKO.ipynb         # retrieve DARKO history
# then re-run ncaa_nba_data.Rmd in R/RStudio to merge
```
 
### Step 2: Launch Jupyter
 
```bash
jupyter notebook
```
 
or:
 
```bash
jupyter lab
```
 
### Step 3: Open the Primary Analysis Notebook
 
```text
matching.ipynb
```
 
### Step 4: Run the Notebook from Top to Bottom
 
The expected pipeline is:
 
1. Load the merged analytic file (`ncaa_nba_data.csv`)
2. Filter to players with one or two years of college and non-missing freshman performance
3. Apply central trimming (5th-95th percentile of freshman impact)
4. Estimate the propensity score via logistic regression on a quadratic in `college_1`
5. Diagnose overlap and balance before matching
6. Run 1:1 nearest-neighbor matching on logit propensity with a Rubin caliper
7. Verify post-matching balance
8. Compute matched ATT for each NBA season and the five-season average
9. Bootstrap-resample matched pairs (2,000 replicates) for 95% confidence intervals
10. Compute the supplementary Hájek IPW estimator for comparison
11. Run sensitivity toggles (`USE_OVERLAP_RESTRICTION`, `caliper_mult`)
12. Generate final figures: propensity overlap, matched ATT plot

### Step 5: Review Outputs
 
The main outputs are:
 
- Propensity-score overlap diagnostics (left: histogram, right: propensity vs. freshman impact)
- Pre/post-matching balance table
- Matched ATT estimates per NBA season with bootstrap 95% confidence intervals
- Five-season average ATT with confidence interval
- Hájek IPW comparison estimate
- Sensitivity analysis under alternative caliper and trimming choices
Final figures used in the memo are saved in `plots/`.
 
---
 
## Methodology
 
### Sample Construction
 
We start from a panel of 393 NBA-drafted players who competed in NCAA Division I men's basketball between 2007-08 and 2024-25 with one or two years of college and non-missing freshman performance. Players drafted directly out of high school (pre-2005-rule eligibility), international players drafted from foreign leagues, and players from the G League Ignite program are excluded because those pathways fall outside the college stay-or-leave decision context. Of the 393 players, 214 are one-and-done and 179 stayed for a second year.
 
### Identification Strategy
 
The core identification challenge is that better players tend to leave sooner. A naive comparison of NBA outcomes between one-year and two-year players therefore confounds the effect of staying with the effect of being talented enough to leave.
 
We address this by matching on freshman performance only. The intuition is to compare each two-year player to a one-and-done player who looked statistically similar at the end of their freshman season, then track both into the NBA.
 
Critically, we do **not** match on draft pick (`Pk`), draft year, or sophomore performance (`college_2`). All three are determined after the stay-or-leave decision, so including them as controls would absorb part of the very effect we are trying to measure.
 
### Propensity Score Model
 
We estimate the propensity score via logistic regression of treatment status on a degree-2 polynomial in `college_1` (linear plus quadratic, no intercept term in the design matrix). The quadratic allows the relationship between freshman performance and the probability of returning to be curved rather than monotone, which matters because the players most likely to return are those at moderate freshman performance, not the highest or lowest.
 
### Matching
 
We match 1:1 on logit(propensity score) using nearest-neighbor matching with a Rubin caliper of 0.25 × SD(logit PS) (approximately 0.168). Controls are not reused across matches. Of 179 two-year players in the analytic window, 93 receive an acceptable match within the caliper.
 
### Balance Diagnostics
 
Standardized mean differences before and after matching:
 
| Variable | Before matching | After matching |
|---|---:|---:|
| `college_1` (freshman impact) | 0.46 | 0.04 |
| Propensity score | 0.57 | ~0 |
 
The matched sample is well-balanced on the variable we balanced on.
 
### Estimation
 
The matched ATT is computed as the mean within-pair difference in DPM (two-year minus matched one-and-done) for each NBA season and for the five-season average. Confidence intervals come from a 2,000-replicate bootstrap resampling matched pairs.
 
---
 
## Key Results
 
After propensity-based 1:1 matching on freshman performance, **none** of the ATT bootstrap 95% confidence intervals exclude zero, for individual NBA seasons 1-5 or for the five-season average. Point estimates oscillate by season, with slight positive values in NBA years 2 and 5 and slight negative values in years 3 and 4. The five-season average ATT is close to zero.
 
| NBA Season | Point Estimate (ATT) | 95% Bootstrap CI |
|---|---:|---:|
| 1 | Slightly negative | Crosses zero |
| 2 | Slightly positive | Crosses zero |
| 3 | Slightly negative | Crosses zero (wider) |
| 4 | Slightly negative | Crosses zero (wider) |
| 5 | Slightly positive | Crosses zero (widest) |
| Average (1-5) | Near zero | Crosses zero |
 
(Exact numerical values are produced by `matching.ipynb` and rendered in `plots/Matched_ATT.png`.)
 
Key takeaways:
 
- The matched comparison does not reveal a statistically distinguishable benefit or harm of staying for a second year on early NBA production.
- The intervals are wide enough that modest effects in either direction remain consistent with the data. This is a precision result, not a precise null.
- Confidence intervals widen in later NBA seasons because more players drop out of the league, reducing the effective sample.
- The Hájek IPW supplementary estimator yields qualitatively similar (non-significant) results.
- Sensitivity to the overlap window and the caliper choice does not change the qualitative pattern: across reasonable settings, point estimates remain near zero and intervals do not exclude zero.
---
 
## Scope of the Finding
 
Roughly half of two-year players in the analytic window cannot be matched within the caliper. The headline result speaks to the matched 93 pairs, who cluster in the middle of the freshman performance distribution.
 
| Freshman performance region | Coverage in matched sample |
|---|---|
| Top-five-pick caliber | Sparse one-and-done comparisons available; matched sample thin |
| Middle of distribution | Strongest coverage and most reliable inference |
| Below draft picture | Two-year players dominate; one-and-done comparisons rare |
 
For freshmen at the extremes, individualized scouting input is more informative than this analysis. The result should not be applied as a universal rule across all freshmen.
 
---
 
## Identification Caveats
 
This is observational data, not a randomized experiment. Identification rests on the assumption that freshman performance, expanded with a quadratic functional form, captures the main confounders of returning versus declaring for the draft. Several plausible determinants of the stay-or-leave choice are not in the data:
 
- Injury history during the freshman year
- Advisory feedback from the NBA's pre-draft process
- Academic standing and eligibility considerations
- Family financial pressure
- Individual player confidence and personal preference
If any of these correlates with NBA outcomes through channels other than freshman performance, the matched ATT will not recover the true causal effect. The results should be read as evidence about the matched comparison group, not as a definitive causal null.
 
NBA outcomes also bundle minutes, role, and selection into the measured impact. A player who is drafted late and rarely plays will have a low DPM regardless of his underlying skill, which compresses the dynamic range of the outcome.
 
---
 
## Limitations
 
This project has several important limitations:
 
1. **Small effective sample.** Only 93 matched pairs drive the headline estimate. With observed Player Factor variances, this sample size produces 95% intervals that span roughly two units in either direction, so a true effect smaller than half a unit would not reliably be detected.
2. **No measurement of off-court determinants.** Injury, advisory feedback, academics, and family circumstances are unmeasured and may confound the stay-or-leave decision.
3. **DPM imputation for unobserved NBA seasons.** Players who never reach a given NBA season are assigned a conservative DPM of -10. The matched-pair identification is robust to this choice, but descriptive averages are sensitive.
4. **Outcome metric bundles minutes and role.** A player's DPM reflects both his skill and his team's choices about how to use him.
5. **Annual data is coarse.** The model cannot distinguish in-season development from cumulative seasonal effects.
6. **Excluded pathways.** Players who reached the NBA via international leagues, the G League Ignite, or directly from high school are outside the analysis. The conclusions do not transfer to those routes.
7. **Generalization to future cohorts.** Player development and NBA team usage have shifted over the 2007-2025 window. Results are average effects over that window and may not extrapolate cleanly to the modern game.

---
 
## Recommended Use
 
This analysis should be used as one input into individualized advising conversations, not as a standalone recommendation system.
 
Recommended use cases:
 
- Calibrating messaging to draft-bubble freshmen by avoiding overclaim in either direction
- Anchoring development conversations in a comparison group that is genuinely similar
- Identifying players whose freshman profile sits in the matched comparison region versus the extremes
- Supporting program-level discussions about how much weight to place on the development narrative

Not recommended as-is for:
 
- Universal rules about whether freshmen should stay or leave
- High-confidence advice for players at the extremes of freshman performance
- Inferences about subgroups (position, conference, body type) that the model does not condition on
- Comparisons between players who took non-college pathways

---
 
## Future Work
 
Future improvements could include:
 
- Adding richer pre-treatment covariates (high school recruiting rank, freshman minutes, position, physical measurements) to tighten the matching and increase precision
- Expanding to three- and four-year college players, with attention to the reduced overlap that staggered timing creates
- Replacing the DPM imputation with a survival-style model for NBA participation
- Conducting subgroup analysis by position and program strength
- Incorporating modern advanced NBA metrics (EPM, LEBRON, RAPTOR) as alternative outcomes
- Building a dashboard for coaches that visualizes a given freshman's location in the matched sample
- Exploring instrumental-variable designs that exploit policy or timing shocks to the stay-or-leave decision

---
 
## References

- NBA.com, Official NBA Statistics and Advanced Analytics (https://www.nba.com/stats)
- Bart Torvik, College Basketball Analytics (https://barttorvik.com)
- DARKO Daily Plus-Minus Player Projections (https://apanalytics.shinyapps.io/DARKO)
- Basketball Reference (https://www.basketball-reference.com)
- Rosenbaum and Rubin (1983), *The Central Role of the Propensity Score in Observational Studies for Causal Effects*
- Stuart (2010), *Matching Methods for Causal Inference: A Review and a Look Forward*
- Imbens and Rubin (2015), *Causal Inference for Statistics, Social, and Biomedical Sciences*
- Abadie and Imbens (2016), *Matching on the Estimated Propensity Score*

---
 
## License
 
This repository was created for academic use as part of Duke University's IDS 701: Unifying Data Science course.
