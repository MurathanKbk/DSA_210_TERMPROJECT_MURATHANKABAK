
Psychoacoustic Profiling: Clustering Latent Auditory Dimensions against Mental Health Metrics
DSA 210 — Introduction to Data Science | Spring 2025–2026
Student: Murathan Kabak | ID: 34374

Motivation
Music's psychological impact is widely studied, yet most research relies on subjective, broad genre labels. This project strips away those labels and analyzes how the underlying mathematical and psychoacoustic profiles of a user's preferred music correlate with self-reported mental health metrics. By combining survey data with quantifiable audio features, the project explores whether latent auditory dimensions align with specific psychological states.

Data Sources
Primary Dataset
Music & Mental Health Survey Results (mxmh_survey_results.csv)

736 respondents
Self-reported listening frequencies across 16 music genres
Continuous mental health scores (0–10) for: Anxiety, Depression, Insomnia, OCD
Additional attributes: age, streaming service, hours per day, music effects

Enrichment Dataset
Spotify Audio Features (dataset.csv)

114,000 tracks across 114 Spotify genre categories
Audio features extracted per track: danceability, energy, valence, tempo, acousticness, instrumentalness, speechiness


Repository Structure
.
├── .gitignore
├── Project PROPOSAL MURATHAN KABAK.pdf
├── README.md
├── edaHypothesisAndML.ipynb    # Main analysis notebook (EDA + Hypothesis Tests + Machine Learning Methods)
├── edaAndhyphothesisTest.ipynb    # Main analysis notebook (EDA + Hypothesis Tests)
├── merged_data.csv                # Enriched merged dataset (output)
├── mxmh_survey_results.csv        # Primary survey dataset
├── dataset.csv                    # Spotify audio features dataset

How to Reproduce
Requirements
bashpip install pandas numpy matplotlib seaborn scipy scikit-learn
Or install from requirements:
bashpip install -r requirements.txt
Steps

Clone the repository
Place mxmh_survey_results.csv and dataset.csv in the root directory
Open edaAndhyphothesisTest.ipynb in Jupyter Notebook or VS Code
Run all cells in order


Analysis Pipeline
Stage 1 — Data Collection & Enrichment
The survey's categorical Fav genre column (16 genres) was mapped to Spotify's genre taxonomy. Audio features were averaged per Spotify genre to create genre-level psychoacoustic profiles. These profiles were then merged back onto each survey respondent based on their favorite genre, transforming categorical genre data into a rich numerical matrix of 7 audio features.
Genre mapping highlights:

R&B → r-n-b
K pop → k-pop
Video game music → electronic (closest available)
Lofi → chill (closest available)

Final merged dataset: 736 rows × 41 columns. Zero unmapped respondents.

Stage 2 — Exploratory Data Analysis
Mental Health Score Distributions
Histograms of all four MH scores reveal non-normal distributions:

Anxiety: Left-skewed, concentrated at scores 6–8
Depression: Bimodal, peaks near 0–2 and 6–7
Insomnia: Right-skewed, majority at score 0–2
OCD: Heavily right-skewed, majority at score 0

Average Mental Health Scores by Genre
Bar charts comparing genre groups show visible differences in average scores. Notable observations:

Lofi listeners report the highest average depression (~6.5)
Latin listeners report the lowest average anxiety (~4.3)
Folk and Hip hop listeners report relatively high anxiety scores

Correlation Heatmap (Audio Features vs MH Scores)
Spearman correlations between the 7 audio features and 4 MH scores are weak across the board (all |r| < 0.15). The strongest observed relationship is valence vs Insomnia (r = −0.12), suggesting that higher musical positivity is weakly associated with lower insomnia.
Missing Value & Outlier Analysis
Missing values: None found across all mental health scores and audio features.
Outlier analysis: The IQR method initially flagged 45–63 apparent outliers across Depression, Insomnia, and OCD. However, directional inspection confirmed 0 true outliers in all three cases, as the bounded integer scale (0–10) makes IQR fences extend beyond the possible scoring range.

Stage 3 — Hypothesis Testing
Normality Assessment
Q-Q plots confirm that all four MH score distributions deviate substantially from normality, justifying the use of non-parametric tests throughout.
Hypothesis Test 1 — Kruskal-Wallis Test
Question: Do anxiety scores differ significantly across favorite genre groups?
Result: Fail to reject H₀ (p = 0.283). No statistically significant difference in anxiety scores across genre groups.
Hypothesis Test 2 — Spearman Correlation Test
Question: Is there a significant monotonic relationship between audio features and mental health scores?
Result: The most robust finding is valence → Insomnia (ρ = −0.079, p = 0.031 at individual level), holding across both genre-level and individual track-level analyses. Insomnia is the most acoustically sensitive mental health dimension.
Hypothesis Test 3 — Mann-Whitney U Test
Question: Do anxiety levels differ between people who report music improves vs. worsens their mood?
Result: Fail to reject H₀ (p = 0.295). No statistically significant difference found.

Stage 4 — Machine Learning
Feature Engineering

Target (Classification): high_anxiety — binary label where Anxiety ≥ 6 = 1 (high risk), Anxiety < 6 = 0 (low risk)
Target (Regression): Raw Anxiety score (0–10, continuous)
Features: 7 Spotify audio features + Age + Hours per day + BPM + genre (One-Hot Encoded, 15 dummy variables)
Total features: 25 (classification), 23 (regression)
Preprocessing: StandardScaler applied; train/test split 80/20 with stratification

Classification — Predicting High Anxiety
Three classifiers were trained and evaluated using 5-fold cross-validation with GridSearchCV for hyperparameter tuning:
ModelAccuracyPrecisionRecallF1ROC AUCDecision Tree0.5120.6360.4610.5340.552KNN (k=15)0.5840.6200.8160.7050.554Random Forest0.5840.6280.7760.6940.591
Key finding: All three models performed near random (ROC AUC ≈ 0.55–0.59), indicating that music preferences and audio features have limited predictive power for anxiety classification. Random Forest feature importance revealed Age as the single most important predictor, outweighing all audio features combined. Genre dummy variables contributed almost no predictive value.
Regression — Predicting Anxiety Score
Two regression models were trained to predict the continuous anxiety score:
ModelMSERMSER²Linear Regression7.7152.7780.059Random Forest Regressor7.5522.7480.079
Key finding: Both models explain less than 8% of the variance in anxiety scores (R² < 0.08), confirming that audio features alone are insufficient predictors of mental health outcomes.
Clustering — Listener Profiling
K-Means clustering (k=3, selected via Elbow Method and Silhouette Score) grouped respondents by their audio feature profiles:
ClusterSizeProfileAvg Anxiety0177High energy, high tempo (~160 BPM), low acousticness5.891315High valence, high danceability, moderate tempo5.882129Low energy, high acousticness, high instrumentalness5.74
Key finding: Despite clear differences in sonic profiles, all three clusters show nearly identical average anxiety, depression, and insomnia scores. This confirms that listening style does not stratify meaningfully by mental health status.

Key Findings

Genre groups do not significantly differ in anxiety scores (Kruskal-Wallis, p = 0.283).
Valence → Insomnia is the most robust finding (ρ = −0.079, p = 0.031 at individual level).
Insomnia is the most acoustically sensitive mental health dimension.
All ML models performed near chance level (ROC AUC ≈ 0.55–0.59), indicating that audio features have limited predictive power for anxiety.
Age is the strongest predictor of anxiety, outweighing all music-related features in Random Forest importance scores.
Clustering by sonic profile does not stratify mental health outcomes — listeners with very different musical tastes report similar anxiety, depression, and insomnia scores.


Limitations

Genre-level averaging flattens individual variation in listening habits.
Self-reported data is subject to recall bias and subjective interpretation of the 0–10 scale.
Cross-sectional design: No causal claims can be made.
Small "Worsen" group: Limited statistical power in Mann-Whitney U test.
Class imbalance: 60.3% of respondents fall in the high anxiety class, requiring class_weight='balanced' in classification models.


AI Tool Usage Disclosure
As required by the course academic integrity policy, AI assistance (Claude, Anthropic) was used in this project for: guidance on test selection, ML model selection and implementation, debugging, and README drafting. All analysis decisions, interpretations, and code were reviewed and understood by the student.

References

Music & Mental Health Survey Results dataset — available on GitHub / Kaggle
Spotify Tracks Dataset — available on Kaggle
scipy.stats documentation — https://docs.scipy.org/doc/scipy/reference/stats.html
scikit-learn documentation — https://scikit-learn.org/stable/