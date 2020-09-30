---
title: "test"
excerpt: "blah blah"
date: 2020-09-23 00:00:00 -0000
---

# For Wind Power, Existing Turbulence Metrics Are Sufficient

I worked in the wind energy industry for several years, creating and running data-driven models to predict energy production of prospective projects. Over that time, the industry faced pressure from investors to improve its 20+ year old data practices in pursuit of more accurate predictions. Updating existing systems is always a bumpy process, and there remain many opportunities for improvement. The following is an experiment I ran to test a hypothesis that spectral information from high frequency data could improve prediction accuracy. It is written for an audience that understands data but has no domain knowledge in wind power. Thanks for reading!

## The Big Picture

When wind developers decide which prospective projects to build, a key piece of information is the estimated future production of that project. These estimates are largely based on a dataset of meteorological measurements taken at each project location. Due to historical precedent, the industry still relies on coarse aggregates of these measurements, and methodologies that were designed 20 to 30 years ago, in a different era of computational limits and aerodynamic understanding. This presents many opportunities for improvement. One such opportunity is to better measure turbulence, an important parameter in the conversion of wind speed to electrical power.

In short, turbulence refers to the temporal frequency of changes in wind speed. This effects power generation because wind turbines must be actively tuned to extract maximum power from incoming wind, but turbines are large machines with limited reaction times. "High turbulence" refers to changes in wind speed that occur faster than the turbine can react. This generally results in lower power production than would be produced with lower turbulence (slower changes), all else being equal.

## Turbulence is Poorly Measured
I hypothesized that outdated measurement standards and a single poorly designed metric make accounting for turbulence unecessarily difficult and noisy.

Wind data is produced by sampling about a dozen sensors, every second (1 Hz), for one to three years. Thirty years ago, when these measurement systems were designed and standardized, collecting 1 Hz data for a year was likely cost prohibitive (a quick estimate: 31.5 million seconds per year * 12 columns * 4 bytes per value = 1.5 GB per year. Trivial today, not so in the 1990s). To preserve storage space, data loggers are programmed to downsample the raw 1 Hz data to 10-minute aggregates, recording a mean, standard deviation, minimum, and maximum from each column for each 10-minute period.

Because of this downsampling, turbulence is now defined as wind speed changes that occur more frequently than once per 10-minutes. But the relevant cutoff frequency - turbine reaction speed - is more like 30 seconds. This is too slow by a factor of 20! Furthermore, information loss could be mitigated with the right aggregate metrics, but a standard deviation discards all temporal information. So there is no record of the actual frequency content of the wind! As a result, the systematic effect of turbulence is lost, instead converted to noise. It is likely too late to change the industry standard 10-minute time interval, but it would be relatively easy to add a new aggregate metric that captures most of this information.

## Experimental Design
The best way to test this hypothesis would be to compare accuracy of existing models to a frequency-aware model over several projects. Unfortunately, this requires proprietary turbine production data and high frequency wind data. There are several companies that could conduct such a study, but I don't have access to their data. Instead, I resort to asking a more limited question with public data: does a frequency-aware aggregate contain different information than existing aggregates? If the frequency-aware data can be accurately reconstructed from existing aggregates, then there is no additional value to be had. This experiment cannot say whether additional frequency information is useful, only whether it exists and how much variation there is. These results won't change the world either way, but the process demonstrates data skills, and what else do you want from a portfolio project??

A good turbulence metric should be designed with the end use in mind - estimating power production. Electrical energy is proportional to wind speed cubed, and relevant wind speeds are those up to the cutoff frequency. By taking the cumulative integral of the cubed frequency spectrum, we can measure the available power up to any arbitrary cutoff frequency. [Those familiar with signal processing may be reminded of the power spectral density - this is very similar but with cubed rather than squared spectral components, followed by cumulative integration.] As a generation model, this essentially replaces high frequency power with the (lower) power of the average speed. I think that is a reasonable assumption, but it is plausible that actual performance would be slightly less due to aerodynamic inefficiency. This would really need to be validated with actual production data, but alas, that is all proprietary.

frequency Lower frequency changes are directly recorded in measured data, while higher frequencies are aggregated into a coarse summary statistic: the standard deviation. The arbitrary cutoff point is determined In the 30 years since these measurement standards were adopted, increasing computing power/storage and our more detailed understanding of wind turbines means there may be value in information above the traditional cutoff frequency.



## Time-of-Generation Matters
The proliferation of cheap solar PV is causing strong seasonal and daily cycles in power prices around the world.  Rival sources of generation must assess their exposure to these low and even negative prices, or risk financial underperformance. Thanks to the relative predictability of the seasonal/diurnal cycles of solar power, risk-conscious developers can assess their exposure by analyzing the temporal patterns in their own production.
Price forecasting is notoriously uncertain, but the structural trend of solar PV deployment seems durable and credible enough to plan around.
The wind power industry has traditionally used annual production as their primary summary metric. But due to the downward price pressure of cheap (and predictable) solar power, among other things, there is growing interest in understanding the temporal distribution of production.
Despite the recent interest in time-aware modelling, 