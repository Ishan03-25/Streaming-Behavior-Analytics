# Spotify Streaming Analytics

End-to-end product analytics on 149K+ listening events: from raw CSV to skip-prediction model and interactive dashboard.

**Tech stack:** Python, Pandas, scikit-learn, Streamlit, Plotly, SQL (SQLite / PostgreSQL)

## Table of Contents

- [Overview](#overview)
- [Highlights](#highlights)
- [Business Questions Answered](#business-questions-answered)
- [Key Findings](#key-findings)
- [Interactive Dashboard](#interactive-dashboard)
- [Quick Start](#quick-start)
- [Data](#data)
- [Methodology](#methodology)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Limitations and Future Work](#limitations-and-future-work)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Overview

Every skipped song is a signal. This project treats a personal Spotify streaming history as a product analytics case study: cleaning the raw event log, engineering behavioral features, and answering the questions a Product or Business Analyst would be asked.

- When do users listen?
- Which platforms drive engagement?
- Why do users skip?
- Can we predict a skip before it happens?

The pipeline runs with a single command and produces a cleaned dataset, a feature-engineered dataset, trained ML models, and a multi-page Streamlit dashboard.

## Highlights

| Area | Detail |
|------|--------|
| Scale | ~149K event-level streaming records |
| Feature engineering | 82 features across temporal, behavioral, and contextual categories |
| Machine learning | Skip prediction (3 models compared) and user segmentation (clustering) |
| SQL analysis | CTEs, window functions, and sessionization queries |
| Dashboard | 5-page interactive Streamlit app with Plotly charts |
| Pipeline | `python run_project.py` runs everything end to end |

## Business Questions Answered

| # | Question | Where to look |
|---|----------|---------------|
| 1 | When do users listen to music most frequently? | Dashboard: Listening Patterns |
| 2 | Which platforms have the highest engagement? | Dashboard: Overview, Skip Analysis |
| 3 | What factors drive users to skip songs? | Dashboard: Skip Analysis, ML Models |
| 4 | How can we improve user retention? | Session and retention analysis (notebooks) |
| 5 | What are the different user behavioral segments? | User clustering (notebooks, `model_pipeline.py`) |

## Key Findings

- **Mobile-first behavior:** Android accounts for about 93.9% of all plays. Desktop usage is minimal.
- **Skip behavior is habitual:** the strongest predictor of a skip is the user's recent skip history, not the track's own attributes.
- **Context matters:** session context and platform show distinct engagement patterns.
- **Best model:** Gradient Boosting reached an AUC of 0.970 on skip prediction.

### Model Comparison

| Model | AUC | Accuracy |
|-------|:---:|:--------:|
| Logistic Regression | 0.954 | 87.2% |
| Random Forest | 0.964 | 88.3% |
| **Gradient Boosting** | **0.970** | **89.7%** |

### Top Feature Importance (Gradient Boosting)

| Feature | Importance |
|---------|:----------:|
| Rolling 5-track skip rate | 0.806 |
| User skip rate | 0.098 |
| Session skip rate | 0.036 |
| Previous track skipped | 0.019 |
| Track position | 0.017 |
| Session track count | 0.005 |

> **Note on interpretation.** One feature (rolling skip rate) carries about 80% of the importance. Because it is derived from past skip labels, the high AUC reflects skip momentum more than content understanding. See [Limitations and Future Work](#limitations-and-future-work).

## Interactive Dashboard

A Streamlit app with a Spotify-inspired dark theme and five sections:

| Page | What it shows |
|------|---------------|
| Overview | Total plays, listening hours, unique tracks and artists, skip rate, date range |
| Listening Patterns | Daily activity trend, platform split, hourly listening curve |
| Content Analysis | Top 10 tracks and artists by play count and hours |
| Skip Analysis | Skip rate by platform and by hour of day |
| ML Models | Model comparison, key findings, feature importance |

## Quick Start

### Prerequisites

- Python 3.9+
- The dataset file `spotify_history.csv` (see [Data](#data))

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/spotify-analytics.git
cd spotify-analytics

# 2. (Recommended) create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt
```

### Run the full pipeline

Place `spotify_history.csv` in the `data/` folder, then run:

```bash
python run_project.py
```

The runner validates your environment, then executes three stages in order:

| Step | Script | Output |
|:----:|--------|--------|
| 1 | `scripts/data_preprocessing.py` | `data/spotify_cleaned.csv` |
| 2 | `scripts/feature_engineering.py` | `data/spotify_features.csv` |
| 3 | `scripts/model_pipeline.py` | `models/` (trained models) |

To run a single stage:

```bash
python run_project.py preprocessing   # clean the raw data
python run_project.py features        # engineer features
python run_project.py models          # train the ML models
python run_project.py help            # show usage
```

### Launch the dashboard

```bash
streamlit run dashboard.py
```

The dashboard reads `spotify_history.csv`, `spotify_cleaned.csv`, and `spotify_features.csv` from `data/`, so run the pipeline first.

## Data

**Source:** Spotify streaming history export, ~150K event-level records.

| Column | Description |
|--------|-------------|
| `spotify_track_uri` | Unique track identifier |
| `ts` | Timestamp of the streaming event |
| `platform` | Device platform (Android, iOS, web player) |
| `ms_played` | Milliseconds the track was played |
| `track_name` | Track title |
| `artist_name` | Artist name |
| `album_name` | Album name |
| `reason_start` | Why the track started (for example, click or auto-advance) |
| `reason_end` | Why the track ended (for example, track done or skip) |
| `shuffle` | Whether shuffle mode was on |
| `skipped` | Whether the track was skipped |

## Methodology

**1. Data preprocessing and cleaning.** Standardizes timestamps, handles missing values, and normalizes formats to produce `spotify_cleaned.csv`.

**2. Feature engineering (82 features).** Builds temporal features (hour, day, weekday), behavioral features (rolling and session-level skip rates, track position), and contextual features (platform, shuffle, start and end reason) into `spotify_features.csv`.

**3. Skip prediction.** Trains and compares Logistic Regression, Random Forest, and Gradient Boosting classifiers, evaluated on AUC and accuracy.

**4. User segmentation.** Clusters listening behavior to find distinct user segments. The optimal solution identified 2 clusters.

**5. SQL analysis.** Top tracks and artists by playtime, skip rate by platform and time, and average session length, using CTEs, window functions, and sessionization logic.

## Project Structure

```
spotify_analytics/
├── run_project.py              # One-command pipeline runner
├── dashboard.py                # Streamlit dashboard
├── requirements.txt            # Python dependencies
├── data/
│   ├── spotify_history.csv     # Raw input (you provide)
│   ├── spotify_cleaned.csv     # Generated: cleaned data
│   └── spotify_features.csv    # Generated: engineered features
├── scripts/
│   ├── data_preprocessing.py   # Step 1: clean and standardize
│   ├── feature_engineering.py  # Step 2: build features
│   └── model_pipeline.py       # Step 3: train models
├── notebooks/
│   ├── 01_eda.ipynb                    # Exploratory data analysis
│   ├── 02_business_insights.ipynb      # Business intelligence
│   └── 03_advanced_analytics.ipynb     # ML and clustering
├── sql/
│   ├── basic_queries.sql       # Top tracks, skip rates
│   ├── advanced_queries.sql    # CTEs, window functions
│   └── sessionization.sql      # Session-level analysis
├── models/                     # Trained model artifacts
├── reports/                    # Analysis outputs
└── logs/
```

## Tech Stack

| Area | Tools |
|------|-------|
| Data processing | Pandas, NumPy |
| Visualization | Matplotlib, Seaborn, Plotly |
| Machine learning | scikit-learn |
| Dashboard | Streamlit |
| Querying | SQL (SQLite / PostgreSQL) |
| Environment | Jupyter |

## Limitations and Future Work

**Known limitations**

- **Possible target leakage.** Rolling and session skip-rate features are computed from prior skip outcomes and dominate model importance (about 80%). To validate real predictive power, compute these features strictly from earlier events only and evaluate with a time-based train/test split rather than a random one.
- **Single-user data.** The dataset appears to represent one listener, so segmentation and platform findings may not generalize to a full user base.
- **Platform imbalance.** With about 94% of plays on Android, cross-platform comparisons rest on small samples.

**Future work**

- [ ] Re-run skip prediction with a leakage-safe, time-based validation split
- [ ] Add a content-only baseline (no skip-history features) for comparison
- [ ] Deploy the dashboard to Streamlit Community Cloud
- [ ] Add retention and churn cohort analysis
- [ ] Serve the best model behind a simple prediction API

## Contributing

Issues and pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License

Distributed under the MIT License. See `LICENSE` for details.

## Acknowledgments

- Dataset: Spotify streaming history export
- Built with Streamlit, Plotly, and scikit-learn
- README structure informed by Make a README and Awesome README
