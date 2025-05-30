---
layout: post
title: "Forecasting Switzerland’s Net Electricity Imports using Weather Data"
author: Fabian Stepinski & Patrick Häusermann
---

## Does Switzerland’s net electricity import correlate with local weather?

Switzerland’s electricity grid operates within a tightly interconnected European energy system. Despite high domestic generation capacity, particularly from hydroelectric power, Switzerland frequently imports and exports electricity to balance its grid. Fluctuations in weather conditions—impacting both renewable energy production and energy consumption—make forecasting net imports a non-trivial challenge.

This blog post outlines a data-driven approach to forecasting Switzerland’s **hourly net electricity imports** using historical weather data and machine learning techniques. The model aims to support short-term operational decisions, offering insights into grid balancing, energy planning, and risk mitigation.

## The Problem

Predicting electricity imports is complex. The net flow of electricity depends on numerous factors, including:

- Domestic energy production, largely hydro-dependent.
- Temperature-driven demand fluctuations.
- Transmission and storage constraints.
- Cross-border energy market dynamics.

Traditional forecasting models often focus on long-term strategic planning or market price volatility. In contrast, this work targets **short-term hourly forecasting** using weather data, a time scale critical for grid operators and energy traders.

## Data Pipeline

The forecasting system is built around a robust data pipeline:

1. **Data Acquisition**:  
   - Weather data sourced from [weatherapi.com](https://weatherapi.com)
   - Energy data (net imports) from [opendata.swiss](https://opendata.swiss)
   - Temporal resolution: hourly

2. **Preprocessing**:
   - Feature engineering: temperature, humidity, wind speed, solar radiation, etc.
   - Lag features to account for temporal dependencies
   - Normalization and missing value imputation

3. **Labeling**:
   - Classification task: binary label indicating surplus or deficit
   - Regression task: continuous label indicating net import volume in GWh

4. **Modeling**:
   - Multi-Layer Perceptron (MLP) classifier for surplus/deficit prediction
   - XGBoost regressor for exact volume estimation

![Diagram Data Pipeline](./assets/img/2025-05-30-group06-diagram_data_pipeline.png)

## Results

### Classification Task: Surplus vs. Deficit

A **Multi-Layer Perceptron (MLP)** was trained to classify whether Switzerland will have a net electricity surplus or deficit in the next hour.

- **Accuracy**: ~91%
- **Feature importance**: temperature, cloud cover, and previous hour’s net import data were strong predictors

This high accuracy demonstrates the feasibility of using weather data for binary operational forecasting.

### Regression Task: Predicting Net Import Volume

An **XGBoost** regression model was used to estimate the actual net electricity import volume.

- **Mean Absolute Error (MAE)**: ~485 GWh
- This level of precision is operationally useful for energy managers

Both models performed well on held-out validation sets, suggesting good generalization capabilities.

## Dashboard and Visualization

The final models are integrated into a **Power BI dashboard** that provides:

- Hourly predictions with confidence intervals
- Visualization of weather trends and their correlation with energy flow
- Historical vs. predicted data overlays for anomaly detection

This makes the model not just a forecasting tool, but also a **decision-support system** for operational teams.

## Conclusion

The project successfully demonstrates that short-term electricity import forecasting in Switzerland can be significantly improved using weather data and modern machine learning models. The combination of classification and regression approaches allows both qualitative (surplus/deficit) and quantitative (GWh) insights.

In future work, model performance can be enhanced by:

- Incorporating additional features such as market prices and grid load
- Using ensemble methods or hybrid architectures
- Integrating real-time feedback for continuous learning

With increasing dependence on renewable energy and ongoing climate volatility, such forecasting tools will become indispensable for national grid stability.

## References

- [weatherapi.com](https://weatherapi.com) – Weather data
- [opendata.swiss](https://opendata.swiss) – Swiss energy data
- Power BI Dashboard for result visualization
- ETH Zurich and SwissInfo articles on hydroelectric dependency and grid strategy
