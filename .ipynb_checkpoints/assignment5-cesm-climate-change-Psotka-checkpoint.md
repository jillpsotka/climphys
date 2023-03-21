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

(nb:aclimchange)=
# Assignment: Climate change in the CESM simulations

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook) by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

+++

## Part 1

Following the examples in the [lecture notes](https://brian-rose.github.io/ClimateLaboratoryBook/courseware/transient-cesm.html), open the four CESM simulations (fully coupled and slab ocean versions). 

Calculate timeseries of **global mean ASR and OLR** and store each of these as a new variable. *Recall that ASR is called `FSNT` in the CESM output, and OLR is called `FLNT`.*

Plot a timeseries of **(ASR - OLR), the net downward energy flux at the top of the model**, along with a **12 month rolling mean**, analogous to the plot of global mean surface air temperature in the lecture notes.  

*Note that the rolling mean is important here because, just like with surface air temperature, there is a large seasonal cycle which makes it harder to see evidence of the climate change signal we wish to focus on.*

```{code-cell} ipython3
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
import xarray as xr
```

```{code-cell} ipython3
# import simulations
casenames = {'cpl_control': 'cpl_1850_f19',
             'cpl_CO2ramp': 'cpl_CO2ramp_f19',
             'som_control': 'som_1850_f19',
             'som_2xCO2':   'som_1850_2xCO2',
            }
# The path to the THREDDS server, should work from anywhere
basepath = 'http://thredds.atmos.albany.edu:8080/thredds/dodsC/CESMA/'
# For better performance if you can access the roselab_rit filesystem (e.g. from JupyterHub)
# basepath = '/roselab_rit/cesm_archive/'
casepaths = {}
for name in casenames:
    casepaths[name] = basepath + casenames[name] + '/concatenated/'
# make a dictionary of all the CAM atmosphere output
atm = {}
for name in casenames:
    path = casepaths[name] + casenames[name] + '.cam.h0.nc'
    print('Attempting to open the dataset ', path)
    atm[name] = xr.open_dataset(path, decode_times=False)
```

```{code-cell} ipython3
# calculate global means
gw = atm['som_control'].gw
OLR_timeseries = {}
ASR_timeseries = {}
for name in casenames:
    ASR_timeseries[name] = (atm[name]['FSNT']*gw).mean(dim=('lat','lon'))/gw.mean(dim='lat')
    OLR_timeseries[name] = (atm[name]['FLNT']*gw).mean(dim=('lat','lon'))/gw.mean(dim='lat') 
```

```{code-cell} ipython3
# plot ASR - OLR
fig, axes = plt.subplots(2,1,figsize=(10,8))
for name in casenames:
    if 'cpl' in name:
        ax = axes[0]
        ax.set_title('Fully coupled ocean')
    else:
        ax = axes[1]
        ax.set_title('Slab ocean')
    field = ASR_timeseries[name] - OLR_timeseries[name]
    field_running = field.rolling(time=12, center=True).mean()
    line = ax.plot(field.time / 365, 
                   field, 
                   label=name,
                   linewidth=0.75,
                   )
    ax.plot(field_running.time / 365, 
            field_running, 
            color=line[0].get_color(),
            linewidth=2,
           )
for ax in axes:
    ax.legend();
    ax.set_xlabel('Years')
    ax.set_ylabel('Net energy flux (Wm-2)')
    ax.grid();
    ax.set_xlim(0,100)
fig.suptitle('Global mean net downward energy flux at the top of the model', fontsize=14);
```

## Part 2

Calculate and show the **time-average ASR** and **time-average OLR** over the final 10 or 20 years of each simulation. Following the lecture notes, use the 20-year slice for the fully coupled simulations, and the 10-year slice for the slab ocean simulations.

```{code-cell} ipython3
# extract the last 10 years from the slab ocean control simulation
# and the last 20 years from the coupled control
nyears_slab = 10
nyears_cpl = 20
clim_slice_slab = slice(-(nyears_slab*12),None)
clim_slice_cpl = slice(-(nyears_cpl*12),None)

# extract the last 10 years from the slab ocean control simulation
OLR_slab = OLR_timeseries['som_control'].isel(time=clim_slice_slab).mean(dim='time')
ASR_slab = ASR_timeseries['som_control'].isel(time=clim_slice_slab).mean(dim='time')
# and the last 20 years from the coupled control
OLR_cpl = OLR_timeseries['cpl_control'].isel(time=clim_slice_cpl).mean(dim='time')
ASR_cpl = ASR_timeseries['cpl_control'].isel(time=clim_slice_cpl).mean(dim='time')
# extract the last 10 years from the slab 2xCO2 simulation
OLR_2x_slab = OLR_timeseries['som_2xCO2'].isel(time=clim_slice_slab).mean(dim='time')
ASR_2x_slab = ASR_timeseries['som_2xCO2'].isel(time=clim_slice_slab).mean(dim='time')
# extract the last 20 years from the coupled CO2 ramp simulation
OLR_2x_cpl = OLR_timeseries['cpl_CO2ramp'].isel(time=clim_slice_cpl).mean(dim='time')
ASR_2x_cpl = ASR_timeseries['cpl_CO2ramp'].isel(time=clim_slice_cpl).mean(dim='time')
```

```{code-cell} ipython3
print('ASR som control = ', ASR_slab.values, 'Wm-2')
print('ASR cpl control = ', ASR_cpl.values, 'Wm-2')
print('ASR som 2xCO2   = ', ASR_2x_slab.values, 'Wm-2')
print('ASR cpl CO2ramp = ', ASR_2x_cpl.values, 'Wm-2')
print('')
print('OLR som control = ', OLR_slab.values, 'Wm-2')
print('OLR cpl control = ', OLR_cpl.values, 'Wm-2')
print('OLR som 2xCO2   = ', OLR_2x_slab.values, 'Wm-2')
print('OLR cpl CO2ramp = ', OLR_2x_cpl.values, 'Wm-2')
```

## Part 3

Based on your plots and numerical results from Parts 1 and 2, answer these questions:

1. Are the two control simulations (fully coupled and slab ocean) near energy balance?
2. In the fully coupled CO2 ramp simulation, does the energy imbalance (ASR-OLR) increase or decrease with time? What is the imbalance at the end of the 80 year simulation?
3. Answer the same questions for the slab ocean abrupt 2xCO2 simulation.
4. Explain in words why the timeseries of ASR-OLR look very different in the fully coupled simulation (1%/year CO2 ramp) versus the slab ocean simulation (abrupt 2xCO2). *Think about both the different radiative forcings and the different ocean heat capacities.*

+++

1. Yes, the control simulations are near energy balance on average. ASR - OLR ~= 0
2. The energy imbalance is increasing with time in the fully coupled CO2 ramp simulation. At the end of the simulation, ASR is about 1 Wm-2 higher than OLR.
3. In the slab ocean abrupt simulation, there is a large energy imbalance at the beginning of the simulation but by the end it is in balance.
4. In the CO2 ramp simulation, the model does not have a chance to adjust itself to equilibrium because we are constantly adding CO2. Therefore the ASR keeps increasing but the OLR does not catch up since there is a lag in response time. Whereas in the slab simulation, the increase in CO2 is all at once so the model can adjust itself to equilibrium after ~20 years. Furthermore, the slab ocean helps the model reach equilirium faster since it has no mixing and is a simplified thin layer whereas the coupled ocean has mixing and takes longer to warm up in response to increased CO2.

+++

## Part 4

Does the global average ASR **increase** or **decrease** because of CO2-driven warming in the CESM? 

Would you describe this as a **positive** or **negative** feedback?

+++

The global average ASR increases because of CO2-driven warming. This is a positive feedback becaause heating leads to more heating.

+++

## Part 5

In the previous question you looked at the global average change in ASR. Now I want you to look at how different parts of the world contribute to this change.

**Make a map** of the **change in ASR** due to the CO2 forcing. Use the average over the last 20 years of the coupled CO2 ramp simulation, comparing against the average over the last 20 years of the control simulation.

```{code-cell} ipython3
import cartopy.crs as ccrs
def make_map(field):
    '''from lecture notes. input field should be a 2D xarray.DataArray on a lat/lon grid.
        Make a filled contour plot of the field, and a line plot of the zonal mean
    '''
    fig = plt.figure(figsize=(14,6))
    nrows = 10; ncols = 3
    mapax = plt.subplot2grid((nrows,ncols), (0,0), colspan=ncols-1, rowspan=nrows-1, projection=ccrs.Robinson())
    barax = plt.subplot2grid((nrows,ncols), (nrows-1,0), colspan=ncols-1)
    cx = mapax.contourf(field.lon, field.lat, field, transform=ccrs.PlateCarree())
    mapax.set_global(); mapax.coastlines();
    cb = plt.colorbar(cx, cax=barax, orientation='horizontal')
    cb.set_label('ASR_ramp - ASR_control (Wm-2)')
    plt.title('Change in ASR due to ramp forcing')
    return fig, (mapax, barax), cx
```

```{code-cell} ipython3
control_ASR = atm['cpl_control']['FSNT'].isel(time=clim_slice_cpl).mean(dim='time')
ramp_ASR = atm['cpl_CO2ramp']['FSNT'].isel(time=clim_slice_cpl).mean(dim='time')
difference = ramp_ASR - control_ASR
fig, axes, cx = make_map(difference)
```

## Part 6

Repeat part 5, but this time instead of the change in ASR, look at the just change in the **clear-sky** component of ASR. You can find this in the output field called `FSNTC`.

*The `FSNTC` field shows shortwave absorption in the absence of clouds, so the **change** in `FSNTC` shows how absorption and reflection of shortwave are affected by processes other than clouds.*

```{code-cell} ipython3
control_ASR_clear = atm['cpl_control']['FSNTC'].isel(time=clim_slice_cpl).mean(dim='time')
ramp_ASR_clear = atm['cpl_CO2ramp']['FSNTC'].isel(time=clim_slice_cpl).mean(dim='time')
difference = ramp_ASR_clear - control_ASR_clear
fig, axes, cx = make_map(difference)
```

## Part 7

Discussion:

- Do your two maps (change in ASR, change in clear-sky ASR) look the same? 
- Offer some ideas about why the clear-sky map looks the way it does.
- Comment on anything interesting, unusual or surprising you found in the maps.

+++

The two maps do not look the same; the total ASR map is a lot more smoothed whereas the clear-sky ASR map has more extreme values. I think this is because the total ASR map is affected by cloud albedo which changes due to CO2 more uniformly around the globe. The clear-sky ASR, however, is mostly looking at the change in surface albedo. I think clear-sky ASR increases so extremely near in the oceans near the poles due to melting sea ice causing albedo to lower. We can also see this due to melting snow on the Rocky mountain range and the Himalayas.

I am slightly confused about the drastic increase in ASR near the Eastern tip of Russia the dips down into the Pacific. This is present even in the total ASR map and I do not know what it could be from.

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
