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

(nb:rcm-feedback)=
# Assignment: Feedbacks in the Radiative-Convective Model

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook) by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

+++

## Learning goals

Students completing this assignment will gain the following skills and concepts:

- Familiarity with setting up and running a single-column Radiative-Convective Model using climlab
- Familiarity with plotting and interpreting vertical air temperature data on meteorological Skew-T charts
- Use of climlab to perform controlled parameter-sensitivity experiments
- Understanding of the lapse rate feedback concept
- Calculation of radiative forcing and climate feedback parameters

+++

## Question 1

Here you look at the effects of doubling CO$_2$ in the single-column Radiative-Convective model. 

*This exercise just repeats what we did in the lecture notes. You want to ensure that you can reproduce the same results before starting the next question, because you will need these results below.*

Following the lecture notes on climate sensitivity, do the following:

- set up a single-column radiative-convective model with specific humidity taken from the CESM control simulation
- Run this control model out to equilibrium
- Using a clone of the control model, calculate the stratosphere-adjusted radiative forcing $\Delta R$.
- Using another model clone, timestep the model out to equilibrium **with fixed specific humidity**
- Calculate the no-feedback Equilibrium Climate Sensitivity (ECS)
- Also calculate the no-feedback climate response parameter $\lambda_0$

Verify and show that you get the same results as we did in the lecture notes.

```{code-cell} ipython3
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
import xarray as xr
from metpy.plots import SkewT
import climlab

# from lab 14 skew T
#  This code is used just to create the skew-T plot of global, annual mean air temperature
ncep_url = "http://www.esrl.noaa.gov/psd/thredds/dodsC/Datasets/ncep.reanalysis.derived/"
ncep_air = xr.open_dataset( ncep_url + "pressure/air.mon.1981-2010.ltm.nc", use_cftime=True)
#  Take global, annual average 
coslat = np.cos(np.deg2rad(ncep_air.lat))
weight = coslat / coslat.mean(dim='lat')
Tglobal = (ncep_air.air * weight).mean(dim=('lat','lon','time'))

#  Resuable function to plot the temperature data on a Skew-T chart
def make_skewT():
    fig = plt.figure(figsize=(9, 9))
    skew = SkewT(fig, rotation=30)
    #skew.plot(Tglobal.level, Tglobal, color='black', linestyle='-', linewidth=2, label='Observations')
    skew.ax.set_ylim(1050, 10)
    skew.ax.set_xlim(-90, 45)
    # Add the relevant special lines
    skew.plot_dry_adiabats(linewidth=0.5)
    skew.plot_moist_adiabats(linewidth=0.5)
    #skew.plot_mixing_lines()
    skew.ax.legend()
    skew.ax.set_xlabel('Temperature (degC)', fontsize=14)
    skew.ax.set_ylabel('Pressure (hPa)', fontsize=14)
    return skew

#  and a function to add extra profiles to this chart
def add_profile(skew, model, linestyle='-', color=None):
    line = skew.plot(model.lev, model.Tatm - climlab.constants.tempCtoK,
             label=model.name, linewidth=2)[0]
    skew.plot(1000, model.Ts - climlab.constants.tempCtoK, 'o', 
              markersize=8, color=line.get_color())
    skew.ax.legend()
```

```{code-cell} ipython3
# Get the water vapor data from CESM output
cesm_data_path = "http://thredds.atmos.albany.edu:8080/thredds/dodsC/CESMA/"
atm_control = xr.open_dataset(cesm_data_path + "cpl_1850_f19/concatenated/cpl_1850_f19.cam.h0.nc")
# Take global, annual average of the specific humidity
weight_factor = atm_control.gw / atm_control.gw.mean(dim='lat')
Qglobal = (atm_control.Q * weight_factor).mean(dim=('lat','lon','time'))
```

```{code-cell} ipython3
#  Make a model on same vertical domain as the GCM
mystate = climlab.column_state(lev=Qglobal.lev, water_depth=2.5)
#  Build the radiation model
rad = climlab.radiation.RRTMG(name='Radiation',
                              state=mystate, 
                              specific_humidity=Qglobal.values,
                              timestep = climlab.constants.seconds_per_day,
                              albedo = 0.25,  # surface albedo, tuned to give reasonable ASR for reference cloud-free model
                             )
#  Build the convection model
conv = climlab.convection.ConvectiveAdjustment(name='Convection',
                                               state=mystate,
                                               adj_lapse_rate=6.5,
                                               timestep=rad.timestep,
                                              )
#  Coupling together the two components
rcm = climlab.couple([rad, conv], name='Radiative-Convective Model')
print(rcm.ASR - rcm.OLR)
```

```{code-cell} ipython3
# Run control model out to equilibrium
rcm.integrate_years(5)
print(rcm.ASR - rcm.OLR)  # if this is small then we have reached equilibruim
```

```{code-cell} ipython3
# Make a clone to calculate radiative forcing
rcm_2 = climlab.process_like(rcm)
rcm_2.name = '2xCO2'
rcm_2.subprocess['Radiation'].absorber_vmr['CO2'] *= 2
rcm_2.subprocess['Radiation'].absorber_vmr['CO2']
```

```{code-cell} ipython3
rcm_2.compute_diagnostics()
ASR_change = rcm_2.ASR - rcm.ASR
OLR_change = rcm_2.OLR - rcm.OLR
deltaR = ASR_change - OLR_change
print(deltaR, 'Wm-2 is the radiative forcing per doubling of CO2')

# now for stratosphere-adjusted radiative forcing
for i in range(1000):
    rcm_2.step_forward()
    rcm_2.Tatm[13:] = rcm.Tatm[13:]
    rcm_2.Ts[:] = rcm.Ts[:]
    
deltaR_strat = (rcm_2.ASR - rcm_2.OLR) - (rcm.ASR - rcm.OLR)
print(deltaR_strat, 'Wm-2 is the stratosphere-adjusted radiative forcing')
```

```{code-cell} ipython3
# Another clone for specific humidity ?? don't know why this is necessary since the original model does this
rcm_3 = climlab.process_like(rcm)
rcm_3.name = 'fixedq'

# step forward to equilibrium while keeping humidity fixed
while abs(rcm_3.ASR - rcm_3.OLR) > 0.01:
    rcm_3.step_forward()
    rcm_3.subprocess['Radiation'].specific_humidity[:] = Qglobal.values
```

```{code-cell} ipython3
# calculate ECS
rcm_2_eq = climlab.process_like(rcm_2)
rcm_2_eq.name = '2xCO2 strat at equilibrium'
rcm_2_eq.integrate_years(5)
ECS_nofeedback = rcm_2_eq.Ts - rcm.Ts
print(ECS_nofeedback, 'K is the ECS to doubling of CO2')
```

```{code-cell} ipython3
# calculate reponse parameter lambda
lambda_0 = deltaR_strat / ECS_nofeedback
print(lambda_0,'Wm-2K-1 is the no-feedback sensitivity parameter.')
```

## Question 2: combined lapse rate and water vapor feedback in the RCM

### Instructions

A typical, expected feature of global warming is that the **upper troposphere warms more than the surface**. (Later we will see that this does occur in the CESM simulations).

This feature is **not represented in our radiative-convective model**, which is forced to a single prescribed lapse rate due to our convective adjustment.

Here you will suppose that other physical processes modify this lapse rate as the climate warms. 

**Repeat the RCM global warming calculation, but implement two different feedbacks:**

- a water vapor feedback using **fixed relative humidity**
- a **lapse rate feedback** using this formula:

$$ \Gamma = \Gamma_{ref} - (0.3 \text{ km}) \Delta T_s $$

where $\Gamma_{ref}$ is the critical lapse rate you used in your control model, probably 6.5 K / km, and $\Delta T_s$ is the **current value of the surface warming relative to the control** in units of K. 

So, for example if the model has warmed by 1 K at the surface, then our parameterization says that the critical lapse rate should be 6.5 - 0.3 = 6.2 K / km.

Follow the example in the lecture notes where we implemented the fixed relative humidity. In addition to adjusting the `specific_humidity` at each timestep, you should also change the attribute

```
adj_lapse_rate
```
of the convection process at each timestep.

For example, if you have a model called `mymodel` that contains a `ConvectiveAdjustment` process called `Convection`:
```
mymodel.subprocess['Convection'].adj_lapse_rate = newvalue
```
where `newvalue` is a number in K / km.

### Specific questions:

1. Make a nice skew-T plot that shows three temperature profiles:
    - RCM control
    - RCM, equilibrium after doubling CO$_2$ without feedback
    - RCM, equilibrium after doubling CO$_2$ with combined water vapor and lapse rate feedback
2. Based on your plot, where in the column do you find the greatest warming?
3. Calculate the ECS of the new version of the model with combined water vapor and lapse rate feedback
4. Is this sensitivity larger or smaller than the "no feedback" ECS? Is it larger or smaller than the ECS with water vapor feedback alone (which we calculated in the lecture notes)?
5. Calculate the combined feedback parameter for (water vapor plus lapse rate).
6. Compare this result to the IPCC figure with feedback results from comprehensive models in our lecture notes (labeled "WV+LR"). Do you find a similar number?
7. Would you describe the **lapse rate feedback** as positive or negative?

```{code-cell} ipython3
#  actual specific humidity
q = rcm.subprocess['Radiation'].specific_humidity
#  saturation specific humidity (a function of temperature and pressure)
qsat = climlab.utils.thermo.qsat(rcm.Tatm, rcm.lev)
#  Relative humidity
rh = q/qsat
```

```{code-cell} ipython3
rcm_h2o = climlab.process_like(rcm)
rcm_h2o.name = 'with H2O and lapse rate feeback'
rcm_h2o.subprocess['Radiation'].absorber_vmr['CO2'] *= 2
for i in range(2000):
    qsat = climlab.utils.thermo.qsat(rcm_h2o.Tatm, rcm_h2o.lev)
    rcm_h2o.subprocess['Radiation'].specific_humidity[:] = rh * qsat
    new_lapse_rate = 6.5 - 0.3*(rcm_h2o.Ts - rcm.Ts)
    rcm_h2o.subprocess['Convection'].adj_lapse_rate = new_lapse_rate
    rcm_h2o.step_forward()
print(rcm_h2o.ASR - rcm_h2o.OLR)
```

```{code-cell} ipython3
skew = make_skewT()
add_profile(skew, rcm)
add_profile(skew, rcm_2)
add_profile(skew, rcm_h2o)
```

The greatest rate of warming in all models is through the stratosphere (>150 hPa). The greatest temperature difference between the H2O + lapse rate feedback model compared to the control is at the top of the troposphere (~200 hPa).

```{code-cell} ipython3
ECS_feedback = rcm_h2o.Ts - rcm.Ts
print(ECS_feedback, 'K is the climate sensitivity to doubling of CO2 with H2O and lapse rate feedbacks.')
```

This is larger than the no-feedback value (1.3 K) and smaller than the H2O feedback value (3 K).

```{code-cell} ipython3
# calculate reponse parameter lambda
lambda_f = deltaR_strat / ECS_feedback
print(lambda_f,'Wm-2K-1 is the H2O + lapse rate feedback sensitivity parameter.')
```

This is slightly higher than the literature values from IPCC which are around 1-1.5 Wm-2K-1.

The lapse rate feedback is negative because we can see that it diminished the sensitivity when combined with water vapour. This makes sense because the lapse rate feedback decreases the lapse rate with warming, and this will cause OLR to increase since the water vapour at the top of the troposphere that is emitting to space is warmer.

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
