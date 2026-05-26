---
title: "The Earth Is Speaking, 140 Years of Climate Data, and What It's Telling Us"
date: 2026-05-26 09:00:00 +0530
categories: [Climate Signal Lab, Data Science]
tags: [climate, eda, python, nasa, plotly, wed2026, time-series]
math: true
description: "Article 1 of 5 — Using NASA GISS data, rolling statistics, and Python to understand 140 years of global temperature change."
---

This is the first article in a 5-part series leading up to **World Environment Day on June 5, 2026**. Each article pairs real data analysis with honest opinion. No doom, no denial — just what the numbers actually say.

---

Let me start with a number.

## +1.29

Wondering what this number is?

That is how much warmer 2024 was compared to the average temperature of the Earth between 1951 and 1980. Not a prediction. Not a worst-case scenario from a climate model. The actual measured temperature, recorded by thousands of weather stations and ocean buoys across the planet, compiled by NASA.

Now you might read that and think — one degree? That doesn't sound like much.

I thought the same thing. Until I actually looked at the data myself and realized what a single degree can actually do.

---

## Why I Decided to Do This

I'm a computer science student who is also a very pro-environment human, and honestly, I was tired of reading about climate change through the filter of news articles and opinion pieces. Everyone has an angle and a different perspective. The global left is panicking while the right is dismissing.

I really wanted clarity. Are we doing better or not? Did conditions improve? There were a lot of questions, and all I could think was — *okay, why not do the research on my own and come to a conclusion ?*

So here I am doing what I do best. I went to the source. I downloaded 145 years of raw temperature data directly from NASA. I wrote Python code to analyze it. I plotted it myself. No editorial spin. Just numbers.

What I found genuinely surprised me. And I think it'll surprise you too.


---

## First, What Are We Even Measuring?

The dataset I used is called the [NASA GISS Surface Temperature record (v4)](https://data.giss.nasa.gov/gistemp/). It goes back to 1880 — that's 145 years of data.

Here's something important to understand: scientists don't track *absolute* global temperature. They track *anomalies* — how much warmer or cooler a given year was compared to a fixed reference period. NASA uses 1951–1980 as that reference. This is the standard I'll be following for my entire series.

So when I say 2024 was +1.29°C, I mean it was 1.29 degrees warmer than the average of those 30 years.

Why does this matter? Because absolute temperature varies wildly across the planet — the Sahara and Antarctica are both "on Earth" but their temperatures are incomparable. Anomalies strip away that location noise and give you a clean, honest picture of whether the planet as a whole is warming or cooling.

I also pulled global CO₂ emissions data from [Our World in Data](https://ourworldindata.org/co2-and-greenhouse-gas-emissions) — because temperature doesn't exist in isolation. Carbon dioxide is the engine behind the warming, and I wanted to see both stories on the same chart. We've all heard of global warming and greenhouse gases. I dug deep to analyze how CO₂ is actually showing up in the numbers.

---

## The Math Behind the Conclusions

*(It's simpler than it looks — I promise)*

I used three mathematical tools to analyze the data. I want to explain them in plain English, because understanding *how* the analysis works is what separates this from just reading a headline.

### Tool 1: Rolling Mean — Finding the Real Trend

Year-to-year temperature can be noisy. A single volcanic eruption can temporarily cool the planet. An El Niño event can spike temperatures for one year. If you just look at raw annual data, it's hard to see the underlying trend.

The "rolling mean" smooths this out. For each year, I calculated the average temperature of the 20 years surrounding it:

$$\bar{x}_{t} = \frac{1}{20}\sum_{i=t-10}^{t+10} x_i$$

> *Not a math person? The idea is simply — zoom out to see the real trend, ignore the yearly noise.*

Think of it like a moving average in stock markets. It filters the noise and shows you the actual direction things are moving. In climate terms, it's the difference between weather and climate — one is the noise, the other is the signal.

---

### Tool 2: Standard Deviation Band — Measuring Stability

Around that rolling mean, I calculated a band of ±1 standard deviation:

$$\sigma_t = \sqrt{\frac{1}{20}\sum_{i=t-10}^{t+10}(x_i - \bar{x}_t)^2}$$

> *Plain English: this band shows how predictable the climate was in each era. Wider band = more erratic = more surprises.*

A narrow band means temperatures were consistent. A widening band means the system is becoming more unstable — bigger swings, more extreme years. You'll notice in the chart that this band gets noticeably wider in recent decades.

---

### Tool 3: Acceleration Detection — When Did Things Really Speed Up?

This was the most interesting one to me. It does a year-by-year analysis and tells us exactly which years the mercury really jumped.

I calculated year-over-year temperature change for every year in the dataset:

$$\Delta_t = x_t - x_{t-1}$$

Then I flagged any year where this jump was unusually large — specifically, more than 1.5 standard deviations above the average rate of change:

$$\text{Flagged if: } \Delta_t > \mu_\Delta + 1.5\sigma_\Delta$$

> *Plain English: find the years where warming didn't just continue — it suddenly lurched forward.*

This tells us not just *that* temperatures are rising, but *when the rate of rise itself suddenly jumped*. Those are the moments worth paying close attention to.

---

## What I Found — And What It Actually Means

Here's where it gets real.

**The planet's baseline has fundamentally shifted.**

When I compared the smoothed temperature trend before 1940 versus after 2000, here's what came out:

| Era | Average Temperature Anomaly |
|-----|----------------------------|
| Pre-1940 | −0.237°C |
| Post-2000 | +0.639°C |
| **Total Shift** | **+0.876°C** |

Nearly a full degree of shift — and remember, this is the *smoothed* trend. The noise has already been removed. This is the underlying reality.

To put that in perspective: the temperature difference between today and the last ice age — when massive glaciers covered much of North America and Europe — is only about 5°C. We've moved almost a fifth of the way toward ice-age-level change, but in the *opposite direction*, and in just 60 years instead of thousands.

Isn't that concerning?

---

**Now let's look at the gas we exhale — yes, carbon*die*oxide.**

CO₂ didn't just grow. It exploded.

- CO₂ emissions in 1880: **858 billion tonnes**
- CO₂ emissions in 2022: **37,527 billion tonnes**
- That is **43.7 times more** CO₂ in 140 years

Not 43% more. Not 4x more. **43.7 times.** The scale of this is genuinely hard to comprehend — which is probably why most people don't.

---

**The warming isn't a straight line — it accelerated. And it won't stop soon** (unless we as citizens and our governments actually take action — which, so far, we haven't done nearly enough of).

My analysis detected **8 years of statistically significant acceleration** in the warming rate. But more telling than individual years is the overall shape: the 20-year rolling mean, which was barely moving in the early 1900s, begins curving upward around the 1970s, and by the 2000s it's going nearly vertical.

This is not a gradual drift. The system changed pace.

---

## The Visualization

Everything you just read is visible in this chart — built directly from NASA and OWID data. No approximations, no styling tricks. Every line is computed from raw numbers.

<div style="width:100%; height:700px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article1_chart.html"
        width="100%" height="700px"
        style="border:none; border-radius:8px;">
</iframe>
</div>

*Hover over any point to see exact values. The bold red line is the 20-year rolling mean. The shaded region is the ±1σ band. The dashed line is the 1951–1980 baseline. The blue chart below shows CO₂ emissions over the same period.*

Spend a minute with this chart. Look at where the red line was in 1900. Then follow it to 2024. That curve is not a political opinion — it's 145 years of thermometer readings.

---

## My Honest Take

Here's something that genuinely frustrates me.

This data has been publicly available for decades. NASA publishes it for free. Anyone with Python and an afternoon can reproduce everything I did here — which is exactly what I did. So my question is: what is stopping governments from acting on it?

This is information about where we live. About the air we breathe and the seasons our children will experience.

I'm writing this in May, sitting in my house in Andhra Pradesh with sweat pouring down. The temperature here is touching 50°C. As a child, I used to wonder how hot the Sahara and Thar deserts were. Unfortunately, I no longer have to go to Rajasthan to find out.

The numbers in this article are not abstract to me. They are the weather outside my window.

---

## What's Coming Next

**Article 2 drops on May 28th.**

We'll take this same dataset and ask a harder question: *when exactly did the climate trend break?* Not visually — statistically. Using an algorithm called PELT (Pruned Exact Linear Time), we'll detect the precise structural breakpoints in the temperature record and compute what the warming rate was before and after each one.

It's the difference between saying *"things got worse"* and being able to say *exactly when, by how much, and how fast.*

Along with the data, something more from me and my personal experience.

---

*The Earth doesn't editorialize. It just records.*

*What surprised you most when you first saw raw climate data? Drop it in the comments — I'd genuinely like to know.*

---

*Data: [NASA GISS v4](https://data.giss.nasa.gov/gistemp/) · [Our World in Data CO₂](https://ourworldindata.org/co2-and-greenhouse-gas-emissions)*  
*Code: [Jupyter Notebook on GitHub](https://github.com/NIKHCODE/climate-signal-lab/blob/main/notebooks/article1_eda.ipynb)*  
*Series: Climate Signal Lab — 5 articles for World Environment Day 2026*
