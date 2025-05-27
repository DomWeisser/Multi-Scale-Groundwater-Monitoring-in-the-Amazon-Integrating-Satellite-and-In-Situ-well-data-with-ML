# Predicting Monthly Groundwater Storage Changes in the Amazon Basin: A Multi-Sensor Machine Learning Approach Using GRACE, Sentinel-1, and Ground-Based Observations

# Project Overview
The Amazon region's abundant surface water contrasts with its heavy reliance on groundwater, with over two-thirds of its urban population depending on this resource for domestic and industrial use. The Alter do Chão Aquifer (ACA) is one of Brazil's largest and most strategically important freshwater resources, extending beneath parts of the Amazonas, Pará, and Amapá states. Despite its significance, the monthly dynamics and storage changes of the ACA remain poorly understood, particularly regarding the relationship between regional satellite-derived storage changes and local groundwater fluctuations measured at well sites.
This study develops a comprehensive machine learning framework to predict monthly groundwater storage changes using multi-sensor satellite observations. The research addresses critical challenges in Amazon groundwater monitoring by establishing direct relationships between GRACE-derived regional storage changes and local well measurements, enhanced by surface moisture indicators from Sentinel-1 radar and seasonal pattern analysis. The innovative approach combines multiple data sources to achieve 82% variance explanation in groundwater storage change predictions, providing both point estimates and uncertainty quantification essential for water resource management in data-sparse regions.
Key contributions include quantifying scale relationships between regional GRACE storage changes and local well dynamics, developing robust uncertainty quantification methods using Bayesian and bootstrap approaches, and demonstrating the effectiveness of comprehensive multi-sensor feature integration for groundwater monitoring applications. The framework provides operationally useful prediction intervals (±17.5 cm) suitable for trend detection, anomaly identification, and informed water management decision-making in remote Amazon regions.

# Study Area

The analysis focuses on a ~400 km² region in central Amazon, Pará State, Brazil (60.8°W–58.2°W, 3.7°S–1.7°S), selected for its hydrogeological diversity and suitability for integrating satellite data with local well measurements. The area features high-porosity sandy formations typical of the Alter do Chão Aquifer, enabling strong regional–local groundwater connectivity (r = 0.888). With clear wet (Nov–Apr) and dry (May–Oct) seasons, it provides ideal conditions for capturing seasonal groundwater dynamics. Sparse ground infrastructure further emphasizes the value of remote sensing and robust uncertainty quantification in monitoring groundwater in data-limited regions.

# Datasets

- GRACE (Gravity Recovery and Climate Experiment): GRACE is a satellite mission that measures tiny changes in Earth's gravity field caused by shifts in mass—primarily water movement. By detecting how much mass is added or lost in a region each month, GRACE provides regional estimates of total water storage (including groundwater, soil moisture, surface water, and snow) over time. It’s especially valuable for monitoring groundwater in areas with limited on-the-ground data.

- GLDAS (Global Land Data Assimilation System): GLDAS is a land surface modeling system that uses satellite and ground-based observations to simulate surface water storage components like soil moisture, canopy water, and snow. When paired with GRACE, GLDAS helps separate the groundwater component by subtracting surface contributions—allowing us to estimate how much water is stored underground.

- Sentinel-1: Offers high-resolution surface moisture indicators via dual-polarization radar (VV/VH), adding local-scale context to improve model predictions.

- In-situ Wells: Deliver point-scale groundwater level measurements, used as both the target variable and validation benchmark for satellite-derived estimates.

| Dataset | Source | Temporal Resolution | Spatial Resolution | Coverage | Format |
| --- | --- | --- | --- | --- | --- |
| GRACE-FO | NASA JPL | Monthly (2019)  | 0.5 x 0.5 | Global | NetCDF |
| GLDAS-2.1 Noah | NASA GSFC | Monthly (2019) | 0.25° x 0.25°  | Global | NetCDF |
| Well Measurements | RIMAS (Brazil) | Monthly (2019) | Point-based  | 4 wells in ROI | CSV |
| Sentinel-1 SAR | ESA Copernicus | Monthly (2019) | 10m  | Regional ROI | Geo TIFF |

# Methodology
## Data Processing Pipeline
- **Region of Interest Definition:**

The region of Interst is extracted from Sentinel-1 image coverage with a 0.05° buffer

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_5.00.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_0.05.png" alt="Second image" width="400"></td>
  </tr>
</table>

- **Well Data Analysis:**

The well data represents monthly groundwater level changes, directly measured by the RIMAS monitoring network within Brazilian aquifers. The graph below illustrates the average month-on-month change in groundwater levels across the 4 wells that fall within my region of interest. It clearly depicts the hydrological cycle, showing significant groundwater level increases during the wet season, with a lag of 1-2 months into June, followed by marked declines during the dry season with a similar lag period.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/7946eea39a512b3a0fb471044ede44492f6ef97c/Images/well_analysis%20(1).png)


- **GRACE/GLDAS Processing:**

The GRACE dataset provides Total Water Storage (TWS) anomalies, representing the sum of all water stored on and within the Earth's landmasses, including surface water, soil moisture, snow, ice, and groundwater. GRACE data is retrieved at a 1-degree spatial resolution and a monthly time step. The GLDAS dataset provides surface water components such as soil moisture (across layers: 0-10 cm, 10-40 cm, 40-100 cm, 100-200 cm), snow water equivalent (SWE), and canopy water storage, at a 0.25-degree resolution. Using these datasets, I calculate the groundwater storage anomaly (GWSA) within the region of interest using the water balance approach: (TALK ABOUT THE UP/DOWN SAMPLING I'VE DONE)

Groundwater Storage Anomaly (GWSA) = TWS - (Soil Moisture + Snow Water + Canopy Water)

From these anomalies, I derive the month-on-month change in groundwater level, representing the average change in groundwater storage from one month to the next, used as the primary variable for prediction.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/33dd6aa3a78b9723947cdfa60cf03ec2764df239/Images/grace_gwa_change_analysis%20(1).png)


- **Sentinel-1 Processing:**

Sentinel-1 backscatter data (VV and VH polarizations) are processed to compute regional monthly averages. These statistics capture surface conditions that may correlate with groundwater dynamics, such as soil moisture or vegetation changes.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_analysis.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_normalized_analysis.png" alt="Second image" width="400"></td>
  </tr>
</table>


## Machine Learning Framework
In this analysis, I apply three machine learning (ML) models - Linear Regression, Ridge Regression and Bayesian Ridge Regression - to predict month-to-month changes in groundwater levels in the region of interest. Linear Regression achieved the highest performance (R² = 0.822), offering both accuracy and interpretability, which is critical for operational hydrology. Ridge Regression (α = 10) delivered lower accuracy (R² = 0.586) but enhanced robustness, highlighting the importance of meaningful feature selection over regularization strength. Bayesian Ridge Regression (R² = 0.674) provided a balanced approach with the added benefit of uncertainty quantification, essential for risk-informed water management.

The feature set combined GRACE-derived groundwater storage changes (month-to-month), groundwater anomalies (GWA) for contextual variability, and Sentinel-1 mean backscatter values (S1_mean_VH and S1_mean_VV) to capture surface moisture and vegetation water content. Additionally, temporal features—sine and cosine transformations of the month—were used to model systematic seasonal cycles independent of specific climate anomalies. This diverse, physically-informed feature set allowed the models to represent multiple hydrological processes simultaneously, leading to significantly improved predictive performance compared to models relying on a single data source.

Given the small dataset—comprising only 12 months of Sentinel-1, GRACE, and GLDAS data—careful validation is essential to avoid overfitting and to ensure reliable performance estimates. I implemented Leave-One-Out Cross-Validation (LOOCV), training the model on 11 months and testing on the remaining month in each iteration. This approach maximizes data use while maintaining statistical rigor, making it well-suited for small hydrological time series where monthly samples can be treated as conditionally independent. LOOCV yields unbiased, out-of-sample performance estimates, with the optimal Linear Regression model achieving a mean absolute error of 17.5 ± 13 cm, and errors ranging from 0.7 to 49 cm. These results confirm the model’s robustness and precision across varying seasonal and anomaly conditions, with consistently high performance (R² = 0.822) across all iterations.


# Results 

The Linear Regression model achieved exceptional predictive performance, with a cross-validation R² of 0.822, successfully explaining 82% of the variance in groundwater storage through integrated multi-sensor features. The training R² of 0.968 indicates strong pattern learning with only moderate overfitting, acceptable given the robust cross-validated results. The mean absolute error of 17.5 ± 13 cm, or roughly 11% of the total signal range, offers high precision suitable for groundwater monitoring, resource management, and trend analysis. A strong correlation (r = 0.888) between GRACE-derived regional storage and in-situ well measurements underscores the model’s ability to capture cross-scale aquifer dynamics. Feature importance analysis highlights the value of combining direct storage changes, groundwater anomalies, Sentinel-1 surface moisture indicators, and seasonal components, resulting in a synergistic model that captures the full complexity of groundwater behavior far beyond what single-source models can achieve.

## **Uncertainty Analysis**

Understanding how confident we can be in each prediction is crucial—especially with a small dataset of only 12 months. To address this, I implemented two complementary uncertainty quantification methods:

- Bayesian Ridge Regression provides internal model-based uncertainty, estimating prediction intervals of approximately ±23 cm.

- Bootstrap resampling (100 iterations) captures variability introduced by the data itself, producing wider intervals of ±36 cm.

Despite the dataset's limited size, results are promising. Bayesian predictions (red squares) closely track actual well measurements (blue circles) throughout the year, successfully capturing seasonal dynamics from +80 cm in the wet season to -70 cm in the dry season. However, the wider bootstrap intervals (green shading) highlight that small sample size introduces greater variability than the model alone anticipates. The ~60% difference between the two interval widths provides a valuable insight: while the model performs well on available data, the more conservative bootstrap intervals offer a more reliable basis for real-world decision-making.

Importantly, the majority of observed values fall within both uncertainty bounds—especially during key seasonal transitions.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/997f4358d4da4851065c370c9f23a792dd439fc2/Images/uncertainty_time_series.png)

## Anomaly Detection

Anomaly detection plays a crucial role in groundwater monitoring by identifying months where storage behavior deviates significantly from expected seasonal patterns. In data-scarce environments like the Amazon, where consistent long-term in-situ records are limited, anomaly detection helps flag unusual events that might indicate early signs of groundwater stress, measurement errors, or changes in climate and land use impacts.

In this project, I implemented three complementary methods to detect anomalies in monthly groundwater storage changes:
- Isolation Forest (machine learning-based),
- Elliptic Envelope (geometric outlier detection), and
- Z-score analysis (traditional statistical thresholding).

By using a consensus approach across these methods, the framework identified January, July, and December as consistently anomalous. Each of these months corresponds to a critical point in the Amazonian hydrological cycle:
- January: The start of the time series and the baseline month, potentially skewed due to its role as a reference point.
- July: The peak of the dry season, when groundwater reaches its lowest levels and aquifer stress is likely highest.
- December: A transition period marking the start of wet season recharge, often associated with rapid storage changes.

# Limitations and Uncertainties 
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





