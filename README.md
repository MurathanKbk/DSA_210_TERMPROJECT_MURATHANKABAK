# Psychoacoustic Profiling: Clustering Latent Auditory Dimensions against Mental Health Metrics

**DSA 210 — Introduction to Data Science | Spring 2025–2026**  
**Student:** Murathan Kabak | ID: 34374

---

## Motivation

Music's psychological impact is widely studied, yet most research relies on subjective, broad genre labels. This project strips away those labels and analyzes how the underlying mathematical and psychoacoustic profiles of a user's preferred music correlate with self-reported mental health metrics. By combining survey data with quantifiable audio features, the project explores whether latent auditory dimensions align with specific psychological states.

---

## Data Sources

### Primary Dataset
**Music & Mental Health Survey Results** (`mxmh_survey_results.csv`)
- 736 respondents
- Self-reported listening frequencies across 16 music genres
- Continuous mental health scores (0–10) for: Anxiety, Depression, Insomnia, OCD
- Additional attributes: age, streaming service, hours per day, music effects

### Enrichment Dataset
**Spotify Audio Features** (`dataset.csv`)
- 114,000 tracks across 114 Spotify genre categories
- Audio features per track: `danceability`, `energy`, `valence`, `tempo`, `acousticness`, `instrumentalness`, `speechiness`

---

## Repository Structure
.
├── .gitignore
├── Project PROPOSAL MURATHAN KABAK.pdf
├── README.md
├── edaHypothesisAndML.ipynb         # Main analysis notebook (EDA + Hypothesis Tests + ML)
├── merged_data.csv                  # Enriched merged dataset (output)
├── mxmh_survey_results.csv          # Primary survey dataset
├── dataset.csv                      # Spotify audio features dataset
├── requirements.txt

---

## How to Reproduce

### Requirements

```bash
pip install -r requirements.txt
```

### Steps

1. Clone the repository
2. Place `mxmh_survey_results.csv` and `dataset.csv` in the root directory
3. Open `edaHypothesisAndML.ipynb` in Jupyter Notebook or VS Code
4. Run all cells in order

---

## Analysis Pipeline

### Stage 1 — Data Collection & Enrichment

The survey's categorical `Fav genre` column (16 genres) was mapped to Spotify's genre taxonomy. Audio features were averaged per Spotify genre to create genre-level psychoacoustic profiles. These profiles were merged back onto each survey respondent based on their favorite genre, transforming categorical genre data into a rich numerical matrix of 7 audio features.

**Genre mapping highlights:**
- `R&B` → `r-n-b`
- `K pop` → `k-pop`
- `Video game music` → `electronic` (closest available)
- `Lofi` → `chill` (closest available)

Final merged dataset: **736 rows × 41 columns**. Zero unmapped respondents.

---

### Stage 2 — Exploratory Data Analysis

#### Mental Health Score Distributions
- **Anxiety**: Left-skewed, concentrated at scores 6–8
- **Depression**: Bimodal, peaks near 0–2 and 6–7
- **Insomnia**: Right-skewed, majority at score 0–2
- **OCD**: Heavily right-skewed, majority at score 0

#### Average Mental Health Scores by Genre
- **Lofi** listeners report the highest average depression (~6.5)
- **Latin** listeners report the lowest average anxiety (~4.3)
- **Folk** and **Hip hop** listeners report relatively high anxiety scores

#### Correlation Heatmap
Spearman correlations between the 7 audio features and 4 MH scores are weak across the board (all |r| < 0.15). The strongest observed relationship is `valence` vs `Insomnia` (r = −0.12).

#### Missing Value & Outlier Analysis
- **Missing values:** None found across all mental health scores and audio features
- **Outliers:** IQR method flagged 45–63 apparent outliers across Depression, Insomnia, and OCD, but directional inspection confirmed 0 true outliers due to the bounded integer scale (0–10)

---

### Stage 3 — Hypothesis Testing

#### Normality Assessment
Q-Q plots confirm that all four MH score distributions deviate substantially from normality, justifying the use of non-parametric tests throughout.

#### Test 1 — Kruskal-Wallis Test
**Question:** Do anxiety scores differ significantly across favorite genre groups?  
**Result:** Fail to reject H₀ (p = 0.283). No statistically significant difference across genre groups.

#### Test 2 — Spearman Correlation Test
**Question:** Is there a significant monotonic relationship between audio features and mental health scores?  
**Result:** The most robust finding is **valence → Insomnia** (ρ = −0.079, p = 0.031 at individual level), holding across both genre-level and individual track-level analyses.

#### Test 3 — Mann-Whitney U Test
**Question:** Do anxiety levels differ between people who report music improves vs. worsens their mood?  
**Result:** Fail to reject H₀ (p = 0.295). No statistically significant difference found.

---

### Stage 4 — Machine Learning

#### Feature Engineering
- **Classification target:** `high_anxiety` — Anxiety ≥ 6 → 1 (high risk), Anxiety < 6 → 0 (low risk)
- **Regression target:** Raw `Anxiety` score (0–10, continuous)
- **Features:** 7 Spotify audio features + Age + Hours per day + BPM + genre (One-Hot Encoded)
- **Total features:** 25 (classification), 23 (regression)
- **Preprocessing:** StandardScaler, 80/20 train-test split with stratification

#### Classification Results

| Model | Accuracy | Precision | Recall | F1 | ROC AUC |
|---|---|---|---|---|---|
| Decision Tree | 0.512 | 0.636 | 0.461 | 0.534 | 0.552 |
| KNN (k=15) | 0.584 | 0.620 | 0.816 | 0.705 | 0.554 |
| Random Forest | 0.584 | 0.628 | 0.776 | 0.694 | 0.591 |

All three models performed near random (ROC AUC ≈ 0.55–0.59). Random Forest feature importance revealed **Age** as the single most important predictor, outweighing all audio features combined.

#### Regression Results

| Model | MSE | RMSE | R² |
|---|---|---|---|
| Linear Regression | 7.715 | 2.778 | 0.059 |
| Random Forest Regressor | 7.552 | 2.748 | 0.079 |

Both models explain less than 8% of variance in anxiety scores (R² < 0.08).

#### Clustering Results

K-Means clustering (k=3) grouped respondents by sonic profile:

| Cluster | Size | Profile | Avg Anxiety |
|---|---|---|---|
| 0 | 177 | High energy, high tempo (~160 BPM), low acousticness | 5.89 |
| 1 | 315 | High valence, high danceability, moderate tempo | 5.88 |
| 2 | 129 | Low energy, high acousticness, high instrumentalness | 5.74 |

Despite clear differences in sonic profiles, all three clusters show nearly identical mental health scores.

---

## Key Findings

1. **Genre groups do not significantly differ in anxiety scores** (Kruskal-Wallis, p = 0.283)
2. **Valence → Insomnia is the most robust finding** (ρ = −0.079, p = 0.031 at individual level)
3. **Insomnia is the most acoustically sensitive mental health dimension**
4. **All ML models performed near chance level** (ROC AUC ≈ 0.55–0.59)
5. **Age is the strongest predictor** of anxiety, outweighing all music-related features
6. **Clustering by sonic profile does not stratify mental health outcomes**

---

## Limitations

- Genre-level averaging flattens individual variation in listening habits
- Self-reported data is subject to recall bias
- Cross-sectional design: no causal claims can be made
- Small "Worsen" group limits statistical power in Mann-Whitney U test
- Class imbalance (60.3% high anxiety) required `class_weight='balanced'`

---

## AI Tool Usage Disclosure

As required by the course academic integrity policy, AI assistance (Claude, Anthropic) was used for: guidance on test selection, ML model selection and implementation, debugging, and README drafting. All analysis decisions, interpretations, and code were reviewed and understood by the student.

---

## References

- Music & Mental Health Survey Results dataset — Kaggle
- Spotify Tracks Dataset — Kaggle
- scipy.stats documentation — https://docs.scipy.org/doc/scipy/reference/stats.html
- scikit-learn documentation — https://scikit-learn.org/stable/