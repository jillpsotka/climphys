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

(nb:simple-clouds)=
# Assignment: Clouds in the Leaky Greenhouse Model

This notebook is part of [The Climate Laboratory](https://brian-rose.github.io/ClimateLaboratoryBook) by [Brian E. J. Rose](http://www.atmos.albany.edu/facstaff/brose/index.html), University at Albany.

+++

## Learning goals

Students completing this assignment will gain the following skills and concepts:

- Continued practice working with the Jupyter notebook
- Familiarity with the toy "leaky greenhouse" model
- Conceptual understanding of the role of clouds in the planetary energy budget

+++

## Instructions

***This assignment requires some mathematics. You can present your work in this notebook, on hand-written paper, or a combination. Just make sure you communicate clearly which answers belong to which question.***

For answers presented in the notebook, follow the usual procedures to ensure that your code is well commented and runs clearly without errors (see previous assignment instructions).

+++

## Introduction

Consider the two-layer "leaky greenhouse" (or grey radiation) model from [these lecture notes](https://brian-rose.github.io/ClimateLaboratoryBook/courseware/elementary-greenhouse.html).

Here you will use this model to investigate the **radiative effects of clouds**.

Clouds simultaneously **reflect shortwave radiation** and **absorb longwave radiation**. These two effects often oppose each other in nature, and which one is stronger depends (among other things) on whether the clouds are **low** or **high** (i.e. in layer 0 or layer 1).

For this question we will suppose (as we did in the lecture notes) that there is **no absorption of shortwave radiation** in the atmosphere.

+++

## Question 1

Suppose a cloud reflects a fraction $\alpha_c$ of the shortwave beam incoming from above. $\alpha_c$ is a number between 0 and 1. Provide a coherent argument (in words, sketches, and/or equations) for why the **shortwave** effects of a cloud should alway be a **cooling** on the surface. Is this cooling effect different if the cloud is low or high? Explain.

+++

Shortwave effects of clouds are cooling because they reflect shortwave radiation from the sun back up to space. This prevents some of the radiation from reaching the surface, leading to cooling because there is less energy for the surface to absorb. This effect is only dependent on albedo and does not change if the cloud is low or high. This is because the atmosphere is transparent to shortwave radiation so if the albedos are the same then the same amount of shortwave radiation would reach the cloud and be reflected no matter where it is in the atmosphere. In reality, lower clouds tend to have higher albedo so will have a higher cooling effect.

+++

## Question 2

Because the liquid water droplets in a cloud are effective absorbers of longwave radiation, a cloud will **increase the longwave absorptivity / emissivity** of the layer in which it resides. 

We can represent this in the two-layer atmosphere by letting the absorptivity of a cloudy layer be $\epsilon + \epsilon_c$, where $\epsilon_c$ is an additional absorptivity due to the cloud. Derive a formula (i.e. an algebraic expression) for the OLR in terms of the temperatures $T_s, T_0, T_1$ and the emissivities $\epsilon, \epsilon_c$ for two different cases:

- a low cloud (the additional $\epsilon_c$ is in layer 0)
- a high cloud (the additional $\epsilon_c$ is in layer 1)

+++

Upwelling radiation from the surface to layer 0 stays the same in both cases:

$$U_0 = \sigma T_s^4$$

Layer 0 to layer 1 upwelling radiation is whatever is transmitted through layer 0 from the surface + emission from layer 0. For low cloud:

$$U_{1,low} = (1-\epsilon-\epsilon_c) \sigma T_s^4 + (\epsilon+\epsilon_c) \sigma T_0^4$$

and for high cloud:

$$U_{1,high} = (1-\epsilon) \sigma T_s^4 + \epsilon \sigma T_0^4$$

Layer 1 upwelling radiation is whatever is transmitted through from layer 0 + emission from layer 1. For low cloud:

$$U_{2,low} = (1-\epsilon) U_1 + \epsilon \sigma T_1^4$$

and for high cloud:

$$U_{2,high} = (1-\epsilon-\epsilon_c) U_1 + (\epsilon+\epsilon_c) \sigma T_1^4$$

OLR is just $U_2$. Subbing in the expressions for $U_1$ leads to:

$$OLR_{low} = (1-\epsilon)(1-\epsilon-\epsilon_c) \sigma T_s^4 + (\epsilon+\epsilon_c)(1-\epsilon)\sigma T_0^4 + \epsilon \sigma T_1^4$$

$$OLR_{high} = (1-\epsilon)(1-\epsilon-\epsilon_c) \sigma T_s^4 + \epsilon(1-\epsilon-\epsilon_c)\sigma T_0^4 + (\epsilon+\epsilon_c) \sigma T_1^4$$

Finally, rearranging a bit in order to see similarities in the second and third terms,

$$OLR_{low} = (1-\epsilon)(1-\epsilon-\epsilon_c) \sigma T_s^4 + (\epsilon-\epsilon^2-\epsilon\epsilon_c)\sigma T_0^4 + \epsilon_c\sigma
T_0^4 + \epsilon \sigma T_1^4$$

$$OLR_{high} = (1-\epsilon)(1-\epsilon-\epsilon_c) \sigma T_s^4 + (\epsilon-\epsilon^2-\epsilon\epsilon_c)\sigma T_0^4 + \epsilon_c\sigma T_1^4 + \epsilon \sigma T_1^4$$

We can see that the only difference lies in the third term of these final equations.

+++

## Question 3

Now use the tuned numerical values we used in class:

- $T_s = 288$ K
- $T_0 = 275$ K
- $T_1 = 230$ K
- $\epsilon = 0.586$

and take $\epsilon_c = 0.2$

(a) Repeat the following for both a high cloud and a low cloud:

- Calculate the **difference in OLR** due to the presence of the cloud, compared to the case with no cloud. 
- Does this represent a warming or cooling effect?

(b) Which one has a larger effect, the low cloud or the high cloud?

```{code-cell} ipython3
Ts = 288
T0 = 275
T1 = 230
epsilon = 0.586
epsilon_c = 0.2
sigma = 5.67e-8

OLR_nocloud = (1-epsilon)**2*sigma*Ts**4 + epsilon*(1-epsilon)*sigma*T0**4 + epsilon*sigma*T1**4
print('OLR with no clouds:',round(OLR_nocloud),'Wm-2')

OLR_low = (1-epsilon)*(1-epsilon-epsilon_c)*sigma*Ts**4 + (epsilon-epsilon**2-epsilon*epsilon_c)*sigma*T0**4 + epsilon_c*sigma*T0**4 + epsilon*sigma*T1**4
OLR_high = (1-epsilon)*(1-epsilon-epsilon_c)*sigma*Ts**4 + (epsilon-epsilon**2-epsilon*epsilon_c)*sigma*T0**4 + epsilon_c*sigma*T1**4 + epsilon*sigma*T1**4
print('OLR with low cloud:',round(OLR_low),'Wm-2')
print('OLR with high cloud:',round(OLR_high),'Wm-2')
```

So OLR is 6 Wm-2 less if there is low cloud and 39 Wm-2 less if there is high cloud. The decrease in OLR indicates warming in both cases because less radiation is being lost to space. There is a larger effect from the high cloud.

+++

## Question 4

Based on your results in questions 1-3, which do you think is more likely to produce a **net warming effect** on the climate: a low cloud or a high cloud? Give an explanation in words.

+++

High clouds are more likely to produce a net warming effect because the increased emissivity higher in the atmosphere leads to less OLR. If the clouds have the same albedo, they will decrease the ASR by the same amount. Furthermore, if high clouds have lower albedo (as they often do), then there would be less cooling so even more likely to have a stronger warming effect.

+++

## Question 5

How would your answer change if the atmosphere were **isothermal**, i.e. $T_s = T_0 = T_1$?

+++

If the atmosphere were isothermal, we can simplify the $OLR_{low}$ and $OLR_{high}$ equations above using $T_s=T_0=T_1=T$. We can see right away that $OLR_{low}=OLR_{high}$. Let's generalize it to $OLR_{cloud}$ and expand:

$OLR_{cloud} = (1-\epsilon-\epsilon_c-\epsilon+\epsilon^2+\epsilon\epsilon_c) \sigma T^4 + (\epsilon-\epsilon^2-\epsilon\epsilon_c+\epsilon_c+\epsilon)\sigma T^4$

$OLR_{cloud} = \sigma T^4$

This is just the blackbody emission, and it is the same answer that we would get from an isothermal atmosphere with no clouds. This shows that, in an isothermal atmosphere, clouds have no effect on OLR and neither does emissivity. This means that the presense of clouds would have a net cooling effect since they would still reflect shortwave radiation.

+++

## Question 6

Under what circumstances (structure of atmospheric temperature, i.e. relative values of $T_s$, $T_0$ and $T_1$) would you answer to question 4 be the opposite, and why?

+++

As seen in the $OLR_{low}$ and $OLR_{high}$ equations above, the effect of high vs low clouds on OLR differs only by the additive term $\epsilon_c\sigma T_0^4$ (for low clouds) vs $\epsilon_c\sigma T_1^4$ (for high clouds). Therefore the only way for low clouds to have a larger warming effect is if $T_0 < T_1$, as this will lead to low clouds having a smaller OLR than high clouds. The surface emission contributes the same to OLR whether there are high or low clouds, so $T_s$ does not affect this answer.

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
