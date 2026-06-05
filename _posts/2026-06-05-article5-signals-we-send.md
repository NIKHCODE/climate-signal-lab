---
title: "The Signals We Choose to Send Back - A Climate Forecast for 2050"
date: 2026-06-05 09:00:00 +0530
categories: [Climate Signal Lab, Data Science]
tags: [climate, lstm, neural-network, forecast, india, wed2026, python]
math: true
description: "Article 5 of 5 - Training an LSTM neural network on 145 years of NASA data to forecast three climate scenarios to 2050, with an India-specific projection."
---

*Article 5 of 5 in the **Climate Signal Lab** series. Published on World Environment Day, June 5, 2026.*

---

This is the final article in the series.

Over the past ten days I built four data-driven pieces about climate change. Temperature trends going back to 1880. The exact years the warming accelerated. Solar energy growing at 24% per year. A machine learning model that found wealth, not fossil fuels, as the primary driver of emissions.

Today, on World Environment Day, I want to do something different. I want to look forward.

I trained a neural network on 145 years of temperature data and asked it a simple question: where are we going?

The answer is uncomfortable. But it is honest. And I think honesty is the only thing worth offering on a day like today.

---

## What is an LSTM and Why Does It Matter Here

Most machine learning models treat each data point independently. Feed in a country's GDP and renewable percentage, get an emissions prediction. Each input is evaluated on its own.

Climate does not work that way. Temperature in 2024 is not independent of temperature in 2014 or 1994. The climate system has memory. CO2 emitted fifty years ago is still warming the planet today. Decisions made in the 1970s are showing up in the thermometer readings of 2026.

An LSTM, which stands for Long Short-Term Memory, is specifically designed for this kind of sequential, memory-dependent data. It was developed by Sepp Hochreiter and Jurgen Schmidhuber in 1997 and has since become one of the most widely used architectures for time series forecasting.

The key innovation of an LSTM is its cell state, which acts as a long-term memory that runs through the entire sequence. At each step, three gates control what information flows in, what gets forgotten, and what gets passed forward:

$$f_t = \sigma(W_f \cdot [h_{t-1}, x_t] + b_f)$$

This is the forget gate. It looks at the previous output and the current input and decides what fraction of the existing memory to keep. A value close to 1 means keep everything. A value close to 0 means forget it.

$$i_t = \sigma(W_i \cdot [h_{t-1}, x_t] + b_i)$$

This is the input gate. It decides how much of the new information to write into memory.

$$C_t = f_t \cdot C_{t-1} + i_t \cdot \tilde{C}_t$$

This is the cell state update. Old memory multiplied by the forget gate, plus new information multiplied by the input gate. The cell state carries the history of the sequence forward.

$$h_t = o_t \cdot \tanh(C_t)$$

This is the output. What the network communicates to the next timestep, and ultimately what becomes the prediction.

The reason this architecture is appropriate for climate data is the same reason climate science is difficult. The past matters enormously. A neural network that only looks at recent inputs would miss the accumulated momentum of decades of emissions. The LSTM does not.

---

## How I Built the Model

I trained a two-layer LSTM with 64 hidden units on the NASA GISS temperature dataset spanning 1880 to 2024.

The training setup:

Each input to the model is a sequence of 20 consecutive years of normalized temperature readings. The model's task is to predict the 21st year. This was repeated across 100 training sequences drawn from the historical record.

The model was trained for 300 epochs using the Adam optimizer with a learning rate of 0.001 and Mean Squared Error as the loss function:

$$\text{MSE} = \frac{1}{n}\sum_{i=1}^{n}(y_i - \hat{y}_i)^2$$

Training results:

- Initial loss: 0.0495
- Final loss: 0.0043
- Improvement: 91.3%
- R² on test data: 0.7388
- RMSE: 0.108°C

An R² of 0.74 means the model explains 74% of the variation in unseen temperature data. For a single-variable climate forecast trained on 145 years of data, this is a solid result. The 0.108°C average error is well within the range of meaningful climate signal.

---

## The Three Scenarios

Once trained, I ran the model forward from 2025 to 2050 under three different assumptions about how the world responds to climate change.

**Scenario 1: Business as Usual**

Current policies continue. Emissions reductions happen slowly. The model follows its learned momentum with a small compounding upward bias representing the continued accumulation of greenhouse gases.

Result by 2050: **+3.03°C**

**Scenario 2: Moderate Action**

Current commitments are met but not exceeded. Renewable deployment continues at its current pace. No acceleration, no backsliding. The model runs on its own momentum without additional bias.

Result by 2050: **+2.92°C**

**Scenario 3: Aggressive Climate Action**

Major global policy shift. Rapid decarbonization. The model receives a compounding downward bias representing strong intervention in the warming trajectory.

Result by 2050: **+2.74°C**

The gap between the best and worst scenario is 0.29°C. That number deserves some context. At global scale, 0.29°C is the difference between hundreds of millions of people facing manageable climate stress versus catastrophic displacement. It is the difference between coral reefs surviving and dying. It is the difference between the Himalayan glaciers feeding Indian rivers through 2100 or collapsing by 2070.

The difference between doing nothing and doing everything we possibly can is 0.29°C. That is both encouraging and devastating depending on how you look at it.

---

## The LSTM Forecast Chart

[View Interactive LSTM Forecast Chart](https://nikhcode.github.io/climate-signal-lab/assets/article5_chart.html){:target="_blank"}

Look at where the three scenario lines begin in 2025. They are nearly identical for the first few years. This is not a modelling artifact. It reflects something real and important: the near-term trajectory of global temperatures is already largely determined by emissions already in the atmosphere. The choices we make today will show their full effect in 2040 and beyond, not in 2027.

The 1.5°C Paris Agreement target, marked by the yellow dashed line, is crossed by all three scenarios within the first few years of the forecast. This is consistent with what climate scientists have been saying for several years. The 1.5°C target is effectively no longer achievable. The question now is how far beyond it we go.

The 2.0°C line, marked in orange, is crossed by the Business as Usual scenario around 2040. The Aggressive Action scenario crosses it later but still crosses it before 2050.

This is the honest picture. There is no scenario in this model where we stay below 2.0°C by 2050. The question is whether we end up at 2.74°C or 3.03°C, and what we build between now and then to adapt to whichever world we create.

---

## India's Climate Choice

Earlier in this series, Article 4 showed that India emits 1.9 tonnes of CO2 per capita, 69% below the world average, primarily because of low income levels rather than deliberate climate policy.

But India is growing. Its economy is expanding rapidly. Its middle class is enlarging. Its energy demand is rising. The question is not whether India's emissions will increase. They will. The question is whether that increase can be managed through renewable energy investment.

I ran three India-specific scenarios using the same LSTM model:

**Current Path (19% renewable energy today):** If India continues on its existing trajectory without significantly accelerating clean energy deployment, the model projects a contribution to global warming of **+3.00°C** by 2050.

**Government Target (50% renewable by 2035):** India has committed to reaching 500 GW of renewable capacity by 2030 and 50% clean energy by 2035. Following this path reduces the projected contribution to **+2.86°C** by 2050.

**Brazil Path (83% renewable by 2040):** Brazil has achieved 83% renewable energy primarily through hydropower built over decades. If India could match this mix through solar and wind by 2040, the projected contribution falls to **+2.74°C** by 2050.

The difference between doing nothing and matching Brazil is 0.20°C at the global level. But at the India level, the difference is the energy security, air quality, and climate resilience of 1.4 billion people.

[View India 2050 Scenarios](https://nikhcode.github.io/climate-signal-lab/assets/article5_india.html){:target="_blank"}


The Brazil comparison matters because it shows what is possible. Brazil did not stay poor to keep its emissions low. It built renewable infrastructure decades ago and is now a developing economy with clean energy and relatively low per capita emissions. The path exists. It requires capital, political will, and international support.

Whether India receives that support, or is left to choose between development and climate responsibility on its own, will be one of the defining questions of the next twenty years.

---

## The Complete Series Dashboard

Below is every chart from all five articles in one place. From 1880 temperature records to 2050 forecasts. From change point detection to solar growth curves to SHAP analysis to neural network scenarios.

[View Complete Series Dashboard](https://nikhcode.github.io/climate-signal-lab/assets/article5_dashboard.html){:target="_blank"}

---

## My Take

*(Here are honest directions for you to write from)*

Today is World Environment Day 2026. It is being hosted in Azerbaijan, a country that earns the majority of its government revenue from oil and gas exports, situated in a region where geopolitical tensions have made international cooperation harder than at any point in recent memory.

Write about the gap between the ceremony of climate events and the reality of climate data. The speeches today will talk about hope and commitment. The data says we will cross 1.5°C regardless of what is said at any podium.

Write about what it means to be a young person in India looking at these numbers. The 0.29°C difference between best and worst case scenarios represents real consequences for real people in real places, including the city you live in, the temperatures you are already experiencing, and the water sources your region depends on.

Write about the India question specifically. Is it fair to ask a country that contributed almost nothing to the historical accumulation of greenhouse gases to bear equal responsibility for solving the problem? The LSTM does not have an opinion on this. You do.

Write about what building this series taught you. Five articles. Five notebooks. Real data. Real models. What did looking at this data yourself, without a journalist or politician framing it for you, change about how you see the problem?

The Earth has been sending signals for 145 years. The question the series was always building toward is not whether we can read them. We can. The question is what we choose to do with what we have read.

---

## A Note on This Series

This series was built entirely from publicly available data and open source tools. Every dataset is linked. Every notebook is on GitHub. Every chart is interactive and built from raw numbers.

I am a computer science student. I am not a climate scientist. I did not discover anything that was not already known. What I tried to do was make the known things visible in a form that anyone can look at directly, without needing to trust a news anchor or a government report or a think tank.

If one person reads these five articles and comes away with a clearer, more honest picture of where we are and where we are going, the series achieved what it set out to do.

Thank you for reading.

---

*Data: NASA GISS, IRENA, World Bank, IEA, Our World in Data*
*Code: [GitHub Repository](https://github.com/NIKHCODE/climate-signal-lab)*
*Series: Climate Signal Lab, 5 articles for World Environment Day 2026*
