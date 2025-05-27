# Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML

# Project Overview
The Amazon region's abundant surface water contrasts with its heavy reliance on groundwater, with over two-thirds of its urban population depending on this resource for domestic and industrial use. The Alter do Chão Aquifer (ACA) is one of Brazil's largest and most strategically important freshwater resources, extending beneath parts of the Amazonas, Pará, and Amapá states. Despite its significance, the seasonal and long-term fluctuations of the ACA remain poorly understood, particularly regarding multi-scale relationships between regional satellite observations and local ground measurements.
This study addresses critical challenges in Amazon groundwater monitoring by:

- Quantifying scale relationships between regional GRACE/GLDAS satellite data and local well measurements
- Developing machine learning approaches for uncertainty quantification in data-sparse regions
- Analyzing temporal patterns across different monitoring scales using Bayesian and leave-one-out validation methods
- Evaluating the potential of multi-sensor fusion for groundwater monitoring

# Study Area
The analysis focuses on a ~400 km² region within the Alter do Chão Aquifer system, selected based on Sentinel-1 image coverage. The study area encompasses:

- Location: Central Amazon, Pará State, Brazil
- Coordinates: ~60.8°W to ~58.2°W, ~3.7°S to ~1.7°S
- Hydrogeology: Predominantly sandy formations with high porosity and permeability
- Climate: Strong seasonal precipitation patterns (wet: Nov-Apr, dry: May-Oct)
- Monitoring: Limited ground-based infrastructure typical of remote Amazon regions

| Dataset | Source | Temporal Resolution | Spatial Resolution | Coverage | Format |
| --- | --- | --- | --- | --- | --- |
| GRACE-FO | NASA JPL | Monthly (2019)  | 0.5 x 0.5 | Global | NetCDF |
| GLDAS-2.1 Noah | NASA GSFC | Monthly (2019) | 0.25° x 0.25°  | Global | NetCDF |
| Well Measurements | RIMAS (Brazil) | Monthly (2019) | Point-based  | 4 wells in ROI | CSV |
| Sentinel-1 SAR | ESA Copernicus | Monthly (2019) | 10m  | Regional ROI | Geo TIFF |

# Methodology
## Data Processing Pipeline
- Region of Interest Definition:

The region of Interst is extracted from Sentinel-1 image coverage with a 0.05° buffer

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_5.00.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_0.05.png" alt="Second image" width="400"></td>
  </tr>
</table>

- Well Data Analysis:

The well data represents monthly groundwater level changes, directly measured by the RIMAS monitoring network within Brazilian aquifers. The graph below illustrates the average month-on-month change in groundwater levels across the 4 wells that fall within my region of interest. It clearly depicts the hydrological cycle, showing significant groundwater level increases during the wet season, with a lag of 1-2 months into June, followed by marked declines during the dry season with a similar lag period.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/7946eea39a512b3a0fb471044ede44492f6ef97c/Images/well_analysis%20(1).png)


- GRACE/GLDAS Processing:

The GRACE dataset provides Total Water Storage (TWS) anomalies, representing the sum of all water stored on and within the Earth's landmasses, including surface water, soil moisture, snow, ice, and groundwater. GRACE data is retrieved at a 1-degree spatial resolution and a monthly time step. The GLDAS dataset provides surface water components such as soil moisture (across layers: 0-10 cm, 10-40 cm, 40-100 cm, 100-200 cm), snow water equivalent (SWE), and canopy water storage, at a 0.25-degree resolution. Using these datasets, I calculate the groundwater storage anomaly (GWSA) within the region of interest using the water balance approach: (TALK ABOUT THE UP/DOWN SAMPLING I'VE DONE)

Groundwater Storage Anomaly (GWSA) = TWS - (Soil Moisture + Snow Water + Canopy Water)

From these anomalies, I derive the month-on-month change in groundwater level, representing the average change in groundwater storage from one month to the next, used as the primary variable for prediction.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/33dd6aa3a78b9723947cdfa60cf03ec2764df239/Images/grace_gwa_change_analysis%20(1).png)


- Sentinel-1 Processing:

Sentinel-1 backscatter data (VV and VH polarizations) are processed to compute regional monthly averages. These statistics capture surface conditions that may correlate with groundwater dynamics, such as soil moisture or vegetation changes.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_analysis.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_normalized_analysis.png" alt="Second image" width="400"></td>
  </tr>
</table>


## Machine Learning Framework
With only 12 monthly samples, traditional ML approaches risk overfitting. This study implements methods tailored for small datasets:


Bayesian Uncertainty Quantification

Purpose: Quantify prediction uncertainty for groundwater changes.
Method: Bayesian Ridge Regression with regularization to provide honest uncertainty estimates.
Output: Predictions with confidence intervals (±23.772 cm mean uncertainty).

Leave-One-Out Cross-Validation (LOO)

Purpose: Unbiased performance evaluation for small datasets.
Method: Each month is predicted using the remaining 11 months as training data.

Anomaly Detection

Purpose: Identify unusual groundwater patterns
Method: Random Forest classification to detect seasonal deviations.
Output: Anomalous months and their driving factors.

## Scale Relationship Analysis
Strong positive correlation (r = 0.886, p = 0.000) between GRACE regional month-on-month changes and local well storage changes.

Physical Interpretation:

GRACE Data: Represents regional groundwater changes derived from TWS anomalies, adjusted for surface water components.

Well Data: Local month-on-month changes in groundwater storage, reflecting aquifer-level dynamics.

Positive Correlation: Indicates that regional GRACE changes align closely with local well measurements, suggesting consistent hydrological processes at both scales in this region.


Implications:

Validates GRACE as a reliable proxy for local groundwater changes in this context.

Suggests good aquifer connectivity between local and regional scales, with minimal lag in response times.

# Results 
## Temporal Pattern Analysis
GRACE/GLDAS Results:

- Seasonal amplitude: ~0.83 m between peak recovery and maximum depletion
- Maximum depletion: November (-1.04 m)
- Minimum depletion: June (-0.21 m)
- Pattern follows expected Amazon hydrology with 1-2 month lag after precipitation

Well Measurements:

- Range: 20.1 to 27.7 cm (monthly averages)
- Inverse seasonal pattern relative to GRACE
- Wet season: Higher relative storage (>25 cm)
- Dry season: Lower relative storage (<23 cm)

Sentinel-1 Analysis:

- Generally follows expected seasonal pattern (higher in dry season)
-Notable anomalies: February peak (flooding) and August/September minimum (severe drought)
- Weak correlation with groundwater variables (r < 0.3)

## Machine Learning Performance
Multi-Sensor Model (GRACE + Sentinel-1 + Temporal features):

Leave-One-Out R²: 0.90-0.91 (excellent generalization)
Mean Absolute Error: 0.73-0.80 cm
Bayesian Uncertainty: ±0.96 cm (realistic confidence intervals)

GRACE-Only Model:

Leave-One-Out R²: 0.78-0.82 (strong performance)
Feature Importance: GRACE groundwater anomaly most important
Validation: Robust across all months

Key Insight: Adding Sentinel-1 features marginally improves performance but increases overfitting risk. GRACE-only models provide more reliable predictions for this application.
Anomaly Detection Results
Anomalous Months Identified: June, November, December

Physical Interpretation: Transition periods between wet/dry seasons
Feature Importance: Balanced across GRACE TWS (36%), Wells (32%), and GRACE GWA (32%)
Classification Accuracy: 100% (note: small sample limitation)

#Limitations and Uncertainties 
Data Limitations

Temporal scope: Single year (2019) limits long-term trend analysis
Spatial coverage: Small ROI (~400 km²) may not represent broader Amazon
Well density: Only 4 wells for ground truth validation
Scale mismatch: GRACE pixels (~100 km) vs. point measurements

Methodological Considerations

Small sample size: 12 monthly observations limit ML model complexity
Reference baseline differences: GRACE and well data use different historical baselines
Interpolation uncertainty: Limited well data restricts spatial interpolation accuracy

Statistical Caveats

Cross-validation performance may be optimistic due to temporal autocorrelation
Feature selection limited by sample size constraints
Generalization beyond study period uncertain

# Future Research Directions
- Multi-year analysis to validate temporal patterns and assess climate variability impacts
- Expanded spatial coverage across different Amazon sub-regions
- Integration of precipitation data to understand driving mechanisms
- Lag analysis to quantify response times between datasets
- Comparison with hydrological models for process understanding





