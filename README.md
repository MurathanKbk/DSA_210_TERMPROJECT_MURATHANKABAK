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
- Audio features extracted per track: `danceability`, `energy`, `valence`, `tempo`, `acousticness`, `instrumentalness`, `speechiness`

---

## Repository Structure

```
.
├── README.md
├── myData.ipynb              # Main analysis notebook
├── mxmh_survey_results.csv   # Primary survey dataset
├── dataset.csv               # Spotify audio features dataset
├── merged_data.csv           # Enriched merged dataset (output)
├── mh_distributions.png      # Mental health score distributions
├── genre_mh_scores.png       # Average MH scores by genre
├── correlation_heatmap.png   # Audio features vs MH scores heatmap
└── qq_plots.png              # Q-Q plots for normality assessment
```

---

## How to Reproduce

### Requirements

```bash
pip install pandas numpy matplotlib seaborn scipy
```

Or install from requirements:

```bash
pip install -r requirements.txt
```

### Steps

1. Clone the repository
2. Place `mxmh_survey_results.csv` and `dataset.csv` in the root directory
3. Open `myData.ipynb` in Jupyter Notebook or VS Code
4. Run all cells in order

---

## Analysis Pipeline

### Stage 1 — Data Collection & Enrichment

The survey's categorical `Fav genre` column (16 genres) was mapped to Spotify's genre taxonomy. Audio features were averaged per Spotify genre to create genre-level psychoacoustic profiles. These profiles were then merged back onto each survey respondent based on their favorite genre, transforming categorical genre data into a rich numerical matrix of 7 audio features.

**Genre mapping highlights:**
- `R&B` → `r-n-b`
- `K pop` → `k-pop`
- `Video game music` → `electronic` (closest available)
- `Lofi` → `chill` (closest available)

Final merged dataset: **736 rows × 41 columns**. Zero unmapped respondents.

---

### Stage 2 — Exploratory Data Analysis

#### Mental Health Score Distributions

Histograms of all four MH scores reveal non-normal distributions:

- **Anxiety**: Left-skewed, concentrated at scores 6–8
- **Depression**: Bimodal, peaks near 0–2 and 6–7
- **Insomnia**: Right-skewed, majority at score 0–2
- **OCD**: Heavily right-skewed, majority at score 0

#### Average Mental Health Scores by Genre

Bar charts comparing genre groups show visible differences in average scores. Notable observations:
- **Lofi** listeners report the highest average depression (~6.5)
- **Latin** listeners report the lowest average anxiety (~4.3)
- **Folk** and **Hip hop** listeners report relatively high anxiety scores

#### Correlation Heatmap (Audio Features vs MH Scores)

Spearman correlations between the 7 audio features and 4 MH scores are weak across the board (all |r| < 0.15). The strongest observed relationship is `valence` vs `Insomnia` (r = −0.12), suggesting that higher musical positivity is weakly associated with lower insomnia — consistent with intuition but not statistically conclusive at the genre-average level.

#### Missing Value & Outlier Analysis

**Missing values:** None found across all mental health scores and audio features. The merged dataset is complete with no imputation required.

**Outlier analysis:** The IQR method initially flagged 45–63 apparent outliers across Depression, Insomnia, and OCD. However, directional inspection confirmed 0 low and 0 high outliers in all three cases. This is expected behavior — because the scores are bounded integers (0–10) with wide natural spread, the IQR fences extend beyond the possible scoring range, making true outliers mathematically impossible. No data removal or cleaning was necessary.

---

### Stage 3 — Hypothesis Testing

#### Normality Assessment

Q-Q plots confirm that all four MH score distributions deviate substantially from normality. This justifies the use of **non-parametric tests** for all subsequent hypothesis testing, consistent with the parametric vs. non-parametric framework covered in the course.

---

#### Hypothesis Test 1 — Kruskal-Wallis Test

**Question:** Do anxiety scores differ significantly across favorite genre groups?

| | |
|---|---|
| **H₀** | Anxiety scores are equal across all favorite genre groups |
| **H₁** | At least one genre group has a different anxiety score distribution |
| **Test** | Kruskal-Wallis H-test (`scipy.stats.kruskal`) |
| **α** | 0.05 |

**Result:** Fail to reject H₀ (p > 0.05)

**Interpretation:** There is no statistically significant difference in anxiety scores across favorite genre groups. While bar charts show visible differences in group means, the within-group variance is large enough that these differences cannot be distinguished from random sampling variation.

---

#### Hypothesis Test 2 — Spearman Correlation Test

**Question:** Is there a significant monotonic relationship between audio features and mental health scores?

| | |
|---|---|
| **H₀** | No monotonic relationship exists between audio features and MH scores |
| **H₁** | A significant monotonic relationship exists |
| **Test** | Spearman rank correlation (`scipy.stats.spearmanr`) |
| **α** | 0.05 |

This test was run twice — first with genre-level average features, then with individual track-level features matched via each respondent's self-reported BPM (valid range 40–250 BPM; 621/736 rows matched individually, 115 filled with genre average fallback).

**Results — Genre-level average features:**

| Feature | Target | ρ | p-value | Decision |
|---|---|---|---|---|
| valence | Anxiety | 0.0420 | 0.2548 | Fail to reject H₀ |
| valence | Depression | 0.0202 | 0.5848 | Fail to reject H₀ |
| **valence** | **Insomnia** | **−0.1020** | **0.0056** | **Reject H₀ ✓** |
| energy | Anxiety | 0.0199 | 0.5902 | Fail to reject H₀ |
| **energy** | **Depression** | **0.0807** | **0.0286** | **Reject H₀ ✓** |
| energy | Insomnia | 0.0510 | 0.1666 | Fail to reject H₀ |
| danceability | Anxiety | 0.0187 | 0.6127 | Fail to reject H₀ |
| danceability | Depression | 0.0039 | 0.9152 | Fail to reject H₀ |
| **danceability** | **Insomnia** | **−0.0784** | **0.0335** | **Reject H₀ ✓** |
| acousticness | Anxiety | −0.0259 | 0.4826 | Fail to reject H₀ |
| **acousticness** | **Depression** | **−0.0869** | **0.0183** | **Reject H₀ ✓** |
| acousticness | Insomnia | −0.0543 | 0.1415 | Fail to reject H₀ |
| tempo | Anxiety | 0.0413 | 0.2637 | Fail to reject H₀ |
| tempo | Depression | 0.0594 | 0.1075 | Fail to reject H₀ |
| tempo | Insomnia | 0.0656 | 0.0753 | Fail to reject H₀ |

**Results — Individual track-level features (BPM-matched):**

| Feature | Target | ρ | p-value | Decision |
|---|---|---|---|---|
| **valence** | **Insomnia** | **−0.0794** | **0.0314** | **Reject H₀ ✓** |
| valence | Anxiety | 0.0390 | 0.2903 | Fail to reject H₀ |
| valence | Depression | −0.0429 | 0.2451 | Fail to reject H₀ |
| energy | Anxiety | 0.0184 | 0.6183 | Fail to reject H₀ |
| energy | Depression | 0.0309 | 0.4032 | Fail to reject H₀ |
| energy | Insomnia | −0.0149 | 0.6874 | Fail to reject H₀ |
| danceability | Anxiety | 0.0410 | 0.2664 | Fail to reject H₀ |
| danceability | Depression | −0.0031 | 0.9333 | Fail to reject H₀ |
| danceability | Insomnia | −0.0561 | 0.1281 | Fail to reject H₀ |
| acousticness | Anxiety | −0.0104 | 0.7788 | Fail to reject H₀ |
| acousticness | Depression | −0.0722 | 0.0502 | Fail to reject H₀ |
| acousticness | Insomnia | −0.0693 | 0.0602 | Fail to reject H₀ |
| tempo | Anxiety | 0.0559 | 0.1295 | Fail to reject H₀ |
| tempo | Depression | 0.0647 | 0.0795 | Fail to reject H₀ |
| tempo | Insomnia | 0.0708 | 0.0549 | Fail to reject H₀ |

**Interpretation:** The most robust and consistent finding across both approaches is **valence → Insomnia** (genre-level: ρ = −0.102, p = 0.006; individual-level: ρ = −0.079, p = 0.031). This relationship holds regardless of the feature assignment method, indicating that listeners whose music has higher positivity tend to report lower insomnia scores. At the genre-level, additional significant pairs emerged (energy → Depression, danceability → Insomnia, acousticness → Depression), but these did not survive the transition to individual track features, suggesting they were partially driven by genre-level aggregation rather than individual listening patterns. **Insomnia** consistently emerges as the mental health dimension most sensitive to musical audio properties.

---

#### Hypothesis Test 3 — Mann-Whitney U Test

**Question:** Do anxiety levels differ between people who report music improves their mood versus those who report it worsens their mood?

| | |
|---|---|
| **H₀** | Anxiety is equal between "Improve" and "Worsen" music effect groups |
| **H₁** | There is a difference in anxiety between the two groups |
| **Test** | Mann-Whitney U test (`scipy.stats.mannwhitneyu`, two-sided) |
| **α** | 0.05 |

**Result:** Fail to reject H₀ (p > 0.05)

**Interpretation:** No statistically significant difference in anxiety scores was found between respondents who report music improves their mood and those who report it worsens it. Note that the "Worsen" group is substantially smaller than the "Improve" group, which limits statistical power.

---

## Key Findings

1. **Genre groups do not significantly differ in anxiety scores.** Kruskal-Wallis test found no statistically significant difference across 16 genre groups, suggesting that genre preference alone is not a reliable predictor of anxiety.

2. **Valence → Insomnia is the most robust finding across both analysis approaches.** This negative relationship (higher musical positivity = lower insomnia) held at both genre-level (ρ = −0.102, p = 0.006) and individual track-level (ρ = −0.079, p = 0.031), making it the most reliable signal in the dataset.

3. **Genre-level aggregation inflates the number of significant correlations.** Four pairs were significant at genre-level but only one survived individual track-level analysis, revealing that some signals were artifacts of averaging rather than true individual-level relationships.

4. **Insomnia is the most acoustically sensitive MH dimension**, appearing as the only significant target at the individual track level, and the most frequently significant at genre level.

5. **Distributions are non-normal.** Visual inspection via Q-Q plots and histograms confirms that all MH scores are non-normally distributed, validating the non-parametric approach throughout.

6. **107 respondents had missing or invalid BPM values** (outside 40–250 range). These were filled with genre-level average features as a fallback, ensuring full dataset coverage for all analyses.

---

## Limitations

- **Genre-level averaging** flattens individual variation in listening habits. A respondent who listens to "Rock" may primarily listen to soft rock or heavy metal — very different psychoacoustic profiles collapsed into one average.
- **Self-reported data** is subject to recall bias and subjective interpretation of the 0–10 scale.
- **Cross-sectional design**: No causal claims can be made. Correlations (or lack thereof) do not imply that music causes or results from mental health states.
- **Small "Worsen" group**: The Mann-Whitney U test had limited statistical power due to the small number of respondents reporting that music worsens their mood.

---

## Next Steps (ML Phase — Due May 5)

1. **PCA** on the 7 audio feature vectors to reduce dimensionality and identify latent psychoacoustic dimensions
2. **K-Means clustering** to group respondents by sonic phenotype rather than genre label
3. **Post-hoc hypothesis testing** (Kruskal-Wallis / Mann-Whitney U) to determine whether the acoustically-defined clusters exhibit statistically significant differences in Anxiety, Depression, or Insomnia scores

---

## AI Tool Usage Disclosure

As required by the course academic integrity policy, AI assistance (Claude, Anthropic) was used in this project for: guidance on test selection (Kruskal-Wallis, Spearman, Mann-Whitney U), debugging merge logic, and README drafting. All analysis decisions, interpretations, and code were reviewed and understood by the student.

---

## References

- Music & Mental Health Survey Results dataset — available on GitHub / Kaggle
- Spotify Tracks Dataset — available on Kaggle
- Larson & Farber, *Elementary Statistics* (course reference)
- scipy.stats documentation — https://docs.scipy.org/doc/scipy/reference/stats.html