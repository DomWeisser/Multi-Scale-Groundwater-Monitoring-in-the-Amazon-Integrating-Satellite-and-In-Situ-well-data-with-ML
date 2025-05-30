# Predicting Monthly Groundwater Storage Changes in the Amazon Basin: A Multi-Sensor Machine Learning Approach Using GRACE, Sentinel-1, and Ground-Based Observations

# Project Overview
Despite the Amazon region’s vast surface water resources, over two-thirds of its urban population rely on groundwater from aquifers for domestic and industrial needs [1]. One of Brazil’s most important freshwater reserves is the Alter do Chão Aquifer (ACA), which stretches beneath parts of the Amazonas, Pará, and Amapá states. However, the monthly patterns and overall behavior of groundwater storage in this aquifer remain poorly understood, especially the link between satellite-based water-storage data and local well-level measurements. Recent extreme droughts (notably in 2016 and 2023) have underscored the urgent need for better insight into these dynamics.
This project presents a proof-of-concept machine learning (ML) framework designed to predict month-on-month changes in groundwater levels of wells using satellite observations. The goal is to explore the relationship between GRACE-derived regional water storage estimates (with a coarse spatial resolution of 1° x 1°) and localised well data, supported by Sentinel-1 radar backscatter and seasonal trend analysis.

Due to the remoteness of the study area, only four wells provided monthly groundwater change data for 2019. The sparse sparial coverage made interpolation across the region unreliable, so I used the average of these readings as a regional proxy, a necessary simplification for this analysis. Combining GRACE satellite-derived total water storage with GLDAS land surface model components enabled extraction of regional groundwater changes while Sentinel-1 radar backscatter (VH and VV polarisations) was averaged over the region to capture surface conditions related to vegetation and soil moisture dynamics. While this represents a significant spatial simplification given the Amazon's heterogeneity, I deemed it suitable for establishing proof of concept relationships between satellite observations and ground-based measurements in this densely forested environment.

Feature importance analysis revealed that GRACE storage change dominated model performance (permutation importance: 9.35), confirming the strong physical relationship between satellite gravity measurements and ground-based observations. Sentinel-1 VH backscatter emerged as the second most important feature (7.41), indicating that radar-derived vegetation/surface water signals provide significant complementary information despite the regional averaging. Seasonal patterns showed strong linear relationships but lower functional importance, while GRACE groundwater anomalies provided moderate additional predictive value. The analysis demonstrated that direct GRACE measurements remain the primary driver, but multi-sensor fusion significantly enhances model performance.

Despite the data limitatinos, the best model (Linear Regression) performed well, achieveing a R² of 0.82 during cross-validation. I also trained Ridge and Bayesian ridge models to provide uncertainty quantification and improve regularisation for this limited dataset. While constrained by limited spatial coverage and only a single year of observations, the results demonstrate the strong potential for satellite-based groundwater monitoring in data-scarse tropical regions and the potential for improvement with expanded well networks and multi-year time series.

# Study Area
My analysis focuses on a ~ 280km x 220km area around Manaus, Brazil (60.8°W–58.2°W, 3.7°S–1.7°S), chosen for its hydrogeological diversity and the opportunity it offers to integrate satellite data with limited well-based observations. This region includes the high-porosity sandy formations typical of the Alter do Chão Aquifer and experiences distinct wet (November–April) and dry (May–October) seasons, making it well-suited for studying seasonal groundwater behavior [2]. Outside of Manaus, ground monitoring infrastructure is extremely limited, highlighting the importance of remote sensing and reliable uncertainty quantification to track groundwater trends.

# Datasets

- GRACE (Gravity Recovery and Climate Experiment): GRACE is a satellite mission that measures tiny changes in Earth’s gravity field caused by shifts in mass, primarily due to the movement of water. The mission uses a pair of satellites that orbit Earth about 220 kilometers apart. As they follow one another, they constantly send microwave signals back and forth to precisely measure the distance between them. When the lead satellite passes over an area with slightly stronger gravity - typically due to a greater concentration of mass such as water or ice - it is pulled slightly ahead, changing the distance between the two satellites. The trailing satellite experiences the same pull as it crosses the anomaly, allowing the system to detect minute variations in Earth's gravitational field. Each satellite is equipped with an accelerometer to account for non-gravitational forces like atmospheric drag, and GPS receivers to track their exact positions within a centimeter. Together, these instruments isolate gravity-related changes and create monthly maps of Earth’s gravity field. These maps reveal how mass, especially water in forms such as groundwater, soil moisture, surface water, and snow, change over time. This data is particularly valuable for monitoring water resources in regions lacking ground-based observations.

- GLDAS (Global Land Data Assimilation System): GLDAS is a land surface modeling system that uses satellite and ground-based observations to simulate surface water storage components like soil moisture, canopy water, and snow. When paired with GRACE, GLDAS helps separate the groundwater component by subtracting surface contributions, allowing us to estimate how much water is stored underground.

- Sentinel-1: Sentinel-1 is a satellite mission equipped with synthetic aperture radar (SAR), which operates in all weather conditions and can penetrate cloud cover and vegetation. It uses dual-polarization radar (VV and VH) to detect surface characteristics such as roughness and moisture content.

- In-situ Wells: Groundwater level measurements are obtained from the RIMAS monitoring network, which tracks aquifer behavior across Brazil. For this study, the dataset includes monthly groundwater level changes from four wells within the region of interest. These point-based measurements serve as the target variable for model training.


| Dataset | Source | Temporal Resolution Used | Spatial Resolution | Coverage | Format |
| --- | --- | --- | --- | --- | --- |
| GRACE-FO | NASA JPL | Monthly (2019)  | 1° x 1° | Global | NetCDF |
| GLDAS-2.1 Noah | NASA GSFC | Monthly (2019) | 0.25° x 0.25°  | Global | NetCDF |
| Well Measurements | RIMAS (Brazil) | Monthly (2019) | Point-based  | 4 wells in ROI | CSV |
| Sentinel-1 SAR | ESA Copernicus | Monthly (2019) | 10m  | Regional ROI | Geo TIFF |

# Methodology
## Data Processing Pipeline
- **Region of Interest Definition:**

The region of Interest is equivalent to the bounds from the Sentinel-1 images with a 0.02° buffer

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_5.00.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/220da82684a60d4085086cb1e3116bcc0eae959f/Images/roi_coverage_basemap_zoom_0.05.png" alt="Second image" width="400"></td>
  </tr>
</table>

- **Well Data Analysis:**

The graph below shows the average **month-on-month change in groundwater levels** across the four wells within my study area. It clearly reflects the regional hydrological cycle: groundwater levels rise significantly during the wet season, with a 1–2 month lag that leads to peak levels in June, weeks into the dry season. Similarly, levels decline throughout the dry season, again with a lag, reaching their lowest point in the end of November/start of December.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/bbd121844d62f620afb6f33064937ae6f5fddbd7/Images/well_analysis%20(2).png)


- **GRACE/GLDAS Processing:**

The GRACE dataset provides monthly Total Water Storage (TWS) anomalies from a baseline, which represent deviations from the long-term average of all water stored on and beneath the land surface. This includes groundwater, soil moisture, surface water, snow, and canopy water. To isolate groundwater storage, I use GLDAS, which provides modeled estimates of surface water components. These include soil moisture (at multiple depths: 0–10 cm, 10–40 cm, 40–100 cm, and 100–200 cm), snow water equivalent (SWE), and canopy water content. By subtracting these surface components from the GRACE TWS anomalies, I estimate the Groundwater Storage Anomalies (GWSA) using this formula:

Groundwater Storage Anomaly (GWSA) = TWS - (Soil Moisture + Snow Water + Canopy Water)

From the GWSA time series, I calculate the **month-on-month change in groundwater storage**, which serves as the primary target variable for prediction in this study.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/512c8090f7eb86eca8df747d132c71ff4b388a77/Images/grace_gwa_change_analysis.png)


- **Sentinel-1 Processing:**

For my analysis, I averaged Sentinel-1 backscatter (VV and VH polarisations) across the entire region of interst to create point-based values consistent with the well data and GRACE/GLDAS groundwater datasets. These regional averages are used to capture surface conditions that may relate to underlying groundwater dynamics. The graphs below show how these raw and normalised averages change throughout the year. 

Based on Amazon hydrology, I would expect high backscatter during the wet season due to increased surface mositure, standing water and lush vegetations, and lower backscatter during the dry season due to reduced surface moisture abd vegetation stress. When looking at the normalised graph complex seasonal patterns emerge that align with but also deviate from expected Amazon seasonality. Patterns matching expectations include the peak in January coninciding with wet season onset, the August minimum representing peak dry season conditions, and the October VH peak aligning with the dry-to-wet season transitions. Unexpected patterns include the pronounced July VV peak and the reduction in values in November-December.

These complex patterns reflect the realistic consequences of my spatial simplification assumption. Regional averaging across the entire Sentinel-1 coverage masks the Amazon's enormous spatial heterogeneity into a single aggregated signal. In future work, a more granular approach using original 10m Sentinel-1 resolution with spatial interpolation of groundwater data would likely reveal clearer localised seasonal patterns and improve the interpretability of radar-groundwater relationships.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_analysis.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/27b858c6a9443c39e3a530111d09b8fe353f4489/Images/sentinel1_normalized_analysis.png" alt="Second image" width="400"></td>
  </tr>
</table>

# Results 
## Machine Learning Framework
In this analysis, I apply three machine learning (ML) models - Linear Regression, Ridge Regression and Bayesian Ridge Regression - to predict month-to-month changes in groundwater levels in the region of interest. The feature set I use includes GRACE-derived groundwater storage changes (month-to-month), groundwater anomalies (GWA), Sentinel-1 mean backscatter values (S1_mean_VH and S1_mean_VV). Additionally, I used temporal features, sine and cosine transformations of the month, to model systematic seasonal cycles independent of specific climate anomalies. 

Given the small dataset, comprising only 12 months of Sentinel-1, GRACE and GLDAS data, careful validation is essential to avoid overfitting. I implement Leave-one-out Cross Validation which trains the model on 11 months and tests on the remaining month for each iteration.

The Linear Regression model achieved the best predictive performance, with a cross-validation R² of 0.822, successfully explaining 82% of the variance in groundwater storage and achieving a mean absolute error of 17.5 ± 13cm. The training R² of 0.968 indicates strong pattern learning with moderate overfitting due to the difference with the cross-validated results. This is expected given the sample-feature ratio. Ridge Regression and Bayesian Ridge Regression achieve CV R² of 0.586 and 0.674 respectively.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Model%20Comparison.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Best%20Model%20Performance.png" alt="Second image" width="400"></td>
  </tr>
</table>


Below is a table showing the correlations between each feature and the target variable, this will be further studied by feature importance below.


## **Uncertainty Analysis**

Understanding how confident we can be in each prediction is crucial, especially with a small dataset of only 12 months. To address this, I implemented two uncertainty quantification methods:

- Bayesian Ridge Regression provides internal model-based uncertainty, estimating prediction intervals of approximately ±23 cm.

- Bootstrap resampling (100 iterations) captures variability introduced by the data itself, producing wider intervals of ±36 cm.

Despite the dataset's limited size, results are promising. Bayesian predictions (red squares) closely track actual well measurements (blue circles) throughout the year, successfully capturing seasonal dynamics from +80 cm in the wet season to -70 cm in the dry season. However, the wider bootstrap intervals (green shading) highlight that small sample size introduces variability. The ~60% difference between the two interval widths provides a valuable insight: while the model performs well on available data, the more conservative bootstrap intervals offer a more reliable basis for real-world decision-making.

Importantly, the majority of observed values fall within both uncertainty bounds—especially during key seasonal transitions.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/997f4358d4da4851065c370c9f23a792dd439fc2/Images/uncertainty_time_series.png)

## Feature Importance

Feature importance analysis helps highlight which variables matter most when predicting changes in groundwater storage, key for understanding what drives aquifer behavior. This is especially useful in data-scarce regions like the Amazon, where long-term ground measurements are limited. By showing which remote sensing signals and seasonal trends are most valuable, the analysis can also help focus future data collection efforts.

In this project, I used several  methods to assess feature importance. Across the board, GRACE satellite data stood out as the most important predictor, confirming its strong link to groundwater changes. Sentinel-1 radar data, especially VH polarization, was also highly influential in some models, likely reflecting interactions between vegetation and surface water. Seasonal patterns showed moderate importance, especially the cosine signal tied to the annual water cycle. Regularization in Ridge made feature contributions more balanced, while the Bayesian model added uncertainty estimates to the rankings. Overall, the findings reinforce GRACE data as the backbone of groundwater monitoring, with radar inputs offering valuable backup—particularly useful when GRACE data isn’t available.





# Limitations and Uncertainties 
Several simplifying assumptions introduce uncertainty within my analysis. Averaging four wells obscured local-scale variability, Sentinel-1 backscatter is spatially averaged over a large region, and GRACE/GLDAS data assume regional uniformity across a coarse grid. Furthermore, while the results from the ML models are promising, it needs to be remembered only 12 months of data has been used which constrains model complexity and increases sensitivity to model assumptions. 


While scale mismatches between point-scale well measurements and coarse-resolution satellite data introduce some uncertainty, the strong correlation (r = 0.888) between the two suggests robust cross-scale consistency within this aquifer system. This project serves as a proof of concept, demonstrating the viability of satellite-driven groundwater prediction in data-sparse regions and laying the groundwork for more detailed, multi-year analysis in the future. Future work should include more well data spread across the region of interest, allowing for interpolation which allows for pixel level analysis with GRACE/GLDAS and Sentinel-1 imagery.

# Environmental Impacts

Running my project on Google Colab consumed approximately 0.5 kWh of energy. However, this excludes the significant data transfer energy costs: downloading 12 months of GRACE satellite data (~6.2 MB total), Sentinel-1 radar imagery (~24 GB total), GLDAS datasets (~300 MB total), and well measurements (~3.3 MB total) likely consumed an additional 1-2 kWh for data transmission and cloud storage access, bringing the total direct computational footprint to over 2 kWh.
While the direct computational impact remains modest, the project relies on satellite infrastructure with enormous upstream environmental costs that are difficult to quantify but highly significant. The GRACE mission cost ~$500 million and Sentinel-1 ~€400 million, representing substantial embodied carbon from manufacturing, launch operations (each rocket launch produces ~300-400 tons of CO₂), and ongoing mission operations. 
From a computational efficiency perspective, my analysis leverages standard CPU processing without GPU acceleration, uses vectorised NumPy operations for data processing, and employs lightweight linear regression models rather than deep learning approaches, keeping the computational overhead low. However, the project's environmental impact could be reduced through: migrating to renewable energy-powered cloud computing, implementing more efficient data compression and caching strategies, and developing lightweight model variants for real-time applications. 
Given this it is out duty as Environmental Data Scientists to make sure the positive impacts our research and analysis produces outweighs these enviornmental costs. With further development of my project the results will enable sustainable groundwater managment in the Amazon basin, support precision agriculture to reduce water waste, facilitate early drought detection and provide critical data for climate adaptation startegies in one of Earth's most important ecosystems. This project aims to connect communities most affected by climate change with cutting-edge technologies, recognising that these innovations are frequently not deployed to those who need them most







# References
[1]: https://www.sciencedirect.com/science/article/pii/S2352801X25000384?casa_token=Gf1ryYWVSZ4AAAAA:4xYYbn2YwsatJyetPnysCKJIT_g7G6vWr_DXk65WmILkhuCEi3o3XUv60Lez7jrpOwQPA2u9PA: Groundwater dynamics and hydrogeological processes in the Alter do Chão Aquifer: A case study in Manaus, Amazonas – Brazil
[2]: https://www.sciencedirect.com/science/article/pii/S0895981121004429?casa_token=pW97eOPB26AAAAAA:nFoOVws0NzNC7_ZcMXi1pFmNKiIb85eTcA3K2qo31xKTnkItB9SktC7ndkgilFK3p4zWjRoh4Q: Flow patterns and aquifer recharge controls under Amazon rainforest influence: The case of the Alter do Chão aquifer system


