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

(nb:assignment-zero)=
# Assignment: Climate change in the zero-dimensional EBM

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook) by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

+++

## Learning goals

Students completing this assignment will gain the following skills and concepts:

- Familiarity with the Jupyter notebook
- Familiarity with the zero-dimensional Energy Balance Model
- Understanding of the adjustment toward equilibrium temperature
- Introduction to the concept of albedo feedback
- Use of numerical timestepping to find the equilibrium temperature
- Python programming skills: arrays, loops, and simple graphs

+++

## Instructions

- In a local copy of this notebook (on the JupyterHub or your own device) **add your answers in additional cells**.
- **Complete the required problems** below. 
- Some assignments have **optional bonus problems**. These are meant to be interesting and thought-provoking, but are not required. Extra credit will be given for interesting answers to the bonus problems.
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

## Problem 1: Time-dependent warming in the zero-dimensional Energy Balance Model

+++

In lecture we defined a zero-dimensional energy balance model for the global mean surface temperature $T_s$ as follows

$$ C  \frac{dT_s}{dt} = \text{ASR} - \text{OLR}$$

$$ \text{ASR} = (1-\alpha) Q $$

$$ \text{OLR} = \tau \sigma T_s^4$$

where we defined these terms:

- $C$ is a heat capacity for the atmosphere-ocean column
- $\alpha$ is the global mean planetary albedo
- $\sigma = 5.67 \times 10^{-8}$ W m$^{-2}$ K$^{-4}$ is the Stefan-Boltzmann constant
- $\tau$ is our transmissivity parameter for the atmosphere.
- $Q$ is the global-mean incoming solar radiation, or *insolation*.

+++

Refer back to our class notes for parameter values.

1. If the heat penetrated to twice as deep into the ocean, the value of $C$ would be twice as large. Would this affect the **equilibrium temperature**? Why or why not?
2. In class we used numerical timestepping to investigate a *hypothetical climate change scenario* in which $\tau$ decreases to 0.57 and $\alpha$ increases to 0.32. We produced a graph of $T_s(t)$ over a twenty year period, starting from an initial temperature of 288 K. Here you will repeat this calculate with a larger value of $C$ and compare the warming rates. Specifically:
    - Repeat our in-class time-stepping calculation with the same parameters we used before (including a heat capacity of $C = 4\times10^8$ J m$^{-2}$ K$^{-1}$), but extend it to 50 years. **You should create an array of temperatures with 51 elements, beginning from 288 K**.
    - Now do it again, but use $C = 8\times10^8$ J m$^{-2}$ K$^{-1}$ (representing 200 meters of water). You should **create another 51-element array** of temperatures also beginning from 288 K.
    - **Make a well-labeled graph** that compares the two temperatures over the 50-year period.
    
4. What do your results show about the role of heat capacity on climate change? **Give a short written answer.**

+++

1. The equilibrium temperature would not be affected because it is defined as the temperature at which OLR = ASR, neither of which depend on C. However, the energy balance itself would be affected because a larger C would lead to a smaller rate of change of temperature (dT/dt) for the same amount of net energy flux.

```{code-cell} ipython3
# 2.
import numpy as np
from matplotlib import pyplot as plt

# constants
sigma = 5.67e-8
dt = 60 * 60 * 24 * 365  # time step of 1 year (s)
alpha = 0.32  # albedo
tau = 0.57  # transmissivity
Q = 341.3  # insolation (W m-2)

# functions to get OLR and ASR
def OLR(T, tau):
    return tau*sigma*T**4

def ASR(alpha):
    return (1-alpha)*Q

def step_forward(T,C):  # take one time step forward
    return T + dt / C * ( ASR(alpha=0.32) - OLR(T, tau=0.57) )
```

```{code-cell} ipython3
# repeat in-class calculation
numsteps = 50
Tsteps = np.zeros(numsteps+1)
Years = np.zeros(numsteps+1)
Tsteps[0] = 288. 
for n in range(numsteps):
    Years[n+1] = n+1
    Tsteps[n+1] = step_forward( Tsteps[n], 4e8 )
#print(Tsteps)
#print(len(Tsteps))  # should be 51 elements
```

```{code-cell} ipython3
# repeat but with new C
Tsteps_new = np.zeros(numsteps+1)
Years = np.zeros(numsteps+1)
Tsteps_new[0] = 288. 
for n in range(numsteps):
    Years[n+1] = n+1
    Tsteps_new[n+1] = step_forward( Tsteps_new[n],8e8 )
#print(Tsteps)
#print(len(Tsteps))  # should be 51 elements
```

```{code-cell} ipython3
# plot the two temperature arrays
plt.plot(Years, Tsteps, label=r'C=4x10$^8$ J m$^{-2}$ K$^{-1}$')
plt.plot(Years, Tsteps_new, label=r'C=8x10$^8$ J m$^{-2}$ K$^{-1}$')
plt.xlabel('Years')
plt.ylabel('Global mean temperature (K)')
plt.legend()
plt.title('Comparing global temperature with varying C')
plt.show()
print('Equilibrium temperature is ~',round(Tsteps[-1],2), 'K')
```

3. This shows that, as expected, a larger heat capacity leads to a smaller rate of change of temperature in the global energy balance. This is because when C is 2x bigger this means that the atmosphere-ocean column takes twice as much energy to change by the same amount of temperature. So, if the energy in/out stays the same, a higher heat capacity will give a smaller change in temperature. The equilibrium temperature stays the same because energy in/out has not changed, but with a higher heat capacity it takes longer to reach it.

+++

## Problem 2: Albedo feedback in the Energy Balance Model

+++

For this exercise, we will introduce a new physical process into our model by **letting the planetary albedo depend on temperature**. The idea is that a warmer planet has less ice and snow at the surface, and thus a lower planetary albedo.

Represent the ice-albedo feedback through the following formula:

$$ \alpha(T) = \left\{ \begin{array}{ccc}
\alpha_i &   & T \le T_i \\
\alpha_o + (\alpha_i-\alpha_o) \frac{(T-T_o)^2}{(T_i-T_o)^2} &   & T_i < T < T_o \\
\alpha_o &   & T \ge T_o \end{array} \right\}$$

with the following parameter values:

- $\alpha_o = 0.289$ is the albedo of a warm, ice-free planet
- $\alpha_i = 0.7$ is the albedo of a very cold, completely ice-covered planet
- $T_o = 293$ K is the threshold temperature above which our model assumes the planet is ice-free
- $T_i = 260$ K is the threshold temperature below which our model assumes the planet is completely ice covered. 

For intermediate temperature, this formula gives a smooth variation in albedo with global mean temperature. It is tuned to reproduce the observed albedo $\alpha = 0.299$ for $T = 288$ K.

+++

1. 
    - Define a Python function that implements the above albedo formula. *There is definitely more than one way to do it. It doesn't matter how you do it as long as it works!*
    -  Use your function to calculate albedos for a wide range on planetary temperature (e.g. from $T=250$ K to $T=300$ K.)
    - Present your results (albedo as a function of global mean temperature, or $\alpha(T)$) in a nicely labeled graph.
    
2. Now investigate a climate change scenario with this new model:
    - Suppose that the transmissivity decreases from 0.611 to 0.57 (same as before)
    - Your task is to **calculate the new equilibrium temperature**. First, explain very briefly why you can't just solve for it analytically as we did when albedo was a fixed number.
    - Instead, you will use numerical time-stepping to find the equilibrium temperature
    - Repeat the procedure from Question 3 *(time-step forward for 50 years from an initial temperature of 288 K and make a graph of the results)*, but this time **use the function you defined above to compute the albedo for the current temperature**.
    - Is the **new equilibrium temperature larger or smaller** than it was in the model with fixed albedo? **Explain why in your own words.**

```{code-cell} ipython3
def albedo(T):
    # takes temperature (K) and returns calculated albedo
    if T <= 260:  # icy planet
        alpha = 0.7
    elif T >= 293:  # warm planet
        alpha = 0.289
    else:  # in-between
        alpha = 0.289 + (0.7-0.289)*(T-293)**2/(260-293)**2
    return alpha
```

```{code-cell} ipython3
# calculate albedo for range of T
T_range = np.linspace(250,301,50)
alpha_range = np.empty(T_range.shape)
for i in range(len(alpha_range)):
    alpha_range[i] = albedo(T_range[i])

#plot
plt.plot(T_range, alpha_range)
plt.xlabel('Temperature (K)')
plt.ylabel('Albedo')
plt.title('Albedo as a function of global mean temperature')
plt.show()
```

2. We cannot solve for equilibrium temperature analytically because we now have 2 unknowns (alpha and T) in the equilibrium temperature equation (ASR = OLR).

```{code-cell} ipython3
def step_forward(T,C=4e8,tau=0.57):
    alpha = albedo(T)
    return T + dt/C * (ASR(alpha) - OLR(T, tau=tau))
```

```{code-cell} ipython3
# now calculate T by stepping forward in time
numsteps = 50
Tsteps = np.zeros(numsteps+1)
Years = np.zeros(numsteps+1)
Tsteps[0] = 288. 
for n in range(numsteps):
    Years[n+1] = n+1
    Tsteps[n+1] = step_forward(Tsteps[n])
    
# plot results
plt.plot(Years, Tsteps)
plt.xlabel('Years')
plt.ylabel('Global mean temperature (K)')
plt.title('Global mean temperature with variable albedo')
plt.show()
```

```{code-cell} ipython3
print('The new equilibrium temperature is ~',round(Tsteps[-1],2), 'K')
```

The new equilibrium temperature is ~ 3 K higher than the fiexed-albedo model. This is because when we allow albedo to change with temperature, in this situation it decreases because the Earth is warming. A smaller albedo leads to more ASR because less radiation gets reflected. When one equates ASR = OLR to find equilibrium temperature, we see that this increase in ASR must come with an increase in OLR. Looking at the OLR equation, if transmissivity and sigma stay constant (which in this case they do), then the temperature must increase to lead to this increase in OLR.

+++

## Bonus problem

*Open-ended investigation for extra credit, not required*

Something very different occurs in this model if you introduce a strong negative radiative forcing, either by substantially reducing greenhouse gases (which we would represent as an increase in the transmissivity $\tau$), or by decreasing the incoming solar radiation $Q$.

Investigate, using your numerical model code, and report your results along with your thoughts.

```{code-cell} ipython3
# try doing the same calculation but increase tau 
numsteps = 70
taus = np.linspace(0.59,0.69,6)
for tau in taus:
    Tsteps = np.zeros(numsteps+1)
    Years = np.zeros(numsteps+1)
    Tsteps[0] = 288. 
    for n in range(numsteps):
        Years[n+1] = n+1
        Tsteps[n+1] = step_forward(Tsteps[n],tau=tau)
    plt.plot(Years, Tsteps,label='tau='+str(round(tau,2)))
    
# plot results
plt.xlabel('Years')
plt.ylabel('Global mean temperature (K)')
plt.title('Global mean temperature for different taus')
plt.legend()
plt.show()
```

When we increase transmissivity above 0.63, we can see that the equilibrium temperatures become much lower. This is an interesting model because we are allowing many factors to be variable: albedo, temperature, and transmissivity, therefore both ASR and OLR. I think what is happening is that the initial increase in tau causes cooling because of increased OLR. This cooling causes albedo to increase, leading to decreased ASR, which causes even more cooling! The model starts to stabilize itself once albedo reaches its icy planet threshold and can no longer keep feeding the temperature drop.

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
