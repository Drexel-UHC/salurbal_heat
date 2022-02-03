
# Computing daily mean temperature from ERA5-Land for 371 Latin American Cities from 1996-2015

This page desribes the process of computing daily mean temperature from the ERA5-Land dataset for 371 Latin American Cities from 1996-2015 as part of the [SALURBAL](https://drexel.edu/lac/salurbal/overview/) project. ERA5-Land is a global, publicly available temperature reanalysis dataset that combines meteorological observations from stations with model data to produce hourly temperature at 2 meters above land surface. ERA5-Land data neglects pixels that have more than 50% water. However, many cities across the world are situated next to the ocean. Since we were losing information for those coastal cities, our team interpolated data from ERA5 (at a 31km x 31km resolution) and imputed it at the ERA5-Land 9km x 9km resolution, filling the gaps from those "missing" pixels.

One of SALURBAL's objectives is investigating temperature as exposure in relation to various health outcomes, and after imputation we weighted the temperature pixels by population to better approximate population exposure. Specifically, we used the 2010 estimates of the spatial distribution of population from WorldPop (100m x 100m). For the cities in Panama and Peru we weighted temperature by the 2010 Global Urban Footprint dataset because population data is not as accurate for Panama and Peru. After population-weighting, we computed mean daily temperature over 1996-2015 for different types of SALURBAL geographies -- cities (AD), sub-cities (L2), and cities' urban extent (UX). 

### Access to raw data:
- [ERA5 hourly data on single levels](https://cds.climate.copernicus.eu/cdsapp#!/dataset/reanalysis-era5-single-levels?tab=overview)
- [WorldPop data](https://www.worldpop.org/project/categories?id=3)
- [Global Urban Footprint data](https://drive.google.com/drive/folders/1_NM6c_SDAqb0LAOXt8LpbTT7eIL3HgAY)

### Access to imputed data:
- [ERA5Land imputed data](https://drive.google.com/drive/u/1/folders/1lpLFuolGD9iz7jNLnbh9kyjODtkg9Yk-)

### Access to final data:
- [L1 AD and UX data](https://drive.google.com/drive/u/1/folders/15eW8UumtN8Q-9b_q9fxbJGkVQA4kRHHn)
- [L2 data](https://drive.google.com/drive/folders/1VMb7JpvLAMVuypfpp7uk_FQebS3qQvHU?usp=sharing)

### Python script for computing population-weighted mean daily temperature for SALURBAL cities:
- [Python code](https://github.com/Drexel-UHC/salurbal_heat/blob/master/scripts/ERA_land_fill__final_version_vZonalStats.py)

---

## Data Imputation

To impute the missing values, we built the following model for each day and geography (L1AD) with missing ERA5Land pixels: 
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=ERA5land=f(X)%2B\epsilon">
</p>

where:
- <img src="https://render.githubusercontent.com/render/math?math=$X$"> is a vector including resampled ERA5 temperature from 31 km resolution to 9 km resolution with cubic resampling, absolute elevation (9 km resolution), relative elevation (elevation difference of a 9x9 km pixel and its surroundings), and aspect (9 km resolution);  
- <img src="https://render.githubusercontent.com/render/math?math=f(X)"> is a function that uses X to regress ERA5land temperature. Here we used random forest regression.  
- <img src="https://render.githubusercontent.com/render/math?math=\epsilon"> is the residual, or <img src="https://render.githubusercontent.com/render/math?math=ERA5land-f(X)">, which we further modeled with kriging spatial interpolation.  

We ran this model by each day and each geographic unit (L1AD) containing missing ERA5 missing pixels. In each geographic unit, we included all ERA5Land pixels and pixels within a 15-pixel buffer from the boundary to have enough samples to build the model above. To avoid overfitting, we used cross-validation to tune the parameters for both random forest regression and kriging spatial interpolation. Finally, we used the resulting model to impute the missing values.  

**Note**: In two cases we did not include kriging spatial interpolation in the imputation:
1. If adding kriging spatial interpolation led to worse model fit when compared with using random forest regression alone, in where we had both ERA5 and ERA5Land coverage;
2. If kriging spatial interpolation produced large values, which we seldomly found for the missing pixels. We decided the threshold to be 1 degree in absolute value. We chose this threshold as we observed the model residuals from using random forest alone were less than 1. Since kriging spatial interpolation was meant to further reduce these residuals, we considered kriging values greater than 1 to be anomalies and thus abandoned. 

## Calculting areal-level temperature 
Daily mean temperature at the city and sub-city levels from 1996 to 2015. We provide area-weighted averages for each spatial unit as well as averages further weighted by population, using 100m x 100m WorldPop data for 2010. Since population data is not accurate for Panama and Peru, for cities in these two countries we weight temperature by urban footprint data (Global Urban Footprint). 
For a spatial unit, its daily mean temperature is calculated as:
T_(i,d)=(∑_(t=1)^n▒T_(t,i,d) )/n
T_d=(∑_(i=1)^m▒〖w_i T〗_(i,d) )/(∑_(i=1)^m▒w_i )
Where T_(t,i,d) is the temperature of hour t in grid cell i on day d, T_(i,d) is the daily mean temperature of day d in grid cell i, T_d is the area-level weighted average of daily mean temperature on day d, w_i is a weighting factor, which equals to:
w_i=〖area〗_i                                                   for area weighted values,
w_i=〖area〗_i×〖population(GUF)〗_i          for area and population (GUF) weighted values
where 〖area〗_i is the area of the overlap between grid cell i and the spatial unit, 〖population(GUF)〗_i is the population or GUF urban footprint area of grid cell i



**Notes**:  
- The final tables of population-weighted mean daily temperature are [L1AD_UX_96_15.csv](https://drive.google.com/drive/u/1/folders/15eW8UumtN8Q-9b_q9fxbJGkVQA4kRHHn) and [L2_96_15.csv](https://drive.google.com/drive/u/1/folders/1VMb7JpvLAMVuypfpp7uk_FQebS3qQvHU). 
---

**Codebook for [L1AD and UX data](https://drive.google.com/drive/u/1/folders/15eW8UumtN8Q-9b_q9fxbJGkVQA4kRHHn):**  
- L1Name: City name
- Country: Country name
- date: year-month-day
- ADtemp_pw: Population weighted temperature mean at L1AD level (city-level)
- ADtemp_x: Unweighted temperature mean at L1AD level (city-level)
- UXtemp_pw:Population weighted temperature mean at L1UX level (urban extent) 
- UXtemp_x:  Unweighted temperature mean at L1UX level (urban extent)


Preview *L1AD_UX_96_15.csv*:  

<img src="scripts/L1_preview.PNG" align="center" width="60%">

**Codebook for [L2 data](https://drive.google.com/drive/folders/1VMb7JpvLAMVuypfpp7uk_FQebS3qQvHU?usp=sharing):**  
- L2Name: Sub-city name
- L2Type: Type of sub-city unit
- L1Name: City name
- Country: Country name
- date: year-month-day
- L2temp_pw: Population weighted temperature mean at L2 level (sub-city)
- L2temp_x: Unweighted temperature mean at L2 level (sub-city)


Preview *L2_96_15.csv*:  

<img src="scripts/L2_preview.PNG" align="center" width="60%">

**Contact:** 
- Yang Ju (yangju90@berkeley.edu)
- Irene Farah (irenef@berkeley.edu)
- Maryia Bakhtsiyarava (mariab@berkeley.edu)

