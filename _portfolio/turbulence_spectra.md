---
title: "test"
excerpt: "blah blah"
date: 2020-09-23 00:00:00 -0000
---

# For Wind Power, Existing Turbulence Metrics Are Sufficient

## Time-of-Generation Matters
The proliferation of cheap solar PV is causing strong seasonal and daily cycles in power prices around the world.  Rival sources of generation must assess their exposure to these low and even negative prices, or risk financial underperformance. Thanks to the relative predictability of the seasonal/diurnal cycles of solar power, risk-conscious developers can assess their exposure by analyzing the temporal patterns in their own production.

Until recently, the wind power industry largely discarded temporal production information in favor of simpler annual totals. Despite the recent interest in time-aware modelling for estimating future production, the industry still relies on measurements and methodologies designed for low-fidelity annual models. This presents many opportunities for improvement. One such opportunity is to better measure turbulence, an important parameter in the conversion of wind speed to electrical power.

In short, turbulence refers to the temporal frequency of changes in wind speed. This effects power generation because wind turbines must be actively tuned to extract maximum power from incoming wind, but turbines are large machines with limited reaction times. "High turbulence" refers to changes in wind speed that occur faster than the turbine can react. This generally results in lower power production than would be produced at the same average wind speed with lower turbulence (slower changes).

This project seeks to understand the seasonal and diurnal cycles, if any, 
[are these different projects? one about solar covariance and one about excess power vs TI?]

## Turbulence is Poorly Measured
In my opinion, outdated measurement standards and a single poorly designed metric make accounting for turbulence unecessarily difficult (and inaccurate).

Wind data is produced by sampling about a dozen sensors, every second or two, for one to three years. Thirty years ago, when these measurement systems were designed and standardized, collecting 1 Hz data for a year (31.5 million records) was likely cost prohibitive (a conservative estimate: 12 columns * 31.5M rows per year * 32 bit values = 1.5 GB per year). Instead of keeping that raw 1 Hz data, the data logger downsamples it to 10-minute aggregates, recording a mean, standard deviation, minimum, and maximum from each column for each 10-minute period. Because of this downsampling, turbulence is now defined as wind speed changes that occur more frequently than once per 10-minutes. But the relevant cutoff frequency - turbine reaction speed - is more like 30 seconds. This is too slow by a factor of 20! Furthermore, information loss could be mitigated with the right aggregate metrics, but a standard deviation discards all temporal information! So there is no record of the actual frequency content of the wind. It is likely too late to change the industry standard 10-minute time interval, but it would be relatively easy to add a new aggregate metric.

A good turbulence metric should be designed with the end use in mind - estimating power production. Electrical energy is proportional to wind speed cubed, and relevant wind speeds are those up to the cutoff frequency. By taking the cumulative integral of the cubed frequency spectrum, we can measure the available power up to any arbitrary cutoff frequency. [Those familiar with signal processing may be reminded of the power spectral density - this is very similar but with cubed rather than squared spectral components, followed by cumulative integration.] As a generation model, this essentially replaces high frequency power with the (lower) power of the average speed. I think that is a reasonable assumption, but it is plausible that actual performance would be slightly less due to aerodynamic inefficiency. This would really need to be validated with actual production data, but alas, that is all proprietary.

frequency Lower frequency changes are directly recorded in measured data, while higher frequencies are aggregated into a coarse summary statistic: the standard deviation. The arbitrary cutoff point is determined In the 30 years since these measurement standards were adopted, increasing computing power/storage and our more detailed understanding of wind turbines means there may be value in information above the traditional cutoff frequency.



Price forecasting is notoriously uncertain, but the structural trend of solar PV deployment seems durable and credible enough to plan around. 