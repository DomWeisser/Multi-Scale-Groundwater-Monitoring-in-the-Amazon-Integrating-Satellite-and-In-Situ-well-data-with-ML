# Predicting Monthly Groundwater Storage Changes in the Amazon Basin: A Multi-Sensor Machine Learning Approach Using GRACE, Sentinel-1, and Ground-Based Observations

# Project Overview
The Amazon region's abundant surface water contrasts with its heavy reliance on groundwater, with over two-thirds of its urban population depending on this resource for domestic and industrial use. The Alter do Chão Aquifer (ACA) is one of Brazil's largest and most strategically important freshwater resources, extending beneath parts of the Amazonas, Pará, and Amapá states. Despite its significance, the monthly dynamics and storage changes of the ACA remain poorly understood, particularly regarding the relationship between regional satellite-derived storage changes (GRACE imagery) and local groundwater fluctuations measured at well sites. Furthemore, in recent years (2016, 2023) severe droughts have further emphasied the need to understand the monthly dynamics of local groundwater supplies.
This study develops a proof of concept machine learning (ML) framework to predict month-on-month groundwater storage changes in local wells using satellite observations. I aim to understand the relationship between GRACE-derived regional storage changes (which have a large spatial resolution of 1° x 1°) and in-situ well observations, enhanced by surface moisture indcators from Sentinel-1 radar and seasonal pattern analysis. Given the regions remoteness, only 4 wells provide monthly groundwater change data for 2019 within my region of interest which I average to create a single value for the region of interest. While this is an over simplification the scarcity of the wells in relation to the region of interest means interpolation will provide unreliable results. Similarly, I use an average result for sentinel-1 backscatter across the region of interest. Again a simplification but because the area is predominantly covered with rainforest, I feel this is suitable for this analysis. Even with this data-scarcity I achieve 81% variance explanation (R-squared=0.81) in month-on-month groundwater storage change predictions, while providing both point estimates and uncertainty quantificatinos through Bayesian and Bootstrap approaches. While I am limited with the data for this project, these promising results indicate this is a project that can be developed with more time and access to in-situ well data.


# Study Area

My analysis focuses on a ~60,000 km² region in Manaus, Brazil (60.8°W–58.2°W, 3.7°S–1.7°S), selected for its hydrogeological diversity and suitability for integrating satellite data with local well measurements. The area features high-porosity sandy formations typical of the ACA and with clear wet (Nov–Apr) and dry (May–Oct) seasons, it provides ideal conditions for capturing seasonal groundwater dynamics. Sparse ground infrastructure outside the city further emphasizes the value of remote sensing and robust uncertainty quantification in monitoring groundwater in data-limited regions.

# Datasets

- GRACE (Gravity Recovery and Climate Experiment): GRACE is a satellite mission that measures tiny changes in Earth’s gravity field caused by shifts in mass, primarily due to the movement of water. The mission uses a pair of satellites that orbit Earth about 137 miles (220 kilometers) apart. As they follow one another, they constantly send microwave signals back and forth to precisely measure the distance between them. When the lead satellite passes over an area with slightly stronger gravity—typically due to a greater concentration of mass such as water or ice—it is pulled slightly ahead, changing the distance between the two satellites. The trailing satellite experiences the same pull as it crosses the anomaly, allowing the system to detect even minute variations in Earth's gravitational field. Each satellite is equipped with an accelerometer to account for non-gravitational forces like atmospheric drag, and GPS receivers to track their exact positions within a centimeter. Together, these instruments allow scientists to isolate gravity-related changes and create monthly maps of Earth’s gravity field. These maps reveal how mass, especially water in forms such as groundwater, soil moisture, surface water, and snow, moves over time. This data is particularly valuable for monitoring water resources in regions lacking ground-based observations.

- GLDAS (Global Land Data Assimilation System): GLDAS is a land surface modeling system that uses satellite and ground-based observations to simulate surface water storage components like soil moisture, canopy water, and snow. When paired with GRACE, GLDAS helps separate the groundwater component by subtracting surface contributions, allowing us to estimate how much water is stored underground.

- Sentinel-1: Offers high-resolution surface moisture indicators via dual-polarization radar (VV/VH), adding local-scale context to improve model predictions.

- In-situ Wells: The well data represents monthly groundwater level changes, directly measured by the RIMAS monitoring network within Brazilian aquifers. Point-scale groundwater level measurements are used as the target variable.

| Dataset | Source | Temporal Resolution | Spatial Resolution | Coverage | Format |
| --- | --- | --- | --- | --- | --- |
| GRACE-FO | NASA JPL | Monthly (2019)  | 1° x 1° | Global | NetCDF |
| GLDAS-2.1 Noah | NASA GSFC | Monthly (2019) | 0.25° x 0.25°  | Global | NetCDF |
| Well Measurements | RIMAS (Brazil) | Monthly (2019) | Point-based  | 4 wells in ROI | CSV |
| Sentinel-1 SAR | ESA Copernicus | Monthly (2019) | 10m  | Regional ROI | Geo TIFF |

# Methodology
## Data Processing Pipeline
- **Region of Interest Definition:**

The region of Interest is equivalebt to the bounds from the Sentinel-1 images with a 0.02° buffer

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_5.00.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_0.05.png" alt="Second image" width="400"></td>
  </tr>
</table>

- **Well Data Analysis:**

The graph below illustrates the average **month-on-month change in groundwater levels** across the 4 wells that fall within my region of interest. It clearly depicts the hydrological cycle, showing significant groundwater level increases during the wet season, with a lag of 1-2 months resulting in June, the start of the dry season, having the most groundwater, followed by marked declines during the dry season with a similar lag period, resulting in November having the lowest levels of groundwater.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/7946eea39a512b3a0fb471044ede44492f6ef97c/Images/well_analysis%20(1).png)


- **GRACE/GLDAS Processing:**

The GRACE dataset provides Total Water Storage (TWS) anomalies from a baseline, representing the sum of all water stored on and within the Earth's landmasses, including surface water, soil moisture, snow, ice, and groundwater. GRACE data is retrieved at a 1-degree spatial resolution and a monthly time step. The GLDAS dataset provides surface water components such as soil moisture (across layers: 0-10 cm, 10-40 cm, 40-100 cm, 100-200 cm), snow water equivalent (SWE), and canopy water storage, at a 0.25-degree resolution. Using these datasets, I calculate the groundwater storage anomaly (GWSA) within the region of interest using the water balance approach: (TALK ABOUT THE UP/DOWN SAMPLING I'VE DONE)

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

# Results 
## Machine Learning Framework
In this analysis, I apply three machine learning (ML) models - Linear Regression, Ridge Regression and Bayesian Ridge Regression - to predict month-to-month changes in groundwater levels in the region of interest. The feature set I use includes GRACE-derived groundwater storage changes (month-to-month), groundwater anomalies (GWA), Sentinel-1 mean backscatter values (S1_mean_VH and S1_mean_VV). Additionally, temporal features, sine and cosine transformations of the month, were used to model systematic seasonal cycles independent of specific climate anomalies. 

Given the small dataset, comprising only 12 months of Sentinel-1, GRACE and GLDAS data, careful validation is essential to avoid overfitting. I implement Leave-one-out Cross Validation which trains the model on 11 months and tests on the remaining month for each iteration.

The Linear Regression model achieved promising predictive performance, with a cross-validation R² of 0.822, successfully explaining 82% of the variance in groundwater storage and achieving a mean absolute error of 17.5 ± 13cm. The training R² of 0.968 indicates strong pattern learning with moderate overfitting which was as expected given the sample-feature ratio. Ridge Regression and Bayesian Ridge Regression achieve CV R² of 0.586 and 0.674 respectively.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Model%20Comparison.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Best%20Model%20Performance.png" alt="Second image" width="400"></td>
  </tr>
</table>


A strong correlation (r = 0.888) between GRACE-derived regional storage and in-situ well measurements underscores the model’s ability to capture cross-scale aquifer dynamics. 





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



