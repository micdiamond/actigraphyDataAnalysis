# Statistical Analysis Methods

## Overview

This document provides a succinct summary of the statistical methods employed in my analysis, emphasizing the nested nature of the data, the application of the multilevel model framework, and the essential role of the RA7 variable.


## Abstract

The overarching aim of the study was to investigate whether an actigraphy
derived measure of circadian rhythm robustness can predict a clinically important measure of mood dysregulation in Bipolar Disorder. 
This aim was motivated by the potential to develop a useful tool to provide critical early warning signs in Bipolar Disorder. Secondary data analyses were conducted on a unique repeated measures dataset containing continuous actigraphy and self-report daily mood data collected from a small sample of people with Bipolar Disorder. A multilevel modelling approach was used to test whether a new actigraphy-based measure of circadian robustness could prospectively predict a new measure of self-reported mood dysregulation in the dataset. Specifically, it was hypothesised that Relative Amplitude of the circadian rhythm (RA) averaged across the previous week (RA7) would significantly predict the following days’ Mood Dysregulation score.

## Table of Contents

- [Data Structure](#data-structure)
- [Multilevel Model Framework](#multilevel-model-framework)
- [RA7 - Circadian Robustness](#ra7---circadian-robustness)
- [Time Lag Integration](#time-lag-integration)
- [Model Building](#model-building)
- [Model Assessment](#model-assessment)
- [Assumptions & Checks](#assumptions--checks)
- [Handling Outliers](#handling-outliers)
- [Missing Data](#missing-data)
- [Tools & Packages](#tools--packages)

---

### Data Structure
- Nested data: days within participants.

### Multilevel Model Framework
- Handles non-independent data points per participant.
- Predictor (RA7): Centered around each participant's mean.
- Outcome (Mood Dysregulation score): Not centered.

### RA7 - Circadian Robustness
- Derived from the 24-hour RA variable.
- Calculation: M10 (activity for the 10 most active hours) minus L5 (activity for the five least active hours).
- RA7 provides a seven-day average for stable circadian robustness, reducing inter-daily variability.
- Calculation uses a time-series sliding window approach, considering only periods without missing RA values in the seven-day window.

### Time Lag Integration
- A lag of -1 day between predictor and outcome to explore predictive associations.

### Model Building
- Started with a fixed intercept model.
- Progressively introduced: Random intercepts, Fixed slopes, Random slopes.

```R
# Fixed Intercept Model
fixedInterceptOnlyAct <- gls(xLagCmood ~ 1, data = act, method = "ML", na.action = na.exclude)

# Random Intercept Model
randomInterceptOnlyAct <- lme(xLagCmood ~ 1, data = act, random = ~1|ID, method = "ML", na.action = na.exclude)
```

### Model Assessment
- Likelihood Ratio Tests to compare model fits (using ANOVAs)
- AIC values for fit vs. complexity evaluation.
- Nagawaka’s R^2 values for explanatory power measurement.
- ICC value for variance quantification due to participant differences.

```R
# Likelihood Ratio Test Example
anova(fixedInterceptOnlyAct, randomInterceptOnlyAct)

# AIC Calculation Example
AIC_value <- AIC(randomInterceptOnlyAct)

# Nagawaka's R^2 Calculation Example
R2_nagawaka <- calc_R2(randomInterceptOnlyAct)

# ICC Calculation Example
ICC_value <- ICC(randomInterceptOnlyAct)

```


### Assumptions & Checks
- Verified normality via Q-Q plots and histogram of residuals.
- Non-constant Variance (NCV) test for heteroscedasticity check.

```R
# Shapiro-Wilk Normality Test Example
shapiro.test(act$x7RA)

# Residual Histogram Example
hist(residuals(xUntransformedModel), xlim = range(-30:10), ylim = range(0:200))

# Non-constant Variance Test (Heteroscedasticity)
library(car)
car::ncvTest(fixedSlopeRandomInterceptRA)
```

#### Aussumptions test through plotting
```{r}
# Residuals vs. Fitted Values Plot (Linearity of Residuals)
plot(fitted(fixedSlopeRandomInterceptRA), residuals(fixedSlopeRandomInterceptRA), main = "Residuals vs. Fitted")


# Fitted Values vs. Residuals Plot (Homogeneity of Variance)
plot(fitted(fixedSlopeRandomInterceptRA), residuals(fixedSlopeRandomInterceptRA), main = "Fitted vs. Residuals")


# Q-Q Plot of Residuals (Normality of Residuals)
qqnorm(residuals(fixedSlopeRandomInterceptRA))
qqline(residuals(fixedSlopeRandomInterceptRA))

# Histogram of Residuals
hist(residuals(fixedSlopeRandomInterceptRA), main = "Histogram of Residuals")

```



### Handling Outliers
- Detailed inspection rather than elimination.
- Focus on extreme mood variations fundamental to BD.

```R
# Cook's Distance Example
cooksd <- cooks.distance(xB)
```

### Missing Data
- Treated based on their position on the activity curve.
  - Data from the base: Replaced using linear interpolation.
  - Peak data: Entire 24-hour period removed due to high variability.

```R
# Missing Data Treatment Example
# Linear Interpolation for Base Data
interpolate_missing_base <- interpolate_base_data(data)

# Removing Peak Data
remove_peak_data(data)

```

### Tools & Packages
- Conducted in R software using various specialized packages:
  - `nlme`
  - `tidyverse`
  - `zoo`
  - `ggplot2`
  - `stats`
  - `moments` (for skewness and kurtosis)
  - `psychometric`

---

