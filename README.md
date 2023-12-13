# MeteoDataXL
MeteoDataXL is a macro-enabled Microsoft Excel workbook for hourly meteorological observations. It can download historical observations directly from selected web services (Frost & AgroMetBase in Norway, and Netatmo's Weather API worldwide). Advanced quality-checking, gap-filling, and plotting.

### Features:
- **Data sources**
    - You can manually copy-paste hourly meteorological data from any source into sheet *'Download'*, to use the quality-checking and gap-filling features. Remember to set the timee zone in (sheet 'Options'). However, MeteodataXL can download data directly from selected web services: 
  - '**Frost API**' (https://frost.met.no/), a web service operated by Norway's Meterological Institute (https://www.met.no/). This gives open access to historical observations for over 1300 meteo stations. MeteoDataXL uses Frost API methods 'sources', 'observations/availableTimeSeries' and 'observations', to download both metadata and houry measurements. MeteoDataXL accesses 27 parameters (incl. temperature, humidity, pressure, precipitation, wind, solar & longwave radiation, albedo, cloud cover, snow depth, and synop codes).
  - '**AgroMetBase**' (https://lmt.nibio.no/), a web service operated by the Norwegian Institute of Bioeconomy Research (NIBIO), specifically their meteorology section 'Agrometeorology Norway' (LMT, LandbruksMeteorologisk Tjeneste). This gives open access to historical observations for over 100 stations in Norway. MetetDataXL uses method 'agrometbase/showweatherdata'. MeteoDataXL accesses 17 parameters (incl. temperature (air & soil), humidity, pressure, precipitation, wind, solar & longwave radiation, albedo, cloud cover, snow depth). Please note that about half of LMT's weather stations are only operated during the growing season (1st April, 31sth Septemper, more info: https://lmt.nibio.no/information/2/).
  - **Netatmo**'s 'Weather API' web service (https://dev.netatmo.com/apidocumentation/weather). MeteoDataXL downloads only outdoor observations (temperature, humidity, pressure, wind and rain), from over 170 counties. The service requires you to open and login to a Natatmo user-account (i.e. Oauth2 authentication). MeteodataXL uses API methods 'getpublicdata' and 'getmeasure', to locate nearly stations by latutude & longitude, and download their hourly observations. Be aware that Netato sensors may have non-ideal locations that seriously affect the measurements, e.g. unshaded outdoor montiors are affected by sunlight on sunny days. To search for Netatmo stations, use the search string "*NEAR* \<latitude\>*N*, \<longitude\>*E*".


- **Downloading data (button *'1:Download'*)**
  - The yellow-coloured input cells *'Meteo stations'* and *'Parameters'* in worksheet *'Download'* are dynamic. For example, if you type "Near 59.9425N,10.7195E" or "Near Oslo kommune" in cell *'Meteo stations'*, it returns a list of the nearest stations for a given latitude & longitude, municipality, or for a given meteo station. This function depends on which API you have choosen; for example, Frost API can search for available parameters for a given station and/or dates, or search for available stations that measure a given parameter. Each yellow input cell has pop-up comments explain input syntax. 
  - Practically an unlimited number of observations can be downloaded from the web services, i.e. many years, many stations, and multiple parameters.
  - For Frost API, MeteoDataXL also downloads the quality flag for each individual hour value. These quality flags are numbered 1-6 in the column to the right of the downloaded value. Spreadsheet 'Download' therefore has two columns for each meteo parameter (value-column, and green-coloured quality-column).

- **Automated quality-checking (button *'2:Check'*)**
  - Vets data for sudden jumps in value, extremely high or low values, and static values (i.e. dead sensor). Uncertain values (exceeding the sigma threshold) are flagged in the green 'quality' column to the right of the observation value. The threshold sigma can be changed in sheet 'Options').
  - Also vets the data by comparing comparing observations with nearby stations that you have downloaded (e.g. if you have downloaded air temperature from two or more meteo stations). This generates two kinds of output:
    - (i) The 4 nearest stations are listed in a pop-up comment in row 3 of the green 'quality' column for each meteo station, with associated Pearson's correlation coefficient (*r*) and linear regression equation.
    - (ii) Uncertain values (exceeding the sigma threshold) are flagged in the green 'quality' column to the right of the observation value.
  - Special checks are conducted for specific meteo parameters:
    - If you have downloaded dry-bulb temperature, dew-point temperature, and relative humidity the same meteo station (i.e. parameter codes TA, UU and PO), then it checks relative humidity and dew-point for equivalence.
    - If you have downloaded solar radiation data (global, direct, or diffuse), then it checks for incorrect time-stamp, night-time calibration offset, and can optionally analyze for obscuring features such as trees or masts that shade the pyranometer. For comparison, additional columns clear-sky irradiance (REST2-model) are output.
    - If you have downloaded cumulative precipitation observations (code RA), then it calculates a new column of hourly precipitation (code RR), properly accounting for jumps due to sensor-emptying and gradual evaporation. Likewise, if you have downloaded precipitation last hour (code RR), then  then it calculates a new column of cumulative precipitation (code RR), starting 1.january each year.
  - Between automated quality-checking and automated gap-filling, it is highy recommended to manually check the data. This can be done two ways:
    - Check-out the green 'quality' column for each parameter. This can be speeded up with hotkey "CTRL+N" to find the next outlier in the sheet. The first time you key CTRL+N finds the most extreme value in the sheet. The second time you press CTRL+N finds the second-most extreme value, etc.
    - View the data in the plotting sheets (see points below). Sheet 'Comparative plot' is especially useful for this purpose. 

- **Plotting (sheets *'Comparative plot'* and *'Carpet plot'*)**
  - **Sheet 'Comparative plot'** plots the time-series (x-axis = time, y-valus=value) for any chosen weather station/parameter and period (day, week, month, year). If sheet *'Download'* contains data from multiple weather stations, then this plot superimposes the plots for the 4 nearest stations, and uncertain values are flagged (black dot for interpolated data, circle for values exceeding sigma threshold).
  - **Sheet 'Carpet plot'** visualizes a carpet plot for any chosen weather station/parameter and period. x-axis is day, y-axis is hour of day, colour scale is value. Missing values are indicated.

- **Automated gap-filling (button *'3:FillGaps'*)**
  - MeteoDataXL has highly developed and tested gap-filling features. It chooses the best possible gap-filling method depending on the type of meteo parameter, gap length, and available data (e.g. whether you have downloaded data from nearly stations, or whether you have downloaded hourly data for multiple years for the same station. The following approach is used:
  - **Priority 1**: If available, priority is given to high quality measurements of other parameters at the same station, e.g. calculating relative himidity if both dew-point and dry-bulb temperature are measured at the same station.
  - **Priority 2**: If you have downloaded data from multiple meteo stations, then observations are transposed from from best-fit nearby stations (kriging by regression). A weighted average of up to 4 nearly stations is used (the optimum number of nearbly stations is automatically chosen to minimise error). Furthermore, the regression period is adjusted depending on the gap length: Shorter gaps have a shorter regression training period, which accounts for changes in weather regime (e.g. temperature inversion), and avoids issues of sensor recalibration, relocation of the neighbour.
  - **Priority 3**: If any gap has no simultaneous data from nearby stations, then the gap is filled by interpolation using a method similar to Kalman filter, which faithfully reproduces the diurnal profile. The interpolation is anchored to the observations before and after the gap in such a way to minimize error. 
  - **Solar radiation** data is given special treatment: 
    - (i) In addition to gap-filling, you are asked of wish to correct solar radiation measurements for systematic offset (zero-point calibration) by automatically analyzing the night-time offset. This offset is either a systematic error (incorrect logger voltage/grounding) or caused by clear-sky longwave radiation. Normally nighttime offset due to cold clear-sky radiation is negative, between -2 and -7 W/m², and can be ignored as it does not affect the daytime measurements. However, a positive nighttime offset is a sign of systematic error that probably also affects daytime readings, and can therfore be corrected if you wish.
    - (ii) When gap-filling solar radiation with no nearby stations, interpolatiion is based on cloud cover (not the traditional *clearness index*), with an adjustable traning period before and after the gap (small gaps have short training period). This approach has been adapted to work with both global and diffuse (shaded pyranometer) measurements.
  - **User-functions**: A few handy user-defined worksheet functions are available for in-cell calculations of psychometry or solar position. These are self-documented in the spreadsheet (Click on the 'fx' button in the Formula bar, and view the *User Functions* category.):
    - **usr_SolarPos**() Calculates solar position (*OutputOption* controls the output parameter). Arguments: *date_*: Exact date/time in local time zone; *lati*: Latitude (°N); *longi*: Longitude (°E); *OutputOption*: 1=day of year, 2=declination, 3=elevation, 4=SIN(elevation), 5=azimuth, 6=hour angle (Negative OutputOption gives description)
    - **Patm_Pa**()  Calculates annual mean atmospheric pressure [Pa] at a given altitude. Arguments: *Height_over_sea_level_m*
    - **Tatm_K**()  Calculates annual mean dry-bulb air temperature [Kelvin] a given altitude [m] and latitude [°]. Arguments: *Height_over_sea_level_m*: Altitude above sea level [m]; *lat_deg*: Latitude [°]; *Mnth*: (Optional) month number (Optional, omitting gives annual mean)
    - **Tdew1_air**()  Calculates dew-point temperature [Kelvin] as a function of vapour pressure. Arguments: *Pw_Pa*: Vapour pressure [Pa]; *ice*: (Optional) TRUE if over ice. Default s FALSE for most Meteo calculations
    - **Tdew2_air**()  Calculates dew-point temperature [Kelvin] as a function of humidity ratio. Arguments: *Patm_Pa*: Atmospheric pressure [Pa]; *HumidRatio_kgkg*: Humidity ratio [kg water vapour / kg dry air]
    - **Tdew3_air**()  Calculates dew-point temperature [Kelvin] as a function of relative humidity.  Arguments: *Ta_K*: Dry-bulb air temperature [Kelvin]; *RH*: Relative humidity [0<RH<1]; *ice*: (Optional) TRUE if over ice. Default s FALSE for most Meteo calculations
    - **Pws**()  Calculates saturation vapour pressure [Pa].  Arguments: *Ta_K*: Dry-bulb air temperature; *ice*: (Optional) TRUE if over ice. Default s FALSE for most Meteo calculations
    - **RH1_air**()  Calculates relative humidity [0<RH<1] given dew-point temperature. Arguments: *Tair_K*: Dry-bulb air temperature [Kelvin]; *Tdew_K*: Dew-point air temperature [Kelvin]; *ice*: (Optional) TRUE if over ice. Default s FALSE for most Meteo calculations
    - **RH2_air**()  Calculates relative humidity [0<RH<1] given humidity ratio.  Arguments: *Tair_K*: Dry-bulb air temperature [Kelvin]; *Patm_Pa*: Atmospheric pressure [Pa]; *HumidRatio_kgkg*: Humidity ratio [kg vapour / kg dry air]
    - **HumidRatio1_air**()  Calculates humidity ratio [kg vapour / kg dry air] given relative humidity. Arguments: *Tair_K*: Dry-bulb air temperature [Kelvin]; *Patm_Pa*: Atmospheric pressure [Pa]; *RH*: Relative humidity [0<RH<1]
    - **HumidRatio2_air**()  Calculates humidity ratio [kg vapour / kg dry air] given dew-point temperature. Arguments: *Tdew_K*: Dew-point air temperature [Kelvin]; *Patm_Pa*: Atmospheric pressure [Pa]; *ice*: (Optional) TRUE if over ice. Default s FALSE for most Meteo calculations

### Licences:
- The MeteoDataXL workbook and source code is free closed-source (Lisence CC BY-ND 4.0): https://creativecommons.org/licenses/by-nd/4.0/
- Data from Frost API can have different licenses depending on source. Most data is licensed under 'Norwegian license for public data' (NLOD) and Creative Commons 4.0 BY International: http://creativecommons.org/licenses/by/4.0/. Terms of use: https://frost.met.no/termsofuse2.html
- Data from NIBIO Landbruksmeteorologisk tjeneste (LMT) AgroMetBase is lisenced under 'Norwegian license for public data' (NLOD): http://data.norge.no/nlod/. Please cite data according to LMT's requirements: https://lmt.nibio.no/information/3/
- Publically available data fron Netatmo API is lisenced: https://dev.netatmo.com/legal

### Author:
MeteoDataXL written by Peter G. Schild, Oslo Metropitan University & SINTEF Community in the period 2008-2021.
Contact the author if you wish to adapt the code to connect to other APIs.

### BibTeX for citing EpXL

```bibtex
@Misc{Schild23,
  author = {Schild, Peter G.},
  title = {{MeteoDataXL}: {Software tool to download and process hourly meteorological data}},
  howpublished = {\url{https://github.com/SchildCode/MeteoDataXL/}},
  year = {2023}
}
```
