# Grant County Sheriff — Law Enforcement Incident Analysis

An end-to-end R analysis of federal law enforcement incident data for Grant County, WA,
producing clearance rate predictions, predictive crime tables, and patrol resource intelligence
using real ICPSR data — the same data infrastructure used by IC and law enforcement analysts.


| JD Requirement | This Project |
|---|---|
| Law enforcement analytics | Federal ICPSR crime incident data, LE domain framing |
| Large volume data analysis | Multi-year incident dataset, 10+ feature dimensions |
| Actionable leads & workflows | Predictive crime table by location + shift |
| ML: Classification | Logistic regression + Random Forest on clearance outcomes |
| R proficiency | Full tidyverse workflow, ggplot2, broom, caret |
| Communicating to non-technical audiences | Forest plot, stacked bar chart, clearance rate rankings |

## What This Project Does

Analyzes real federal crime incident data (ICPSR Study 39270) filtered to the
Grant County Sheriff's Office (`ORI == "WA0130000"`) to answer three operational questions:

1. **What predicts whether a case gets cleared (arrest made)?**
2. **Which crime types have the highest and lowest clearance rates?**
3. **What crimes are most likely to occur by location and shift?**

## Project Structure

```
federal_analysis/
├── federal.Rmd          # Main R Markdown analysis notebook
├── crime_by_time.png    # Exported crime distribution visualization
└── README.md
```

## Analysis Pipeline

### 1. Feature Engineering
Raw ICPSR variables are transformed into operationally meaningful features:

| Engineered Feature | Description |
|---|---|
| `is_cleared` | Target variable — was an arrest made? (Cleared / Not Cleared) |
| `SHIFT` | Day (0600–1359) / Swing (1400–2159) / Graveyard (2200–0559) |
| `OFFENSE_GROUP` | Single vs. Multiple offenses per incident |
| `AGE_GROUP` | Juvenile / Adult / Senior victim classification |
| `LOCATION_GROUP` | Residential / Commercial / Public Space / Other |
| `CRIME_CATEGORY` | Full offense type label from NIBRS coding |

### 2. Clearance Rate Analysis
Clearance rates broken down by offense group and crime type — identifies which
incident categories are most and least likely to result in arrest.

### 3. Logistic Regression Model
Predicts case clearance likelihood from shift, offense group, and victim age.
Results visualized as a **forest plot** showing odds ratios with confidence intervals —
the standard presentation format for law enforcement and public health risk modeling.

```r
logit_model <- glm(target ~ OFFENSE_GROUP + SHIFT + AGE_GROUP,
                   data = final_analysis_data,
                   family = binomial)
```

### 4. Random Forest Classifier
Ensemble classification model adding location group as a predictor.
Feature importance plot identifies the strongest operational predictors of arrest outcomes.

```r
rf_model <- randomForest(
  as.factor(target) ~ OFFENSE_GROUP + SHIFT + AGE_GROUP + LOCATION_GROUP,
  data = final_analysis_data,
  ntree = 500,
  importance = TRUE
)
varImpPlot(rf_model, main = "Feature Importance: Case Clearance Predictors")
```

### 5. Predictive Crime Table
Maps the most probable crime type for each combination of location and shift —
directly analogous to the patrol resource allocation tables used in active law enforcement operations.

### 6. Time-of-Day Crime Distribution
Stacked bar chart showing crime type proportions across four time windows,
enabling shift commanders to anticipate incident mix before deployment.

## Key Findings

- **Clearance rates vary significantly by crime type** — some offense categories
  are resolved at 2–3x the rate of others, indicating resource allocation opportunities
- **Shift and victim age group** are statistically significant predictors of case clearance
- **Location group** improves Random Forest classification — commercial and public
  space incidents show distinct clearance patterns vs. residential
- **Graveyard shift** incidents show different crime composition than day/swing shifts,
  relevant for staffing and response planning

## Quickstart

```r
# Install dependencies
install.packages(c("tidyr", "dplyr", "ggplot2", "GGally",
                   "broom", "haven", "forcats", "randomForest", "caret"))

# Load ICPSR dataset (download from https://www.icpsr.umich.edu/web/ICPSR/studies/39270)
load("path/to/39270-0003-Data.rda")

# Run the full analysis
rmarkdown::render("federal.Rmd")
```

## Data Source

**ICPSR Study 39270** — National Incident-Based Reporting System (NIBRS)
provided by the Inter-university Consortium for Political and Social Research.
Filtered to Grant County Sheriff's Office, Moses Lake, WA.

> Data accessed via ICPSR public repository. Usage complies with ICPSR
> terms of use for academic and research purposes.

## Technologies

R · tidyverse · ggplot2 · GGally · broom · haven · forcats · randomForest · caret ·
Logistic Regression · Random Forest · R Markdown · NIBRS / ICPSR Federal Data
