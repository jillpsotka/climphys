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

+++ {"user_expressions": []}

(resource:intake_esm_LENS)=
# Accessing CESM LENS (and LENS2) data with intake-esm

Preliminary:  You'll need to install `intake-esm`, `xarray-datatree`, and `s3fs` into 

```
mamba install -c conda-forge intake-esm
mamba install -c conda-forge xarray-datatree
mamba install -c conda-forge s3fs
```

This notebook demonstrates how to access Google Cloud CMIP6 data using intake-esm.

Intake-esm is a data cataloging utility built on top of intake, pandas, and
xarray. Intake-esm aims to facilitate:

- the discovery of earth’s climate and weather datasets.
- the ingestion of these datasets into xarray dataset containers.

It's basic usage is shown below. To begin, let's import `intake`:

```{code-cell} ipython3
import intake
import xarray as xr
import numpy as np
from matplotlib import pyplot as plt
from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta
import pandas as pd
import rioxarray
```

+++ {"user_expressions": []}

## Load the catalog - CESM LENS1

At import time, intake-esm plugin is available in intake’s registry as
`esm_datastore` and can be accessed with `intake.open_esm_datastore()` function.
Use the `intake_esm.tutorial.get_url()` method to access smaller subsetted catalogs for tutorial purposes.

```{code-cell} ipython3
import intake_esm
#url = intake_esm.tutorial.get_url('google_cmip6')
#print(url)
# If you want CESM LENS1:
url ="https://raw.githubusercontent.com/NCAR/cesm-lens-aws/main/intake-catalogs/aws-cesm1-le.json"

# If you want CESM LENS2:
#url = "https://raw.githubusercontent.com/NCAR/cesm2-le-aws/main/intake-catalogs/aws-cesm2-le.json"
```

```{code-cell} ipython3
cat.df
```

+++ {"user_expressions": []}

The first data asset listed in the catalog contains:

- the net longwave flux at the surface (variable_id='FLNS'), on daily timescales, from 1920-01-01 to 2005-12-31. 


## Finding unique entries

To get unique values for given columns in the catalog, intake-esm provides a
{py:meth}`~intake_esm.core.esm_datastore.unique` method:

Let's query the data catalog to see what models(`source_id`), experiments
(`experiment_id`) and temporal frequencies (`table_id`) are available.

```{code-cell} ipython3
unique = cat.unique()
unique
```

```{code-cell} ipython3
# Let's look at the different variables that are available:
unique['variable']
```

```{code-cell} ipython3
# If you don't know the shorthand for the variable you're interested in, we can look 
# at the "long_name" instead, which is a more descriptive version of the variable name:
unique['long_name']
```

```{code-cell} ipython3
# Let's look at the different experiments that are available:
unique['experiment']
```

+++ {"user_expressions": []}

## Search for specific datasets

The {py:meth}`~intake_esm.core.esm_datastore.search` method allows the user to
perform a query on a catalog using keyword arguments. The keyword argument names
must match column names in the catalog. The search method returns a
subset of the catalog with all the entries that match the provided query.

In the example below, we are are going to search for the following:

- long_name: `sea level pressure` which stands for
- experiments: ['CTRL','20C', 'HIST','RCP85']:
  - 20C: all forcing of the recent past (20th century).
  - RCP85: emission-driven RCP8.5.

```{code-cell} ipython3
cat_subset = cat.search(
    experiment=["CTRL","20C","HIST", "RCP85"],
    variable='WSPDSRFAV',
)

cat_subset
```

```{code-cell} ipython3
cat_subset.df
```

+++ {"user_expressions": []}

## Load datasets using `to_dataset_dict()`

Intake-esm implements convenience utilities for loading the query results into
higher level xarray datasets. The logic for merging/concatenating the query
results into higher level xarray datasets is provided in the input JSON file and
is available under `.aggregation_info` property of the catalog:

```{code-cell} ipython3
cat.esmcat.aggregation_control
```

+++ {"user_expressions": []}

To load data assets into xarray datasets, we need to use the
{py:meth}`~intake_esm.core.esm_datastore.to_dataset_dict` method. This method
returns a dictionary of aggregate xarray datasets as the name hints.

```{code-cell} ipython3
# For loading CESM LENS1:
dset_dict = cat_subset.to_dataset_dict(zarr_kwargs={"consolidated": True}, storage_options={"anon": True}) #"decode_times":True,"use_cftime":True
```

```{code-cell} ipython3
[key for key in dset_dict.keys()][:10]
```

+++ {"user_expressions": []}

We can access a particular dataset as follows:

```{code-cell} ipython3
ds = dset_dict["atm.20C.daily"]
ds
```

```{code-cell} ipython3
case = ds.WSPDSRFAV.sel(time=slice('1990-01-01', '1999-12-31'))

# weighting for avg
coslat = np.cos(np.deg2rad(case.lat))
weight = coslat / coslat.mean(dim='lat')
globavg = (case*weight).mean(dim=('lat','lon'))
```

```{code-cell} ipython3
case_monthly=case.resample(time='1MS').mean(dim='time')
```

```{code-cell} ipython3
case.sel(lat=slice(-90,0)).lat
```

```{code-cell} ipython3
coslat = np.cos(np.deg2rad(case.sel(lat=slice(0,90)).lat))
weight1 = coslat / coslat.mean(dim='lat')

#savg = (case.sel(lat=slice(-90,0))*weight1).mean(dim=('lat','lon')).resample(time='1MS').mean(dim='time').load()
navg = (case.sel(lat=slice(0,90))*weight1).mean(dim=('lat','lon')).resample(time='1MS').mean(dim='time').load()
```

```{code-cell} ipython3

```

```{code-cell} ipython3
s1 = savg.max(dim='member_id').load()
s2 = savg.min(dim='member_id').load()
n1 = navg.max(dim='member_id').load()
n2 = navg.min(dim='member_id').load()
```

```{code-cell} ipython3
#savg.to_netcdf('south_avg_mon.nc')
navg.to_netcdf('north_avg_mon.nc')
```

```{code-cell} ipython3
#globavg = globavg.resample(time='1MS').mean(dim='time').load()
```

```{code-cell} ipython3
a1 = globavg.max(dim='member_id').load()
a2 = globavg.min(dim='member_id').load()
```

```{code-cell} ipython3
#time_axis = np.arange(datetime(1990,1,1), datetime(1999,12,1),relativedelta(months=1)).astype(datetime)
time_axis = globavg.indexes['time'].to_datetimeindex()
fig, ax = plt.subplots(figsize=(12,6))
for ens in range(40):
    plt.plot(time_axis, globavg.isel(member_id=ens),c='k',linewidth=0.1)
plt.plot(time_axis, globavg.mean(dim='member_id'),c='r',linewidth=1)
plt.fill_between(time_axis, a1, a2, color = 'cyan', alpha = 0.4)
plt.ylabel('Avg surface wind speed (m/s)',fontsize=13)
plt.title('Global surface wind speed variability',fontsize=13)
plt.grid()
#plt.savefig('global_spread_90s.png')
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(12,6))
for ens in range(40):
    plt.plot(time_axis, navg.isel(member_id=ens),c='k',linewidth=0.1)
plt.plot(time_axis, navg.mean(dim='member_id'),c='r',linewidth=1)
plt.fill_between(time_axis, n1, n2, color = 'cyan', alpha = 0.4)
plt.ylabel('Avg surface wind speed (m/s)',fontsize=13)
plt.title('Northern surface wind speed variability',fontsize=13)
plt.grid()
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(12,6))
for ens in range(40):
    plt.plot(time_axis, savg.isel(member_id=ens),c='k',linewidth=0.1)
plt.plot(time_axis, savg.mean(dim='member_id'),c='r',linewidth=1)
plt.fill_between(time_axis, s1, s2, color = 'cyan', alpha = 0.4)
plt.ylabel('Avg surface wind speed (m/s)',fontsize=13)
plt.title('Southern surface wind speed variability',fontsize=13)
plt.grid()
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(12,6))
plt.plot(time_axis, navg.mean(dim='member_id'),c='b',linewidth=1,label='Northern H.')
plt.plot(time_axis, savg.mean(dim='member_id'),c='r',linewidth=1,label='Southern H.')
plt.plot(time_axis, globavg.mean(dim='member_id'),c='k',linewidth=1,label='Global')
#plt.fill_between(time_axis, s1, s2, color = 'cyan', alpha = 0.4)
plt.ylabel('Avg surface wind speed (m/s)',fontsize=13)
plt.title('Surface wind speed variability',fontsize=13)
plt.legend()
plt.grid()
```

look at ERA5:

```{code-cell} ipython3
xr = rioxarray.open_rasterio('C:/Users/jillp/Downloads/clim/10m_speed_90s.nc')
xr
```

```{code-cell} ipython3
time_axis = globavg.isel(time=slice(12,24)).indexes['time'].to_datetimeindex()
fig, ax = plt.subplots(figsize=(12,6))
for ens in range(40):
    plt.plot(time_axis, globavg.isel(member_id=ens,time=slice(12,24)),c='k',linewidth=0.15)

plt.fill_between(time_axis, a1.isel(time=slice(12,24)), a2.isel(time=slice(12,24)), color = 'cyan', alpha = 0.7)
plt.ylabel('Avg surface wind speed (m/s)',fontsize=13)
plt.title('Global surface wind speed variability',fontsize=13)
plt.grid()
```

```{code-cell} ipython3
globavg.to_netcdf('global_avg_mon.nc')
```

+++ {"user_expressions": []}

Let’s create a quick plot for a slice of the data for a subset of the ensemble members

```{code-cell} ipython3
# Plot the ensemble mean on top
ds.U.sel(lat = 50, lon=237, method = 'nearest').isel(lev=0).mean(dim='member_id').plot(color='k')
```

```{code-cell} ipython3
case_monthly
```

```{code-cell} ipython3
import cartopy.crs as ccrs
def make_map_dif(field,title):
    '''from lecture notes. input field should be a 2D xarray.DataArray on a lat/lon grid.
        Make a filled contour plot of the field
    '''
    fig = plt.figure(figsize=(14,6))
    nrows = 10; ncols = 3
    mapax = plt.subplot2grid((nrows,ncols), (0,0), colspan=ncols-1, rowspan=nrows-1, projection=ccrs.Robinson())
    barax = plt.subplot2grid((nrows,ncols), (nrows-1,0), colspan=ncols-1)
    cx = mapax.contourf(field.lon, field.lat, field, transform=ccrs.PlateCarree(),cmap='bwr',levels=[-11,-9,-7,-5,-3,-1,1,3,5,7,9,11])
    mapax.set_global(); mapax.coastlines();
    cb = plt.colorbar(cx, cax=barax, orientation='horizontal')
    cb.set_label('Avg surface wind speed (m/s)')
    cb.set_ticks([-11,-9,-7,-5,-3,-1,1,3,5,7,9,11])
    mapax.set_title('Wind speed difference '+title)
    return fig, (mapax, barax), cx

def make_map(field,title):
    '''from lecture notes. input field should be a 2D xarray.DataArray on a lat/lon grid.
        Make a filled contour plot of the field
    '''
    fig = plt.figure(figsize=(14,6))
    nrows = 10; ncols = 3
    mapax = plt.subplot2grid((nrows,ncols), (0,0), colspan=ncols-1, rowspan=nrows-1, projection=ccrs.Robinson())
    barax = plt.subplot2grid((nrows,ncols), (nrows-1,0), colspan=ncols-1)
    cx = mapax.contourf(field.lon, field.lat, field, transform=ccrs.PlateCarree())
    mapax.set_global(); mapax.coastlines();
    cb = plt.colorbar(cx, cax=barax, orientation='horizontal')
    cb.set_label('Wind speed difference (m/s)')
    mapax.set_title('Wind speeds '+title)
    return fig, (mapax, barax), cx
```

```{code-cell} ipython3
fig, axes, cx = make_map(case_monthly.isel(time=25,member_id=0),'Feb 1992')
```

```{code-cell} ipython3
fig, axes, cx = make_map(case_monthly.isel(time=30,member_id=0),'July 1992')
```

```{code-cell} ipython3
fig, axes, cx = make_map_dif(case_monthly.isel(time=27).mean(dim='member_id')-case_monthly.isel(time=30).mean(dim='member_id'),'Apr - July 1992')
```

```{code-cell} ipython3

```
