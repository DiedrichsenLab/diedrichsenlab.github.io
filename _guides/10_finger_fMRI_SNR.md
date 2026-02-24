---
layout: page
title: Table of SNR for fMRI Experiments
author: Ali Ghavampour
date: 2024-11-19
category: fMRI
description: Signal quality estimated for some datasets from our lab.
---

When designing a new fMRI experiment, we need to select parameters such as the number of conditions, number of trials, spatial resolution, and more. These choices determine the quality of the fMRI signal. Below is a summary table showing **signal quality (~SNR)** in relation to **design parameters** from previous studies in our lab. This table provides insight into the signal quality you may get in your experiment.

## SNR for Finger Studies (M1 & S1):

| Study | #Conditions per Run | #Repetition per Condition | Trial Length | Field | Resolution | SNR ($$ V_s / V_{se}$$) | in-plane acc | multiband factor | TR | 
|----------------|-------------|---------|---------|---------|---------|---------|---------|---------|---------|
| Arbuckle 2020 <a href="https://www.jneurosci.org/content/40/48/9210" target="_blank">(link)</a> | 30 = 5-fingers * 2-directions * 3-forceLevels | 2 | 6s | 7T | 1.5mm | $$8.85\% \pm 1.05sem$$ | 2 | 2 | 1.5s |
| Ejaz 2015 <a href="https://www.nature.com/articles/nn.4038" target="_blank">(link)</a> | 31 chords | 1 | 13.5s | 3T | 2.3mm | $$3.26\% \pm 0.49sem$$ | - | - | 2.72s |
| Diedrichsen 2012 <a href="https://pmc.ncbi.nlm.nih.gov/articles/PMC3643717/" target="_blank">(link)</a> | 10 = 5-fingers * 2-hands | 3 | 8.16s | 3T | 2.3mm |  $$25.63\% \pm 4.09sem$$ | - | - | 2.72s |
| --------------|-------------|---------|---------|---------|---------|---------|---------|---------|---------|


## The design parameters:

* #Conditions per Run & #Repetition per Condition: For some designs, we want to fit as many task conditions as we can. But since a run can't take too long, more conditions means fewer number of trials per condition. For example, Ejaz 2015 with 31 conditions did 1 trial per condition. Diedrichsen 2012 had 10 conditions and could fit 3 trials. Now, compare the achieved SNR values between the two. 

* Trial Length:

* Resolution:

* Field: 


## How to estimate SNR ($$ V_s / V_{se}$$)?

Variance decomposition was used to esimate the signal qualities. For each subject, we get an $$N \times P$$ matrix after glm. N-regressors by P-voxels activity patterns. Let each run be denoted by $$y_i$$, where i is the run index. Each $$y_i$$ can then be modeled as:

$$
y_i = U + \epsilon_i
$$

where $$U$$ is the true activity pattern matrix and $\epsilon_i$ is the measurment noise. If we assume that: 1) $$\epsilon_i$$ is i.i.d., and 2) $$U$$ and $$\epsilon_i$$ are statistically independent, we can estimate:

* Within run variance as:
$$
\mathbb{E}[y_i {y_i}^T] = \mathbb{E}[U U^T] + \mathbb{E}[\epsilon_i {\epsilon_i}^T] = UU^T + \Sigma
$$

* Across runs co-variance as:
$$
\mathbb{E}[y_i {y_j}^T] = UU^T (i \neq j)
$$

Replace the $$\mathbb{E}[]$$ with mean over all possible pairings.

