---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.4
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

(nb:assignment-cesm-control)=
# Assignment: Global average budgets in the CESM pre-industrial control simulation

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook) by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

+++

## Learning goals

Students completing this assignment will gain the following skills and concepts:

- Continued practice working with the Jupyter notebook
- Familiarity with atmospheric output from the CESM simulation
- More complete comparison of the global energy budget in the CESM control simulation to the observations
- Validation of the annual cycle of surface temperature against observations
- Opportunity to formulate a hypothesis about these global temperature variations
- Python programming skills: basic xarray usage: opening gridded dataset and taking averages

+++

## Instructions

- In a local copy of this notebook (on the JupyterHub or your own device) **add your answers in additional cells**.
- **Complete the required problems** below. 
- Remember to set your cell types to `Markdown` for text, and `Code` for Python code!
- **Include comments** in your code to explain your method as necessary.
- Remember to actually answer the questions. **Written answers are required** (not just code and figures!)
- Submit your solutions in **a single Jupyter notebook** that contains your text, your code, and your figures.
- *Make sure that your notebook* ***runs cleanly without errors:***
    - Save your notebook
    - From the `Kernel` menu, select `Restart & Run All`
    - Did the notebook run from start to finish without error and produce the expected output?
    - If yes, save again and submit your notebook file
    - If no, fix the errors and try again.

+++

## Problem 1: The global energy budget in the CESM control simulation

Compute the **global, time average** of each of the following quantities, and compare them to the observed values from the Trenberth and Fasullo (2012) figure in the course notes:

- Solar Radiation budget:
    - Incoming Solar Radiation, or Insolation
    - Reflected Solar Radiation at the top of atmosphere
    - Solar Radiation Reflected by Surface
    - Solar Radiation Absorbed by Surface
    - Solar Radiation Refelected by Clouds and Atmosphere *(you can calculate this as the difference between the reflected radiation at the top of atmosphere and reflected radiation at the surface)*
    - Total Absorbed Solar Radiation (ASR) at the top of atmosphere
    - Solar Radiation Absorbed by Atmosphere *(you can calculate this as the residual of your budget, i.e. what's left over after accounting for all other absorption and reflection)*
- Longwave Radiation budget:
    - Outgoing Longwave Radiation
    - Upward emission from the surface
    - Downwelling radiation at the surface
- Other surface fluxes:
    - "Thermals", or *sensible heat flux*. *You will find this in the field called `SHFLX` in your dataset.*
    - "Evapotranspiration", or *latent heat flux*. *You will find this in the field called `LHFLX` in your dataset.*
    
*Note we will look more carefully at atmospheric absorption and emission processes later. You do not need to try to calculate terms such as "Emitted by Atmosphere" or "Atmospheric Window"*

**Based on your results above, answer the following questions:**

- Is the CESM control simulation at (or near) **energy balance**? 
- Do you think this simulation is near equilibrium?
- Summarize in your own words what you think are the most important similarities and differences of the global energy budgets in the CESM simulation and the observations.

```{code-cell} ipython3
import numpy as np
import matplotlib.pyplot as plt
import xarray as xr

cesm_data_path = "http://thredds.atmos.albany.edu:8080/thredds/dodsC/CESMA/"
atm_control = xr.open_dataset(cesm_data_path + "cpl_1850_f19/concatenated/cpl_1850_f19.cam.h0.nc")
weight_factor = atm_control.gw / atm_control.gw.mean(dim='lat')
```

```{code-cell} ipython3
dims = ["time","lat","lon"]
SOL_TOP_IN = (atm_control.SOLIN*weight_factor).mean(dim=dims)  # shortwave incoming
SOL_TOP_NET = (atm_control.FSNT*weight_factor).mean(dim=dims)  # ASR at top of atmosphere
SOL_TOP_OUT = SOL_TOP_IN - SOL_TOP_NET  # reflected shortwave at top of atmosphere

SOL_SUR_NET = (atm_control.FSNS*weight_factor).mean(dim=dims)  # shortwave absorbed at surface
SOL_SUR_IN = (atm_control.FSDS*weight_factor).mean(dim=dims)  # shortwave received at surface
SOL_SUR_OUT = SOL_SUR_IN - SOL_SUR_NET  # shortwave reflected at surface

SOL_ATM_REF = SOL_TOP_OUT - SOL_SUR_OUT  # shortwave reflected by clouds and atmosphere
SOL_ATM_ABS = SOL_TOP_NET - SOL_SUR_NET  # shortwave absorbed by clouds and atmosphere

LON_TOP_NET = (atm_control.FLNT*weight_factor).mean(dim=dims)  # OLR at top of atmosphere
LON_SUR_IN = (atm_control.FLDS*weight_factor).mean(dim=dims)  # longwave received at surface
LON_SUR_OUT = (atm_control.FLNS*weight_factor).mean(dim=dims) + LON_SUR_IN  # longwave emitted by surface

SHFLX = (atm_control.SHFLX*weight_factor).mean(dim=dims)
LHFLX = (atm_control.LHFLX*weight_factor).mean(dim=dims)
```

```{code-cell} ipython3

```

```{code-cell} ipython3
print("Incoming solar radiation:",str(SOL_TOP_IN.values-341.3),"difference")
print("ASR:",SOL_TOP_NET.values-(341.3-101.9),"difference")
print("Reflected solar radiation at top of atmosphere:",SOL_TOP_OUT.values-101.9,"difference")
print("Absorbed solar radiation at surface:",SOL_SUR_NET.values-161,"difference")
print("Reflected solar radiation at surface:",SOL_SUR_OUT.values-23,"difference")
print("Reflected solar radiation by clouds/atmos:",SOL_ATM_REF.values-79,"difference")
print("Absorbed solar radiation by clouds/atmos:",SOL_ATM_ABS.values-78,"difference \n")
print("OLR at top of atmosphere:",LON_TOP_NET.values-238.5,"difference")
print("Incoming longwave at surface:",LON_SUR_IN.values-333,"difference")
print("Upward emission from surface:",LON_SUR_OUT.values-396,"difference \n")
print("Sensible heat flux:",SHFLX.values-17,"difference")
print("Latent heat flux:",LHFLX.values-80,"difference")
```

```{code-cell} ipython3
print(LON_TOP_NET.values - SOL_TOP_NET.values)
print(341.3-(101.9+238.5))
```

The CESM control simulation is energy balanced and in equilibrium because OLR ~= ASR.

The biggest differences between the simulation and observations are that the simulation seems to 1) underestimate reflected shortwave radiation and 2) overestimate outgoing longwave radiation. We can see the first effect in that the reflected solar radiation by the surface and the clouds/atmos are both higher in observations, leading to a smaller observed ASR than simulated. The OLR is also smaller in observation than simulated, perhaps due to the atmosphere emitting more radiation in the simulation (we can see that the incoming longwave at the surface is also too high).

+++

## Problem 2: Verifying the annual cycle in global mean surface temperature against observations

In the class notes we plotted the **timeseries of global mean surface temperature** in the CESM control simulation, and found an **annual cycle**. The purpose of this exercise is to verify that this phenomenon is also found in the observed temperature record. If so, then we can conclude that it is a real feature of Earth's climate and not an artifact of the numerical model.

For observations, we will use the **NCEP Reanalysis data**.

*Reanalysis data is really a blend of observations and output from numerical weather prediction models. It represents our “best guess” at conditions over the whole globe, including regions where observations are very sparse.*

The necessary data are all served up over the internet. We will look at monthly climatologies averaged over the 30 year period 1981 - 2010.

You can browse the available data here: 
https://www.esrl.noaa.gov/psd/data/gridded/data.ncep.reanalysis.derived.html

**Surface air temperature** is contained in a file called `air.2m.mon.ltm.nc`, which is found in the collection called `Surface Fluxes`. 

Here's a link directly to the catalog page for this data file:
https://www.esrl.noaa.gov/psd/thredds/catalog/Datasets/ncep.reanalysis.derived/surface_gauss/catalog.html?dataset=Datasets/ncep.reanalysis.derived/surface_gauss/air.2m.day.ltm.nc

Now click on the `OPeNDAP` link. A page opens up with lots of information about the contents of the file. The `Data URL` is what we need to read the data into our Python session. For example, this code opens the file and displays a list of the variables it contains:

```{code-cell} ipython3
import xarray as xr
url = "http://www.esrl.noaa.gov/psd/thredds/dodsC/Datasets/ncep.reanalysis.derived/surface_gauss/air.2m.mon.ltm.nc"
ncep_air2m = xr.open_dataset(url)

print(ncep_air2m)
```

The temperature data is called `air`. Take a look at the details:

```{code-cell} ipython3
print(ncep_air2m.air)
```

Notice that the dimensions are `(time: 12, lat: 94, lon: 192)`. The time dimension is calendar months. But note that the lat/lon grid is not the same as our model output! 

*Think about how you will handle calculating the global average of these data.*

+++

### Your task:

- Make a well-labeled timeseries graph of the global-averaged observed average surface air temperature climatology. 
- Verify that the annual cycle we found in the CESM simulation also exists in the observations.
- In your own words, suggest a plausible physical explanation for why this annual cycle exists.

```{code-cell} ipython3
coslat = np.cos(np.deg2rad(ncep_air2m.lat))
ncep_weight_factor = coslat / coslat.mean(dim='lat')

global_avg = (ncep_air2m.air*ncep_weight_factor).mean(dim=['lat','lon'])

cesm_avg = (atm_control.TS*weight_factor).mean(dim=['lat','lon'])
monthly=cesm_avg.groupby(cesm_avg["time.month"])

plt.plot(np.arange(1,13),monthly.mean('time'))
plt.plot(np.arange(1,13),global_avg)
plt.xlabel('Month')
plt.ylabel('Temperature (K)')
plt.title('Avg global surface air temperature climatology')
```

The annual cycle is shown in both CESM and observations. The annual cycle shows that the average global surface temperature is highest in the Nothern hemisphere summer months. I think that this is the case because there is more land in the Northern hemisphere and land heats up/cools down more easily than water. So the Southern hemisphere likely sees less, on average, warming during their summer than the Northern hemisphere does. It is for this reason that the global average temperature would be highest during July/August and lowest during Northern hemisphere winter.

+++

____________

## Credits

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook), an open-source textbook developed and maintained by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

It is licensed for free and open consumption under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license.

Development of these notes and the [climlab software](https://github.com/brian-rose/climlab) is partially supported by the National Science Foundation under award AGS-1455071 to Brian Rose. Any opinions, findings, conclusions or recommendations expressed here are mine and do not necessarily reflect the views of the National Science Foundation.
____________

```{code-cell} ipython3

```
