---
title: "The Breaking Points - When Did the Climate Trend Actually Change?"
date: 2026-05-28 09:00:00 +0530
categories: [Climate Signal Lab, Data Science]
tags: [climate, changepoint, pelt, python, statistics, wed2026]
math: true
description: "Article 2 of 5 — Using the PELT algorithm to statistically detect the exact years the climate trend structurally broke."
---

*Article 2 of 5 in the **Climate Signal Lab** series — published leading up to World Environment Day, June 5, 2026.*

---

In Article 1, we looked at 145 years of NASA temperature data and saw a clear upward curve. The rolling mean told us warming was happening and accelerating.

But "it's getting worse" is not a precise statement. It's an observation.

Today I want to be precise. I want to answer: **exactly when did the climate trend structurally break?** Not visually — statistically. Using a real algorithm on real data.

---

## The Problem With Just "Looking" at Data

When you look at the temperature chart from Article 1, you can roughly see that something changed around the 1980s. The curve starts going up more steeply. But your eyes can be fooled — by noise, by scale, by where you're looking.

This is where statistics becomes genuinely useful. Instead of eyeballing a chart and guessing, we can use an algorithm that reads every single data point and mathematically identifies where the underlying trend changed — with no human bias involved.

That algorithm is called **PELT**.

---

## What is PELT?

PELT stands for **Pruned Exact Linear Time**. It's a change point detection algorithm — meaning it finds the moments in a time series where the statistical behavior of the data fundamentally shifts.

Think of it this way. Imagine you're tracking the daily temperature in your city over 10 years. Most of the time it fluctuates normally. But one year, a factory opens nearby and the average temperature subtly but permanently shifts upward. PELT would find exactly which month that shift happened — even if it's not obvious to the naked eye.

The math behind it finds the best way to divide the data into segments by minimizing a cost function:

$$\sum_{i=1}^{m+1} C(y_{\tau_{i-1}+1:\tau_i}) + \beta m$$

> *Not a math person? Skip the formula. The idea is: find the division of the data into segments where each segment is as internally consistent as possible, while penalizing for adding too many segments.*

Where:
- $C$ = how "inconsistent" the data is within each segment
- $\beta$ = a penalty that stops the algorithm from finding too many breakpoints
- $m$ = number of change points found

The "Pruned" part is what makes it efficient — it intelligently skips segments that can never be optimal, making it fast even on large datasets.

---

## What the Algorithm Found

I ran PELT on 145 years of NASA temperature data with a penalty of 3. It detected **4 change points** — meaning 5 distinct climate regimes in the data.

The years: **1936, 1978, 2000, and 2014.**

Here's what each segment looks like:

| Segment | Years | Warming Rate | Mean Anomaly |
|---------|-------|-------------|--------------|
| 1 | 1880–1936 | +0.001°C per decade | −0.256°C |
| 2 | 1937–1978 | −0.005°C per decade | −0.005°C |
| 3 | 1979–2000 | +0.124°C per decade | +0.311°C |
| 4 | 2001–2014 | +0.096°C per decade | +0.634°C |
| 5 | 2015–2024 | +0.273°C per decade | +0.987°C |

The last segment is warming **222 times faster** than the first.

Let that number sit for a moment.

---

## Breaking Down Each Segment

**1880–1936: The Quiet Era**

For the first 56 years of the record, the warming rate was essentially zero — +0.001°C per decade. The planet was stable. Industrial emissions existed but hadn't yet accumulated enough to meaningfully shift the climate system.

**1937–1978: The Cooling Anomaly**

This surprised me when I first saw it. The algorithm detected a slight *cooling* trend in this period. How?

This is actually a well-documented phenomenon called **global dimming**. Post-World War II industrial expansion released massive amounts of aerosols and particulate pollution into the atmosphere. These particles reflected sunlight back into space, temporarily masking the warming effect of CO₂. It's a strange, uncomfortable fact — the same industrialization that was pumping CO₂ into the atmosphere was also, for a time, partially canceling its own warming effect.

The Clean Air Acts of the 1970s reduced this pollution — and ironically, unmasked the warming that had been building underneath.

**1979–2000: The Awakening**

The trend breaks sharply in 1979. Warming accelerates to +0.124°C per decade — 124 times faster than the first segment. This is the period when climate scientists started raising serious alarms. James Hansen's famous 1988 Congressional testimony happened right in the middle of this segment.

**2001–2014: The New Normal**

The mean anomaly in this segment is +0.634°C — meaning the planet was now, on average, more than half a degree above the baseline. Every single year in this period was warmer than the 1951–1980 average. What was once an anomaly became the baseline.

**2015–2024: Acceleration**

The final segment has a mean anomaly of +0.987°C — nearly a full degree above baseline. The warming rate is +0.273°C per decade, the fastest of any segment. 2024 closed this period at +1.29°C — the hottest year ever recorded.

---

## The Math Section

The cost function PELT minimizes for a Gaussian signal:

$$C(y_{a:b}) = (b-a)\ln(\hat{\sigma}^2_{a:b})$$

And the year-over-year rate I calculated for each segment:

$$r_i = \frac{\Delta T_i}{\Delta t_i} \times 10 \quad \text{(°C per decade)}$$

> *Plain English: for each segment, fit a straight line through the data and measure how steep it is. That steepness is the warming rate.*

The penalty parameter $\beta = 3$ controls sensitivity. A lower penalty finds more breakpoints; higher finds fewer. I tested values from 1 to 10 — the 4 breakpoints at 1936, 1978, 2000, and 2014 were consistently detected across most penalty values, which means they are robust, not artifacts.

---

## The Visualization

<div style="width:100%; height:600px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article2_chart.html"
        width="100%" height="600px"
        style="border:none; border-radius:8px;">
</iframe>
</div>

*Each colored line is the trend for that segment. The dashed vertical lines mark the detected change points. Hover to see exact values.*

---

## My Take

*(Add your personal opinion here — what surprised you, what frustrated you, your perspective as someone living in India experiencing these changes firsthand)*

The number that I keep coming back to is **222x**. The climate is warming 222 times faster in the last decade than it was in the first 56 years of the record. That is not a linear problem. That is an exponential one.

And yet our policy responses have been largely linear — slow committees, incremental targets, decade-long timelines. There is a fundamental mismatch between the speed of the problem and the speed of the response.

The algorithm doesn't know about politics. It just finds where the math changed. The math changed in 1979. We've had 45 years.

---

## What's Coming Next

**Article 3 drops May 31.**

We switch from temperature to energy — specifically, renewable energy adoption. I'll fit exponential and logistic growth curves to global solar capacity data and forecast when renewables could realistically dominate the grid.

After two articles about what's going wrong, it's time to look at what's actually working.

---

*The algorithm found 4 breakpoints. History gave us the reasons.*

*Which of the 5 segments surprised you most? Drop it in the comments.*

---

*Data: [NASA GISS v4](https://data.giss.nasa.gov/gistemp/)*  
*Code: [Jupyter Notebook on GitHub](https://github.com/NIKHCODE/climate-signal-lab/blob/main/notebooks/article2_changepoint.ipynb)*  
*Series: Climate Signal Lab — 5 articles for World Environment Day 2026*
