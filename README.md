# Predicting Monthly Groundwater Storage Changes in the Amazon Basin: A Multi-Sensor Machine Learning Approach Using GRACE, Sentinel-1, and Ground-Based Observations

# Project Overview
The Amazon region's abundant surface water contrasts with its heavy reliance on groundwater, with over two-thirds of its urban population depending on this resource for domestic and industrial use. The Alter do Chão Aquifer (ACA) is one of Brazil's largest and most strategically important freshwater resources, extending beneath parts of the Amazonas, Pará, and Amapá states. Despite its significance, the monthly dynamics and storage changes of the ACA remain poorly understood, particularly regarding the relationship between regional satellite-derived storage changes and local groundwater fluctuations measured at well sites.
This study develops a machine learning (ML) framework to predict monthly groundwater storage changes using multi-sensor satellite observations. The research addresses challenges in Amazon groundwater monitoring by establishing direct relationships between GRACE-derived regional storage changes and local well measurements, enhanced by surface moisture indicators from Sentinel-1 radar and seasonal pattern analysis. The approach combines multiple data sources to achieve 82% variance explanation in groundwater storage change predictions, providing both point estimates and uncertainty quantification essential for data-sparse regions.
Key contributions include quantifying scale relationships between regional GRACE storage changes and local well dynamics, developing robust uncertainty quantification methods using Bayesian and bootstrap approaches, and demonstrating the effectiveness of comprehensive multi-sensor feature integration for groundwater monitoring applications. The framework provides operationally useful prediction intervals (±17.5 cm) suitable for trend detection, anomaly identification, and informed water management decision-making in remote Amazon regions.

# Study Area

The analysis focuses on a ~60,000 km² region in central Amazon, Pará State, Brazil (60.8°W–58.2°W, 3.7°S–1.7°S), selected for its hydrogeological diversity and suitability for integrating satellite data with local well measurements. The area features high-porosity sandy formations typical of the Alter do Chão Aquifer, enabling strong regional–local groundwater correlation (r = 0.888). With clear wet (Nov–Apr) and dry (May–Oct) seasons, it provides ideal conditions for capturing seasonal groundwater dynamics. Sparse ground infrastructure further emphasizes the value of remote sensing and robust uncertainty quantification in monitoring groundwater in data-limited regions.

# Datasets

- GRACE (Gravity Recovery and Climate Experiment): GRACE is a satellite mission that measures tiny changes in Earth's gravity field caused by shifts in mass — primarily due to water movement. By detecting how much mass is added or lost in a region each month, GRACE provides regional estimates of total water storage (including groundwater, soil moisture, surface water, and snow) over time. It’s especially valuable for monitoring groundwater in areas with limited on-the-ground data.

- GLDAS (Global Land Data Assimilation System): GLDAS is a land surface modeling system that uses satellite and ground-based observations to simulate surface water storage components like soil moisture, canopy water, and snow. When paired with GRACE, GLDAS helps separate the groundwater component by subtracting surface contributions, allowing us to estimate how much water is stored underground.

- Sentinel-1: Offers high-resolution surface moisture indicators via dual-polarization radar (VV/VH), adding local-scale context to improve model predictions.

- In-situ Wells: Point-scale groundwater level measurements, used as both the target variable.

| Dataset | Source | Temporal Resolution | Spatial Resolution | Coverage | Format |
| --- | --- | --- | --- | --- | --- |
| GRACE-FO | NASA JPL | Monthly (2019)  | 1° x 1° | Global | NetCDF |
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

The well data represents monthly groundwater level changes, directly measured by the RIMAS monitoring network within Brazilian aquifers. The graph below illustrates the **average month-on-month change in groundwater levels** across the 4 wells that fall within my region of interest. It clearly depicts the hydrological cycle, showing significant groundwater level increases during the wet season, with a lag of 1-2 months into June, followed by marked declines during the dry season with a similar lag period.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/7946eea39a512b3a0fb471044ede44492f6ef97c/Images/well_analysis%20(1).png)


- **GRACE/GLDAS Processing:**

The GRACE dataset provides Total Water Storage (TWS) anomalies, representing the sum of all water stored on and within the Earth's landmasses, including surface water, soil moisture, snow, ice, and groundwater. GRACE data is retrieved at a 1-degree spatial resolution and a monthly time step. The GLDAS dataset provides surface water components such as soil moisture (across layers: 0-10 cm, 10-40 cm, 40-100 cm, 100-200 cm), snow water equivalent (SWE), and canopy water storage, at a 0.25-degree resolution. Using these datasets, I calculate the groundwater storage anomaly (GWSA) within the region of interest using the water balance approach: (TALK ABOUT THE UP/DOWN SAMPLING I'VE DONE)

Groundwater Storage Anomaly (GWSA) = TWS - (Soil Moisture + Snow Water + Canopy Water)

From these anomalies, I derive the **month-on-month change in groundwater level**, used as the primary variable for prediction.

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

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Model%20Comparison.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Best%20Model%20Performance.png" alt="Second image" width="400"></td>
  </tr>
</table>



## **Uncertainty Analysis**

Understanding how confident we can be in each prediction is crucial—especially with a small dataset of only 12 months. To address this, I implemented two complementary uncertainty quantification methods:

- Bayesian Ridge Regression provides internal model-based uncertainty, estimating prediction intervals of approximately ±23 cm.

- Bootstrap resampling (100 iterations) captures variability introduced by the data itself, producing wider intervals of ±36 cm.

Despite the dataset's limited size, results are promising. Bayesian predictions (red squares) closely track actual well measurements (blue circles) throughout the year, successfully capturing seasonal dynamics from +80 cm in the wet season to -70 cm in the dry season. However, the wider bootstrap intervals (green shading) highlight that small sample size introduces greater variability than the model alone anticipates. The ~60% difference between the two interval widths provides a valuable insight: while the model performs well on available data, the more conservative bootstrap intervals offer a more reliable basis for real-world decision-making.

Importantly, the majority of observed values fall within both uncertainty bounds—especially during key seasonal transitions.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/997f4358d4da4851065c370c9f23a792dd439fc2/Images/uncertainty_time_series.png)

## Feature Importance

Feature importance analysis helped highlight which variables matter most when predicting changes in groundwater storage—key for understanding what drives aquifer behavior. This is especially useful in data-scarce regions like the Amazon, where long-term ground measurements are limited. By showing which remote sensing signals and seasonal trends are most valuable, the analysis can also help focus future data collection efforts.

In this project, I used several models (LinearRegression, Ridge, and Bayesian Ridge) and methods to assess feature importance. Across the board, GRACE satellite data stood out as the most important predictor, confirming its strong link to groundwater changes. Sentinel-1 radar data, especially VH polarization, was also highly influential in some models, likely reflecting interactions between vegetation and surface water. Seasonal patterns showed moderate importance, especially the cosine signal tied to the annual water cycle. Regularization in Ridge made feature contributions more balanced, while the Bayesian model added uncertainty estimates to the rankings. Overall, the findings reinforce GRACE data as the backbone of groundwater monitoring, with radar inputs offering valuable backup—particularly useful when GRACE data isn’t available.

# Limitations and Uncertainties 
This analysis faces key limitations primarily due to the small 12-month dataset, which constrains model complexity and increases sensitivity to assumptions. While the Linear Regression model explains 82% of groundwater storage variance—remarkable for such limited data—some overfitting is evident (training R² = 0.968 vs. cross-validated R² = 0.822).

Several simplifying assumptions introduce uncertainty: averaging four wells may obscure local-scale variability, Sentinel-1 backscatter is spatially averaged, and GRACE/GLDAS data assume regional uniformity across a coarse grid. Additionally, the model treats monthly samples as independent, which may not fully reflect temporal hydrological dynamics.

While scale mismatches between point-scale well measurements and coarse-resolution satellite data introduce some uncertainty, the strong correlation (r = 0.888) between the two suggests robust cross-scale consistency within this aquifer system. This project serves as a proof of concept, demonstrating the viability of satellite-driven groundwater prediction in data-sparse regions and laying the groundwork for more detailed, multi-year analysis in the future.

# Environmental Impacts

This groundwater analysis project consumed approximately 0.5 kWh of energy and generated 0.175 kg of CO₂ emissions (equivalent to driving ~0.9 km) during a complete analysis cycle lasting 2 minutes. The computational footprint includes processing multi-temporal GRACE satellite data (~0.5 GB), Sentinel-1 radar imagery (~2 GB), and well measurement datasets, alongside machine learning model training using linear regression, Ridge, and Bayesian Ridge algorithms. The analysis leverages standard CPU processing without GPU acceleration, which keeps energy consumption relatively low but could be optimized through vectorized operations and more efficient data loading techniques.
While the direct computational impact is modest, the project processes large-scale satellite datasets that require significant upstream energy for data acquisition, storage, and distribution by space agencies. However, this environmental cost is offset by the project's potential positive impacts: enabling sustainable groundwater management in the Amazon basin, supporting precision agriculture to reduce water waste, facilitating early drought detection, and providing critical data for climate adaptation strategies. The ML models' high accuracy (R² > 0.95) means efficient decision-making with minimal computational overhead for operational deployment. Future improvements could include migrating to renewable energy-powered cloud computing, implementing more efficient data processing pipelines, and developing lightweight model variants for real-time monitoring applications.



