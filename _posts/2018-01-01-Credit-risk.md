---
layout: post
title:  "Credit risk and hazard rate"
date:   2018-01-01 10:13:32 +0100
---

```python
## import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

# Credit risk

In this notebook we are going to speak about credit risk, what is credit risk, how to compute his main indicators.

Suppose that your society A loan some money to another society B, A face the risk that B go bankrupt and then become unable to reimburse all or just a part of the loan, B is in a default. This is the credit risk, loan money with some chances to not get reimburse totally or not at all.

To evaluate your Expected loss in a case of counterparty default you need to know:

1. Your exposition at default **EAD** this is the total amount of loan that is exposed to credit risk
2. You loss given default **LGD** that is the expected amount of your loan that you are probably not going to recover. It is expressed in percentage of the **EAD**
3. The probability of default of the counterparty, or the probability that he become unable to reimburse his debt or the probability that you do not get reimbursed this **PD** probability of default.

Then your expected loss due to a default of a counterparty is $EL = PD \times LGD \times EAD$

There are different way to model the default risk for PD and LGD estimation :
- A poisson process, very simple based on rate spread (real or historical probability).
- The merton model if you have the informations on the counterparty balance sheeet (risk- neutreal probability).

### HAZARD RATES OR DEFAULT INTENSITY

The hazard rate or default intensity $ \lambda (t) $ is the default probability at time $t$ with the condition of no default between time zero and time $t$.

The hazard rate between time $t$ and $ \Delta t $ is $ \Delta \lambda (t) $ conditional on no default between time zero and $t$.

So when we get conditionnal default probability other a period it is the hazard rate.

### GET THE PROBABILITY OF DEFAULT FROM THE HAZARD RATE.

Assume that $V(t)$ is the cumulative probability of a company surviving  from time zero to time $t$ and $V(t+ \Delta t)$ is the cumulative probability of a company surviving  from time zero to time $t + \Delta t$.

So that $1-V(t)$ and $1-V(t + \Delta t)$ are the cumulative default probabilities to time $t$ and time $ t + \Delta t $.

So the default probability from time $ t $ and time $ t + \Delta t $ condition on no default between time zero to $t$, is the hazard rate $ \Delta \lambda (t) $ :

$$
\frac{(1-V(t + \Delta t)) - (1-V(t))}{V(t)} =  \frac{1-V(t + \Delta t) - 1+V(t)}{V(t)} = \frac{V(t) - V(t + \Delta t)}{V(t)} = \lambda (t) \Delta t$$

$$ \frac{V(t) - V(t + \Delta t)}{\Delta t} = \lambda (t) V(t)$$

$$ \frac{V(t + \Delta t) - V(t)}{\Delta t} = - \lambda (t) V(t) $$

$$ \frac{dV(t)}{V(t)} = - \lambda (t) dt $$

Resolving that differential equation :

$$ \int_0^t \frac{1}{V(s)} dV(s) = \int_0^t - \lambda (s) ds = - \int_0^t \lambda (s) ds$$

$$ ln(V(t)) - ln(V(0)) = - \int_0^t \lambda (s) ds $$

$$ V(t) = e^{-\int_0^t \lambda (s) ds}$$

Then $Q(t)$ the probability of default by time $t$ is :

$$ Q(t) = 1 - V(t) = 1 - e^{-\int_0^t \lambda (s) ds}$$

or

$$ Q(t) = 1 - e^{-\overline{\lambda}(t)dt} $$

Where $ \overline{\lambda}(t) $ is the average hazard rate between time zero and time $t$.


### Probability of default (PD) knowing $ \lambda (t) $
Knowing the hazard rate over a periode, it is possible to deduce the probability of default over any periode.

**Suppose that the hazard rate per year is 2%.**

#### PD over one month of a counterparty

$$Q(T) = 1- e^{-0.02*(1/12)}$$


```python
print("The probabilty of default over one month time horizon : ", 1-np.exp(-0.02*1/12))
```

    The probabilty of default over one month time horizon :  0.0016652785490612887


#### PD over 10 months of a counterparty

$$Q(T) = 1- e^{-0.02*(10/12)}$$


```python
print("The probabilty of default on a 10 months time horizon : ", 1-np.exp(-0.02*10/12))
```

    The probabilty of default on a 10 months time horizon :  0.01652854617838251


#### PD over 20 years of a counterparty

$$Q(T) = 1- e^{-0.02*(20*1)}$$


```python
print("The probabilty of default on a 10 months time horizon : ", 1-np.exp(-0.02*20))
```

    The probabilty of default on a 10 months time horizon :  0.3296799539643607


### References

Risk Management and Financial Institutions, 4th Edition, John C. Hull, ISBN: 978-1-118-95594-9, Mar 2015, WILEY
