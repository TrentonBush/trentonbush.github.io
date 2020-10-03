---
title: "test"
excerpt: "blah blah"
date: 2020-09-23 00:00:00 -0000
---

# For Wind Power, Existing Turbulence Metrics Are Sufficient

I worked in the wind energy industry for several years, creating and running data-driven models to predict energy production of prospective projects. Over that time, the industry faced pressure from investors to improve its 20+ year old data practices in pursuit of more accurate predictions. Updating existing systems is always a bumpy process, and there remain many opportunities for improvement. The following is an experiment I ran to test a hypothesis that spectral information from high frequency data could improve prediction accuracy. It is written for an audience that understands data but has no domain knowledge in energy or wind power. Thanks for reading!

## The Big Picture
### The Role of Data Science in Wind Energy
The primary job of data scientists and engineers at companies that develop or invest in wind power is to estimate future revenue of wind farms. To deliver business value, data scientists must reveal financial upside or downside that counterparties are not yet aware of, and defend against those counterparties' own novel insights. This creates a race to understand future revenue.

At its core, energy production is a commodity business where $revenue = production x market price per unit$ (plus a layer of hedges and other financial engineering that lie outside my understanding). A key nuance in that revenue equation is that prices and production vary wildly in time as the grid constantly tries to balance supply and demand. Prices might be thousands of dollars on a hot summer afternoon or *negative* tens of dollars on a windy winter morning. Understanding correlations between power prices and wind power production can be a driver of business value. Power prices are a complex system beyond the scope of this post, so I will focus on an application in estimating wind power production.

### Turbulence as Opportunity
Future production estimates are largely based on a dataset of meteorological measurements taken at each project location. Due to historical precedent, the industry still relies on coarse aggregate measurements and methodologies that were designed 20 to 30 years ago. That was simply a different era of computational limits. One opportunity for improvement is to better measure turbulence, an important parameter in the conversion of wind speed to electrical power.

In short, turbulence refers to the temporal frequency of changes in wind speed. This effects power generation because wind turbines must constantly tune the orientation and angle of the blades and hub to extract maximum power from incoming wind, but turbines are large machines with limited reaction times. "High turbulence" refers to changes in wind speed that occur faster than the turbine can react. This generally results in lower power production than would be produced with lower turbulence (slower changes), all else being equal.

## Turbulence is Poorly Measured
I hypothesized that outdated measurement standards and a single poorly designed metric make accounting for turbulence unecessarily difficult and noisy.

Wind data is produced by sampling about a dozen sensors every second (1 Hz), for one to three years. Thirty years ago, when these measurement systems were designed and standardized, collecting 1 Hz data for a year was likely cost prohibitive (a quick estimate: 31.5 million seconds per year * 12 columns * 4 bytes per value = 1.5 GB per year, uncompressed. Trivial today, not so in the 1990s). To preserve storage space, data loggers are programmed to downsample the raw 1 Hz data to 10-minute aggregates, recording a mean, standard deviation, minimum, and maximum from each column for each 10-minute period. This reduces data size by a factor of 150, to about 10 MB per year.

Because of this downsampling, turbulence gets redefined as wind speed changes that occur more frequently than the downsampling frequency of once per 10-minutes. But the relevant cutoff frequency - turbine reaction time - is more like once in 30 seconds. The downsampling is too coarse by a factor of 20! Furthermore, this information loss could be mitigated with the right aggregate statistics, but the current statistics discard all temporal information; there is no record of the actual frequency content of the wind! As a result, much of the systematic effect of turbulence is lost, instead converted to noise. It is likely too late to change the industry standard 10-minute time interval, but it would be relatively easy to add a new aggregate metric that captures most of the frequency information.

## Experimental Design
To test whether frequency information helps predict power output, an ideal experiment would be to compare accuracy of existing models to a frequency-aware model over several operational projects. Unfortunately, this requires proprietary turbine production data. There are many companies that could conduct such a study, but I don't have access to their data. Before buying expensive data, I use public data to ask bounding questions:
1. How much additional power is available in recoverable frequencies between 10 minutes and 30 seconds? [Does frequency matter?]
2. Can this excess power be adequately modeled with existing turbulence metrics based on standard deviation? [Do we need to bother?]
3. How does this extra recoverable power vary in time? [Is it forecast-able? Does it correlate with power prices, thereby impacting revenue?]

### Finding Good Data
The story of all successful data projects starts with sourcing good data. Because turbulence is a small-scale phenomena, it is sensitive to small details of sensor setup that could be safely ignored for easier measurements like average temperature. Additionally, the seasonal cycle is important in atmospheric data, so a minimal n=1 sample requires a full year of measurements (and maintenance!). This means I need high-quality, well-documented data that satisfies the following requirements:
1. temporal resolution of 0.5 Hz or higher [to imitate commercial systems]
2. data coverage across high-impact seasonalities (daily and annual). This implies two sub-requirements:
   1.  at least a year of data [a minimal n=1 sample of the seasons]
   2.  well monitored and maintained sensors that don't drift or fail
3. information about the biases of the test environment. For atmospheric data, this means:
   1. documentation of sensor mounting arrangements [even perfect sensor setup produces artifacts in the data. Measuring is hard!]
   2. geographic location
   3. topographic information in approx. 1km radius around the sensor location

These kinds of experiments require expertise (and $$$ and interest) not just to design and set up, but also to maintain. Most public aviation/agriculture/consumer weather stations lack the resolution, documentation, and long term quality required in this analysis, so my search quickly narrows down to government or academic labs. After browsing a few candidates, I settled on the National Renewable Energy Lab's (NREL) National Wind Technology Center (NWTC) meteorological masts.

### Big Data Problems
NREL's NWTC masts fully satisfy my quality requirements, but with three minor compromises on quantity/access (and thus cost). First, their data are sampled at 20Hz, which is 40 times more data than I want to deal with. Second, the data are only provided as flat files, which means I have to download the full volume (1.5 TB total, or 180 GB per year) just to pick out the 10% I actually need. Finally, due to the the glacial response times of their server (1 to 6 seconds for a HEAD request) and sheer number of files (one per 10 minutes = 52,560 per year = 450,000 total), my estimated download time was about 100 hours per year of data. That would strain my time budget for this project, so I had to figure something else out.

#### Subsetting
The full dataset is about 1.5 terabytes, spanning 2012 to the present. That is more than I want to deal with for a portfolio project (33+ days to download in serial), so instead I chose to examine a single year. But which one? I didn't want to spend days downloading data only to find out there were big gaps due to a sensor failure or something.

I used a simple method to decide which year to use: scrape the index.html tables to get the file sizes and timestamped filenames, and use whichever year had the highest total file size. This heuristic accounts for missing files (size 0) as well as failed sensors (repeated values get compressed). It cannot determine *which* sensors are available or *why* one file might be more compressible than another (maybe true windspeed was 0, not a sensor failure). But that was the best I could do without downloading everything.

#### Downloading
Due to the several-second response time of the server, I chose to spend a day replacing my serial requests.get() download script with an asynchronous one built on httpx, with rate- and concurrency-limiting to avoid crushing NREL's beleagured server. [link to script] This cut download time from 100 hours per year to less than 30.

After the initial download, I verified that all the files missing due to HTTP errors were in fact missing, and also re-downloaded any files that failed to open due to interrupted downloads. These steps recovered 426 files, 0.8% of the dataset.

#### Parallel Processing
Two more steps remained in the ETL process: getting the data out of .matlab files, and simulating industry-standard downsampling to 10-minute aggregates. These operations involve computation over the whole 180 GB dataset, which is larger than memory on my machine. Initially, I thought I would need to use Dask to parallelize and manage the out-of-core computation graph. But my processing turned out to be a simple mapping with output that fit in memory, so I just used the builtin multiprocessing module for an easy 4x speedup down to 40 minutes. I could have squeezed extra performance out of a fifth core if I had one, but beyond that the bottleneck would be file IO off my SSD.

Reading .matlab files was easier than I thought, thanks to the kind souls behind scipy.io.loadmat. As usual with external data, I still had to make a few functions to fetch the information I needed and get it in the right format.




A good turbulence metric should be designed with the end use in mind - estimating power production. Electrical energy is proportional to wind speed cubed, and relevant wind speeds are those up to the cutoff frequency. By taking the cumulative integral of the cubed frequency spectrum, we can measure the available power up to any arbitrary cutoff frequency. [Those familiar with signal processing may be reminded of the power spectral density - this is very similar but with cubed rather than squared spectral components, followed by cumulative integration.] As a generation model, this essentially replaces high frequency power with the (lower) power of the average speed. I think that is a reasonable assumption, but it is plausible that actual performance would be slightly less due to aerodynamic inefficiency. This would really need to be validated with actual production data, but alas, that is all proprietary.

frequency Lower frequency changes are directly recorded in measured data, while higher frequencies are aggregated into a coarse summary statistic: the standard deviation. The arbitrary cutoff point is determined In the 30 years since these measurement standards were adopted, increasing computing power/storage and our more detailed understanding of wind turbines means there may be value in information above the traditional cutoff frequency.



## Time-of-Generation Matters
The proliferation of cheap solar PV is causing strong seasonal and daily cycles in power prices around the world.  Rival sources of generation must assess their exposure to these low and even negative prices, or risk financial underperformance. Thanks to the relative predictability of the seasonal/diurnal cycles of solar power, risk-conscious developers can assess their exposure by analyzing the temporal patterns in their own production.
Price forecasting is notoriously uncertain, but the structural trend of solar PV deployment seems durable and credible enough to plan around.
The wind power industry has traditionally used annual production as their primary summary metric. But due to the downward price pressure of cheap (and predictable) solar power, among other things, there is growing interest in understanding the temporal distribution of production.
Despite the recent interest in time-aware modelling, 