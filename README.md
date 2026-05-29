# Modeling California Daily Maximum Temperature

## Separating Seasonality and Multi-Decade Trends with Meteorological Predictors

## Overview

This project models California daily maximum temperature using meteorological, seasonal, long-term trend, and fire-related predictors. The analysis focuses on separating regular seasonal temperature cycles from multi-decade trends and short-term weather effects.

California has experienced increasingly frequent and severe wildfires over the last several decades. Hotter and drier conditions can increase wildfire risk by drying vegetation and soils. This project investigates how daily maximum temperature can be explained using meteorological conditions such as minimum temperature, precipitation, wind speed, lagged weather variables, seasonality, and fire-start indicators.

The project was completed in R using an applied statistical modeling workflow, including data cleaning, feature engineering, exploratory data analysis, transformations, multiple linear regression, model selection, diagnostic checking, and leave-one-out cross-validation.

## Research Question

How can future temperature trends in California be predicted using meteorological conditions, while accounting for seasonal changes and long-term patterns over the last few decades?

More specifically, this project asks:

- Which meteorological predictors are most strongly associated with daily maximum temperature?
- How can seasonality be modeled without treating day-of-year as a simple linear variable?
- Do precipitation, wind speed, and minimum temperature affect maximum temperature differently across seasons?
- Does the long-term year trend remain important after controlling for short-term weather conditions?

## Data

The project uses the public dataset:

**California Weather and Fire Prediction Dataset (1984–2025) with Engineered Features**  
Yavas et al. (2025), published on Zenodo.

The dataset aggregates daily meteorological observations from official sources, including NOAA Climate Data Online, and aligns them with wildfire incident records from CAL FIRE. It provides daily weather variables and engineered features suitable for statistical modeling.

Main variables used in this project include:

- `MAX_TEMP`: daily maximum temperature;
- `MIN_TEMP`: daily minimum temperature;
- `PRECIPITATION`: daily precipitation;
- `AVG_WIND_SPEED`: average wind speed;
- `LAGGED_PRECIPITATION`: lagged precipitation;
- `LAGGED_AVG_WIND_SPEED`: lagged average wind speed;
- `DAY_OF_YEAR`: day of year;
- `YEAR`: calendar year;
- `FIRE_START_DAY`: indicator for whether a fire started on that day.

The multi-decade coverage allows the analysis to estimate long-term temperature trends while separating them from short-term weather effects. The daily resolution also makes the dataset suitable for modeling strong seasonal patterns.

## Tools

This project was completed using R and R Markdown.

Main tools and packages include:

- R
- R Markdown
- `ggplot2`
- `dplyr`
- `tidyr`
- `broom`
- `car`
- `MASS`
- `patchwork`
- `gridExtra`
- `pander`
- `stargazer`
- `skimr`

## Methodology

## 1. Data Processing

The data was first cleaned by removing records with missing values. The fire-start-day variable was coded as a categorical factor, and the year variable was transformed into an adjusted long-term trend variable:

```r
YEAR_adj = YEAR - min(YEAR)
```

This allows the model to interpret year as the number of years since the beginning of the dataset period.

Lagged weather variables were also included to capture short-term delayed effects. For example, precipitation and wind speed from previous periods may influence near-future daily maximum temperature.

## 2. Seasonality Feature Engineering

Daily maximum temperature is strongly seasonal, so DAY_OF_YEAR should not be treated as a simple linear numeric variable. In a linear model, day 365 and day 1 would appear far apart, even though they are only one day apart in the annual cycle.

To capture seasonality more appropriately, DAY_OF_YEAR was converted into sine and cosine terms:

```r
Day_Sin = sin(2 * pi * DAY_OF_YEAR / 365)
Day_Cos = cos(2 * pi * DAY_OF_YEAR / 365)
```

These features allow the model to represent the annual temperature cycle, including summer peaks and winter troughs, in a smooth periodic form.

Residual plots against day of year showed that the smoothed residual pattern stayed close to zero after including the sine and cosine terms, suggesting that most of the seasonal structure was captured by these variables.

## 3. Transformations

Initial diagnostic plots showed slight violations of normality and constant variance assumptions. To improve model assumptions, the response variable was transformed using a Box-Cox inspired inverse-power transformation:

```r
MAX_TEMP_inv = 1 / (MAX_TEMP ^ 1.2)
```

Several predictors were also transformed to reduce non-constant variance and skewness:

- precipitation was transformed to better handle skewness and many zero values;
- average wind speed was log-transformed;
- lagged precipitation was transformed using an inverse-power transformation with a small offset to handle zeros.

Because the response variable was transformed reciprocally, coefficient directions must be interpreted carefully. A negative coefficient on the transformed response scale generally corresponds to a positive effect on the original maximum temperature scale.

## 4. Exploratory Data Analysis

Exploratory data analysis was used to understand the response variable and predictors before model fitting.

The EDA included:

- distribution of daily maximum temperature;
- histograms of key weather predictors;
- seasonal plots using day-of-year;
- residual patterns against day-of-year;
- comparison of wet and dry seasonal temperature curves;
- visual checks for skewness, outliers, and nonlinear structure.

The wet-versus-dry seasonal curves showed different slopes across the year, visually supporting season-dependent precipitation effects and motivating the inclusion of interaction terms.

## 5. Model Building

The final model is a multiple linear regression model using transformed daily maximum temperature as the response variable.

The model includes:

- long-term year trend;
- seasonal sine and cosine terms;
- minimum temperature;
- precipitation;
- average wind speed;
- lagged precipitation;
- lagged average wind speed;
- fire-start-day indicator;
- meteorologically motivated interaction terms.

The model was designed not only to predict temperature, but also to explain how weather effects vary across seasons.

## 6. Model Selection

Candidate models were compared using stepwise model selection guided by AIC and BIC.

The AIC-selected model matched the preliminary full model, while the BIC-selected model was smaller and removed some interaction terms. To evaluate whether removing those terms caused a meaningful loss of fit, the models were compared using:

- overall ANOVA test;
- partial F-tests;
- coefficient-level t-tests;
- adjusted R-squared;
- model interpretability;
- hierarchy principle.

Although AIC and BIC disagreed slightly, the final model retained meteorologically interpretable interaction terms. This decision prioritized explanatory power, model hierarchy, and scientific reasoning rather than relying only on automatic selection.

## 7. Model Diagnostics

The final model was evaluated using several diagnostic methods:

- response vs fitted values;
- residuals vs fitted values;
- normal Q-Q plot;
- residuals vs day of year;
- residuals vs transformed predictors;
- pairwise scatterplots;
- interaction visualizations.

These diagnostics were used to check:

- linearity;
- normality of residuals;
- constant variance;
- remaining seasonal structure;
- influential observations;
- whether transformations improved model fit.

## 8. Multicollinearity and Influence Diagnostics

Multicollinearity was checked using VIF and GVIF. Some values were affected by the presence of interaction terms, but the diagnostics did not suggest severe harmful multicollinearity that required removing major predictors.

Potential problematic observations were evaluated using:

- leverage;
- standardized residuals;
- Cook’s distance;
- DFFITS;
- DFBETAS.

Some observations were flagged by influence diagnostics, but they were not removed unless there was strong evidence that they were invalid. This avoids deleting real extreme weather observations simply because they are influential.

## 9. Model Validation

To assess generalization and guard against overfitting, the final model was validated using leave-one-out cross-validation (LOOCV).

The LOOCV error was consistent with the model’s training performance, suggesting that the model was reasonably stable and could predict unseen daily observations without strong overfitting.

## Key Results

The final model suggests that daily maximum temperature in California is driven more by specific meteorological conditions than by a simple uniform warming trend.

Main findings include:

- Minimum temperature is a major driver: warmer overnight low temperatures are associated with higher daily maximum temperatures.
- Dry conditions increase maximum temperature: lower precipitation reduces evaporative cooling and is associated with hotter days.
- Wind speed contributes to heating effects: higher wind speeds are associated with higher maximum temperatures, consistent with hot-wind phenomena.
- Seasonality matters strongly: sine and cosine terms capture the expected annual cycle of hot summers and cooler winters.
- Weather effects are season-dependent: wind and precipitation effects are stronger or weaker depending on the season.
- Long-term trend must be interpreted carefully: after controlling for meteorological variables, the year trend does not simply show a uniform upward shift in baseline temperature.

Overall, the analysis suggests that future extreme heat patterns may be better understood through the frequency and intensity of dry, windy, and warm-night conditions rather than through a simple linear warming baseline alone.

## Conclusion

This project demonstrates that predicting California daily maximum temperature requires modeling both seasonal cycles and dynamic meteorological drivers.

Instead of relying only on a passive long-term warming trend, the model shows that maximum temperature is strongly influenced by dry conditions, high wind speeds, warmer overnight temperatures, and season-dependent interactions. These factors can intensify daily heat and may contribute to wildfire-related risk conditions.

The results suggest that future warming and extreme heat risk in California may appear through more frequent and intense hot-wind and dry-spell events, rather than only through a uniform upward shift in regional temperature.

## Limitations

Several limitations remain:

- Transformed response interpretation: The response variable was transformed to improve normality and stabilize variance. As a result, coefficients are on the transformed scale and are less intuitive. Future work could report marginal effects back-transformed to the original temperature scale.
- Zero-heavy precipitation variable: Daily precipitation contains many zero values. A single linear model may not fully capture the difference between no-rain days and rainy days. A future improvement could use a two-part model: logistic regression for rain/no-rain and a continuous model for rainfall amount on rainy days.
- Stationary seasonality assumption: The model assumes that the seasonal cycle is stable across decades through fixed sine and cosine terms. However, climate change may shift the timing or amplitude of seasons. Future work could allow seasonality to vary over time and use time-aware cross-validation.

## Project Highlight

Before completing this final version, an earlier dataset was rejected after statistical diagnostics suggested serious reliability issues. The project was rebuilt from a new dataset under a tight deadline, including data selection, cleaning, feature engineering, modeling, diagnostics, and validation.

This experience strengthened the focus on data quality assessment, model reliability, and responsible statistical interpretation rather than mechanically fitting a model.

## How to Run

Clone this repository.
Open new_project.Rmd in RStudio.
Install the required R packages.
Place the dataset file in the data/ folder, or update the file path in the R Markdown file.
Knit the R Markdown file to HTML or PDF.

Required packages can be installed with:

```r
install.packages(c(
  "ggplot2",
  "dplyr",
  "tidyr",
  "broom",
  "skimr",
  "car",
  "MASS",
  "patchwork",
  "gridExtra",
  "pander",
  "stargazer"
))
```

## Skills Demonstrated

- Applied statistical modeling
- Data cleaning
- Feature engineering
- Exploratory data analysis
- Seasonal feature construction
- Multiple linear regression
- Response and predictor transformations
- Interaction term design
- AIC/BIC model comparison
- Partial F-tests
- Regression diagnostics
- VIF/GVIF multicollinearity checking
- Influence diagnostics
- LOOCV model validation
- R Markdown reporting
- Data quality assessment
- Statistical communication through poster presentation

## References

Yavas, C. E., Kadlec, C., Kim, J., & Chen, L. (2025). California Weather and Fire Prediction Dataset (1984–2025) with Engineered Features [Data set]. Zenodo. https://doi.org/10.5281/zenodo.14712845

National Oceanic and Atmospheric Administration. (2022, August 8). Wildfire climate connection. U.S. Department of Commerce. https://www.noaa.gov/noaa-wildfire/wildfire-climate-connection

California Department of Forestry and Fire Protection. (2025). Statistics. CAL FIRE. https://www.fire.ca.gov/our-impact/statistics

Mass, C. F., & Ovens, D. (2019). The northern California wildfires of 8–9 October 2017: The role of a major downslope wind event. Weather, 74(8), 235–256. https://doi.org/10.1002/wea.3495
