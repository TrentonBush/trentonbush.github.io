---
title: "Wrangling 1.5TB of Data"
excerpt: "biggus windus"
date: 2020-09-15 00:00:00 -0000
---

# Retrieving and Processing 1.5TB of Data
## Cleaning
'Data Cleaning' is the step in which we seek to understand and mitigate the potential biases,confounding variables, noise, or outright data loss that can infiltrate our dataset at each step of the data collection process. I find it helpful to conceptualize data cleaning in terms of risk management.

When working with a new dataset, a pragmatic first question is: "how much risk can we tolerate?" This low-stakes portfolio project can tolerate lots of risk, but a commercial application will mandate engagement with stakeholders to assess their risk appetite.

Next, what sources of risk are present in this dataset, and how impactful are they? Data exploration is a powerful tool, but this question is best answered by domain experts and the folks who built the data collection process. Some risks are simply impossible to identify from the data, such as problems that did not occur in this particular sample but will effect future samples. 

Quality checks are dataset specific, but here are some examples from my previous commercial work with IoT timeseries data:
* global outliers (data out of range)
* local outliers (data does not fit autocorrelation pattern, such as spikes, flatlines, or step changes)
* non-stationarity via changepoint detection:
  * mean drift
  * step change in variance
  * correlation change vs similar sensors

This dataset comes from a national lab and has documented quality control processes, most of which I was familiar with from my own experience with weather sensor data. Given the reputable data source and the high risk tolerance of this project, I was content to delegate data cleaning to them.

I still made a few plots of random subsamples of NREL's cleaned data and found small errors, like an erroneous spike that must have slipped below their detector's decision threshold. No amount of cleaning can guarantee risk-free data (because not all risks are in the data), but I was comfortable with the quality level I observed for the purposes of a portfolio project.