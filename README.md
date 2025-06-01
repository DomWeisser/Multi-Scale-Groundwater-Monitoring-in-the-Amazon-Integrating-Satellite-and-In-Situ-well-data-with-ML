# Predicting Monthly Groundwater Storage Changes in the Amazon Basin: A Multi-Sensor Machine Learning Approach Using GRACE, Sentinel-1, and Ground-Based Observations

# Video
[![Watch the video](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/0c936f26e678378f229980a624af0feb58a3a8c7/Images/Screenshot%202025-06-01%20132145.png)](https://www.loom.com/share/7f9294ddcbfd45899d6dc94f38188156?sid=343d850f-b44b-4907-a934-b000f32b8516)

**Click the Picture to take you to my video explaining my Analysis**

# Project Overview
Despite the Amazon region’s vast surface water resources, over two-thirds of its urban population rely on groundwater from aquifers for domestic and industrial needs [1]. One of Brazil’s most important freshwater reserves is the Alter do Chão Aquifer (ACA), which stretches beneath parts of the Amazonas, Pará, and Amapá states. However, the monthly patterns and overall behavior of groundwater storage in this aquifer remain poorly understood, especially the link between satellite-based water-storage data and local well-level measurements. Recent extreme droughts (notably in 2016 and 2023) have underscored the urgent need for better insight into these dynamics.
This project presents a proof-of-concept machine learning (ML) framework designed to predict month-on-month changes in groundwater levels of wells using satellite observations. The goal is to explore the relationship between GRACE-derived regional water storage estimates (with a coarse spatial resolution of 1° x 1°) and localised well data, supported by Sentinel-1 radar backscatter and seasonal trend analysis.

Due to the remoteness of the study area, only four wells provided monthly groundwater change data for 2019. The sparse sparial coverage made interpolation across the region unreliable, so I created point-based estimates by averaging these results. Combining GRACE satellite-derived total water storage with GLDAS land surface model components enabled the extraction of regional groundwater changes while I averaged Sentinel-1 radar backscatter (VH and VV polarisations) over the region to capture surface conditions related to vegetation and soil moisture dynamics. While this represents a significant spatial simplification given the Amazon's heterogeneity, I deemed it suitable for establishing proof of concept relationships between satellite observations and ground-based measurements in this densely forested environment.

Feature importance analysis revealed that GRACE storage change dominated model performance, confirming the strong physical relationship between satellite gravity measurements and ground-based observations. Sentinel-1 VH backscatter emerged as the second most important feature, indicating that radar-derived vegetation/surface water signals provides complementary information despite the regional averaging. Seasonal patterns showed strong linear relationships but lower functional importance, while GRACE groundwater anomalies provided moderate additional predictive value. 

Despite the data limitations, the best model (Linear Regression) performed well, achieveing a R² of 0.82 during cross-validation. I also trained Ridge and Bayesian ridge models to provide uncertainty quantification and improve regularisation for this limited dataset. While constrained by limited spatial coverage and only a single year of observations, the results demonstrate the strong potential for satellite-based groundwater monitoring in data-scarse tropical regions and the potential for improvement with expanded well networks and multi-year time series.

![image_alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/277608a94581a59c7f54984659af80fe76af55fb/Images/WhatsApp%20Image%202025-05-31%20at%2022.51.42.jpeg)

# Study Area
My analysis focuses on a ~ 280km x 220km area around Manaus, Brazil (60.8°W–58.2°W, 3.7°S–1.7°S), chosen for its hydrogeological diversity and the opportunity it offers to integrate satellite data with limited well-based observations. This region includes the high-porosity sandy formations typical of the Alter do Chão Aquifer and experiences distinct wet (November–April) and dry (May–October) seasons, making it well-suited for studying seasonal groundwater behavior [2]. Outside of Manaus, ground monitoring infrastructure is extremely limited, highlighting the importance of remote sensing and reliable uncertainty quantification to track groundwater trends.

# Datasets

- GRACE (Gravity Recovery and Climate Experiment): GRACE is a satellite mission that measures tiny changes in Earth’s gravity field caused by shifts in mass, primarily due to the movement of water. The mission uses a pair of satellites that orbit Earth about 220 kilometers apart. As they follow one another, they constantly send microwave signals back and forth to precisely measure the distance between them. When the lead satellite passes over an area with slightly stronger gravity - typically due to a greater concentration of mass such as water or ice - it is pulled slightly ahead, changing the distance between the two satellites. The trailing satellite experiences the same pull as it crosses the anomaly, allowing the system to detect minute variations in Earth's gravitational field. Together, these instruments isolate gravity-related changes and create monthly maps of Earth’s gravity field. These maps reveal how mass, especially water in forms such as groundwater, soil moisture, surface water, and snow, change over time. 

- GLDAS (Global Land Data Assimilation System): GLDAS is a land surface modeling system that uses satellite and ground-based observations to simulate surface water storage components like soil moisture, canopy water, and snow. When paired with GRACE, GLDAS helps separate the groundwater component by subtracting surface contributions, allowing us to estimate how much water is stored underground.

- Sentinel-1: Sentinel-1 is a satellite mission equipped with synthetic aperture radar (SAR), which operates in all weather conditions and can penetrate cloud cover and vegetation. It uses dual-polarization radar (VV and VH) to detect surface characteristics such as roughness and moisture content.

- In-situ Wells: Groundwater level measurements are obtained from the RIMAS monitoring network, which tracks aquifer behavior across Brazil. For this study, the dataset includes monthly groundwater level changes from four wells within the region of interest. These point-based measurements serve as the target variable for model training. [3]


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

The graph below shows the average **month-on-month change in groundwater levels** across the four wells within my study area (blue for wet season, yellow for dry season). It clearly reflects the regional hydrological cycle: groundwater levels rise significantly during the wet season, with a 1–2 month lag that leads to peak levels in June. Similarly, levels decline throughout the dry season, again with a lag, reaching their lowest point during December.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/bbd121844d62f620afb6f33064937ae6f5fddbd7/Images/well_analysis%20(2).png)


- **GRACE/GLDAS Processing:**

The GRACE dataset provides monthly Total Water Storage (TWS) anomalies from a baseline, which represent deviations from the long-term average of all water stored on and beneath the land surface. This includes groundwater, soil moisture, surface water, snow, and canopy water. To isolate groundwater storage, I use GLDAS, which provides modeled estimates of surface water components. These include soil moisture (at multiple depths: 0–10 cm, 10–40 cm, 40–100 cm, and 100–200 cm), snow water equivalent (SWE), and canopy water content. By subtracting these surface components from the GRACE TWS anomalies, I estimate the Groundwater Storage Anomalies (GWSA) using this formula:

**Groundwater Storage Anomaly (GWSA) = TWS - (Soil Moisture + Snow Water + Canopy Water)**

From the GWSA time series, I calculate the **month-on-month change in groundwater storage**.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/85ab06b473d1dcbc77a8e6e74b3934e96f996ee7/Images/grace_gwa_change_analysis_with_baseline%20(1).png)


- **Sentinel-1 Processing:**

For my analysis, I averaged Sentinel-1 backscatter (VV and VH polarisations) across the entire region of interst to create point-based values consistent with the well data and GRACE/GLDAS groundwater datasets. These averages are used to capture surface conditions that may relate to underlying groundwater dynamics. The graphs below show how these raw and normalised averages change throughout the year. 

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
In this analysis, I apply three machine learning models - Linear Regression, Ridge Regression and Bayesian Ridge Regression - to predict month-to-month changes in groundwater levels for my region of interest. The feature set I use includes month-to-month GRACE/GLDAS-derived groundwater storage changes, GRACE/GLDAS derived groundwater anomalies from a basline (GWA), Sentinel-1 mean backscatter values (S1_mean_VH and S1_mean_VV). Additionally, I used temporal features, sine and cosine transformations of the month, to model seasonal cycles. 

Given the small dataset, comprising only 12 months of Sentinel-1, GRACE and GLDAS data, careful validation is essential to avoid overfitting. I implement Leave-one-out Cross Validation which trains the model on 11 months and tests on the remaining month for each iteration.

The Linear Regression model achieved the best predictive performance, with a cross-validation R² of 0.822, successfully explaining 82% of the variance in monthly groundwater storage changes and achieving a mean absolute error of 17.5 ± 13cm. The training R² of 0.968 and MAE of 7.5cm indicates strong pattern learning with moderate overfitting due to the difference with the cross-validated results. This moderate overfitting is expected given the sample-feature ratio. Ridge Regression and Bayesian Ridge Regression achieve CV R² of 0.586 and 0.674 respectively, with MAE of 25cm for both respectively. This reduction in performance is due to regularisation effects that constrain coefficient magnitudes, though they provide valuable uncertainty quantification.

<table>
  <tr>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Model%20Comparison.png" alt="Monthly mean well depth" width="400"></td>
    <td><img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c2a5cfdc9e25688f0b20a8f92c36ba74cc72c651/Images/Best%20Model%20Performance.png" alt="Second image" width="400"></td>
  </tr>
</table>


Below is a table showing the lineear correlations between each feature and the target variable, this will be further studied by feature importance below. These correlations reflect the strong seasonal patterns observed in the preprocessing analysis. The high correlation between month_sin, which creates a smooth seasonal cycle within the feature, and groundwater changes (r=0.92) directly mirrors the seasonal cycles seen in both the GRACE/GLDAS-derived groundwater data and well measurements, which showed very similar month-to-month patterns throughout 2019. The low correlation with month_cos compared to month_sin indicates that groundwater changes follow the natural hydrological calendar of peak changes during March-April instead of the calendar year cycle. In contrast, the weaker correlations with Sentinel-1 features reflect the complex, regionally-averaged radar patterns that showed unexpected seasonal variations.


<img src="https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/c9158570697724e23fb444946eb5129f1940845c/Images/feature_correlation_with_wells_target.png?raw=true" alt="image_alt" width="500"/>


## **Uncertainty Analysis**

Understanding how confident we can be in each prediction is crucial, especially with a small dataset of only 12 months. To address this, I implemented two uncertainty quantification methods:

- Bayesian Ridge Regression provides internal model-based uncertainty, estimating prediction intervals of approximately ±22 cm based on the model's inherent parameter uncertainty.

- Bootstrap resampling (with 100 iterations) captures variability introduced by the data itself, producing wider intervals of ±36 cm that account for sampling uncertainty in the limited dataset.

Despite the dataset's limited size, results are promising. Bayesian predictions closely track actual well measurements throughout the year, successfully capturing seasonal dynamics from +80 cm during wet season peaks to -70 cm during dry season troughs. The model demonstrates particularly strong performance during critical seasonal transition months. However, the wider botstrap intervals reveal the impact of having only 12 months of data. The bootstrap method accounts for the additional uncertainty from the small sample size, not just the model internal uncertainty. This  shows that while our model works well with the available data, it would be more confident in predictions with a larger dataset. However, since all observed well measurements fall wihtin these wider bounds, they offer appropriate confidence levels for this analysis.

![image alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/bf609c386a4bc7c36ce68e652b5d52eadb8f9e11/Images/uncertainty_time_series%20(2).png)

## Feature Importance

Feature importance analysis helps highlight which variables matter most when predicting changes in groundwater storage, key for understanding what drives aquifer behavior. By showing which remote sensing signals and seasonal trends are most valuable, the analysis can also help focus future data collection efforts.

In this project, I used a couple methods to understand feature importance. **Permutation importance** measures how much model performance drops when the feature is randomly shuffled (breaking it's relationship with the target) and **coefficient magnitude** measures the raw linear weight each feature receives in the model equation after standardisation. 

Permutation feature importance reveal that GRACE groundwater storage change and Sentinel-1 VH backscatter are the most important features. Sentinel-1 VH backscatter shows this high permutation importance despite low correlation because it captures complex vegetation and surface water interactions not available in gravity measurements. When VH is removed, the model loses critical surface signals that help distringuish different types of groundwater changes, highlighting it's significant potential when used at it's original spatial resolution rather than averaged across the whole region of interst as required by my simplification. Despite month_sin having strong individual correlation and receviing large coefficient weights, it's permutation importance is minimal because GRACE groundwater storage change captures most of the same seasonal signal, making explicit temporal features redundant. GRACE groundwater anomaly (GWA) demonstrates the highest coefficient magnitude in Linear Regression, probably due to it's different data characteristics compared to GRACE groundwater storage change. While storage change ranges from -28 to 16cm with a SD of 16.3cm, GWA represents cumulative anomalies raning from -106 to -19cm with a SD of 30.8, requiring larger coefficients to achieve equivalent predictive impact. 

![image_alt](https://github.com/DomWeisser/Multi-Scale-Groundwater-Monitoring-in-the-Amazon-Integrating-Satellite-and-In-Situ-well-data-with-ML/blob/2cd60542b201ee6732e0c14fd8a27b5e54c16aa2/Images/comprehensive_feature_importance%20(1).png)


# Limitations and Future Work
My analysis applied several simplifications that create oppurtunities for significant future impovements in accuracy and granularity. Spatial averaging was the primary simplification, averaging across four wells to create point based values instead of interpolating across the whole region of interest and similarly averaging Sentinel-1 backscatter across the whole region to create equivalent point based values. Additionally, the 12  month dataset constrained model complexity and limits the understanding of inter-annual change and longer-term groundwater trends.

However, these simplifications reveal important insights that point towards furture research directions. The high permutation importance of Sentinel-1 VH backscatter despite regional averaging suggests that radar-derived vegetation and surface water singals contain valuable information about groundwater dynamics that will become even more powerful at higher resolution. Future work should look into expanding the monitoring network with additional wells distributed across the study region, enabling spatial interpolation and pixel-level analysis. This would unlock the full potential of 10-meter resolution Sentinel-1 data while still using insights from GRACE/GLDAS derived groundwater changes. The combination of ground data with satellite imagery could transform groundwater monitoring from regional averaging to more granular mapping that captures the true complexity of Amazon hydrology and supports managment strategies.


# Environmental Impacts

Running my project on Google Colab consumed approximately 0.5 kWh of energy. However, this excludes the significant energy costs of downloading all my datasets: downloading 12 months of GRACE satellite data (~6.2 MB total), Sentinel-1 radar imagery (~24 GB total), GLDAS datasets (~300 MB total), and well measurements (~3.3 MB total) likely consumed an additional 1-2 kWh for data transmission and cloud storage access, bringing the total direct computational footprint to over 2 kWh.
While the direct computational impact remains modest, the project relies on satellite infrastructure with enormous upstream environmental costs that are difficult to quantify but highly significant. The GRACE mission cost ~$500 million [4] and the total cost for the Copernicus missions from ESA cost ~€3.24 billion between 2014-2021 [5], reflecting significant embodied carbon across satellite manufacturing, launch operations and mission lifecycles. Each rocket launch can emit tons of CO₂, along with other compounds that contribute to stratopsheric ozone depletion, particularly from solid rocket motors, which have been shown to cause ozone loss orders of magnitude greater than liquid-fueled alternatives [6]. While my analysis maintains relatively low computational costs by using standard CPU processing and lightweight regression models, the total environmental foorptint extends beyond the compute cycles. 

Given this it is out duty as Environmental Data Scientists to make sure the positive impacts our research and analysis produces outweighs these enviornmental costs. With further development of my project the results will enable sustainable groundwater managment in the Amazon basin, support precision agriculture to reduce water waste, facilitate early drought detection and provide critical data for climate adaptation startegies in one of Earth's most important ecosystems. This project aims to connect communities most affected by climate change with cutting-edge technologies, recognising that these innovations are frequently not deployed to those who need them most







# References
[1]: https://www.sciencedirect.com/science/article/pii/S2352801X25000384?casa_token=Gf1ryYWVSZ4AAAAA:4xYYbn2YwsatJyetPnysCKJIT_g7G6vWr_DXk65WmILkhuCEi3o3XUv60Lez7jrpOwQPA2u9PA: Groundwater dynamics and hydrogeological processes in the Alter do Chão Aquifer: A case study in Manaus, Amazonas – Brazil


[2]: https://www.sciencedirect.com/science/article/pii/S0895981121004429?casa_token=pW97eOPB26AAAAAA:nFoOVws0NzNC7_ZcMXi1pFmNKiIb85eTcA3K2qo31xKTnkItB9SktC7ndkgilFK3p4zWjRoh4Q: Flow patterns and aquifer recharge controls under Amazon rainforest influence: The case of the Alter do Chão aquifer system


[3]: https://figshare.com/articles/code/RIMAS_model_Brazil_ipynb/22009562


[4]: https://spacepolicyonline.com/news/nasas-grace-fo-five-iridium-satellites-share-a-ride-to-space/#:~:text=At%20a%20pre-launch%20press%20conference%20yesterday%2C%20NASA%20and,77%20million%20Euros%20%28about%20%2490%20million%29%20for%20Germany.


[5]: https://www.space.com/copernicus-program


[6]: https://www.sciencedirect.com/science/article/abs/pii/S0959652620302560: The environmental impact of emissions from space launches: A comprehensive review

