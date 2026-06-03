---
title: "What Actually Drives CO2 Emissions - A Machine Learning Answer"
date: 2026-06-03 09:00:00 +0530
categories: [Climate Signal Lab, Data Science]
tags: [climate, machine-learning, shap, random-forest, python, india, wed2026]
math: true
description: "Article 4 of 5 - Using Random Forest and SHAP values to find what really drives CO2 emissions across 40 countries, with a deep dive into India."
---

*Article 4 of 5 in the **Climate Files** series, published leading up to World Environment Day, June 5, 2026.*

---

Everyone has a theory about what causes climate change.

Rich countries blame population growth in developing nations. Developing nations point at the industrial history of the West. Oil companies talk about individual carbon footprints. Governments cite technological limitations. Everyone is pointing at someone else and what we see everyday is a blame game.

I got tired of the circular argument. So I built a machine learning model, fed it data from 40 countries which are economically, socially and geographically important, and let the math decide. 
The answer was more interesting than I expected and more uncomfortable.

---

## What I Built and Why

For this article I trained a **Random Forest** model on country-level data and then used **SHAP analysis** to understand exactly what the model learned. Together these two tools answer not just "can we predict emissions" but "why does each country emit what it emits."
We all know that there is carbon-dioxide emissions and this is the primary reason for global warming but we fail to understand is individual country wise analysis on the factors driving these emissions. In this piece we will discuss on factors with a special case study on India and what probable solutions we can come up with.

The dataset covers 40 countries representing every continent, income level, and energy mix. The five features I used as inputs:

- GDP per capita (how wealthy the country is)
- Renewable energy percentage (how much of energy comes from clean sources)
- Urban population percentage (how city-heavy the population is)
- Industry share of GDP (how manufacturing-intensive the economy is)
- Fossil fuel percentage of energy mix (direct fossil fuel dependency)

The target variable: CO₂ emissions per capita, measured in tonnes per person per year.

---

## The Two Methods Explained

### Random Forest

A decision tree works like a flowchart. It asks questions about a country's data and follows branches to reach a prediction. For example:
Is GDP per capita above $30,000? 
 Yes → Is fossil fuel % above 70%?
 Yes → Predict: 12 tonnes
 No  → Predict: 7 tonnes
 No → Is renewable % above 50%?
 Yes → Predict: 2 tonnes
 No  → Predict: 3.5 tonnes

One tree can be wrong. It might learn patterns specific to the training data that don't generalize. A Random Forest solves this by building 200 such trees, each trained on a slightly different random subset of the data, and averaging their predictions:

$$\hat{y} = \frac{1}{n}\sum_{i=1}^{n} T_i(x)$$

Where $T_i(x)$ is what tree number $i$ predicts for a given country. With 200 trees voting, the quirks of any individual tree get averaged out. The result is a more stable and reliable prediction. 

The model achieved an R² of 0.46 on the test set. This means it explains about 46% of the variation in emissions across countries. For a dataset of only 40 countries with 5 features, this is expected and acceptable. The goal here is not pinpoint prediction accuracy. The goal is understanding which factors matter most, and for that, the SHAP analysis is what we care about.

### SHAP Analysis

SHAP stands for SHapley Additive exPlanations. The idea comes from a branch of mathematics called cooperative game theory, specifically from work by Lloyd Shapley in 1953 for which he received the Nobel Prize in Economics in 2012.

The question SHAP answers is: for a specific prediction, how much did each feature contribute?

The mathematical formulation:

$$\phi_i = \sum_{S \subseteq F \setminus \{i\}} \frac{|S|!(|F|-|S|-1)!}{|F|!} \left[ f(S \cup \{i\}) - f(S) \right]$$

This looks complex but the logic is straightforward. For each feature, consider every possible combination of the other features. In each combination, measure how much the prediction changes when you add this feature in. Average those changes across all combinations. The result is a fair, mathematically grounded measure of that feature's contribution.

A positive SHAP value means the feature pushed the prediction higher. A negative SHAP value means it pulled the prediction lower. The magnitude tells you how strong the effect was.

The key advantage over simple feature importance is that SHAP gives you country-specific values. We do not just know that GDP matters globally. We know exactly how much GDP is pushing up emissions for Australia specifically, and how much renewables are pulling down emissions for Norway specifically.

---

## How Each Factor Affects CO₂ Emissions

Before looking at the results, let me explain how each of the five features connects to carbon emissions in the real world.

**GDP per capita**

Wealth is the most direct driver of emissions because it determines lifestyle. A higher income means more cars, more flights, larger homes that need heating and cooling, more consumption of goods that require manufacturing and shipping, more meat-heavy diets which are carbon intensive. When a country becomes richer, its people do not just consume more of the same things. They consume more of everything, and those things tend to be energy-intensive.

This is not a criticism of prosperity. It is a description of how modern economies work. The entire infrastructure of a wealthy society, from highways to airports to supermarket supply chains, is built around cheap energy. When that energy comes from fossil fuels, wealth and emissions move together.

**Renewable Energy Percentage**

This is the counterforce to fossil fuels. A country that generates 80% of its electricity from hydropower, wind, or solar is breaking the link between energy consumption and CO₂. You can be wealthy and consume a lot of energy, but if that energy is clean, your emissions stay low. Norway is the best example in the dataset: high GDP ($82,000 per capita), 98% renewable energy, and only 7.6 tonnes of CO₂ per person.

However, the relationship is not simple. Renewables affect electricity emissions directly. Many other sources of emissions, particularly transportation, industrial processes, and agriculture, are harder to decarbonize with renewables alone.

**Urban Population Percentage**

This one surprised me. Our general perception is that Urban areas are polluted and carbon-heavy. But at the per-capita level, cities are actually more efficient. People in dense cities share infrastructure, use public transport, live in smaller spaces, and travel shorter distances for daily needs. Rural populations in large countries often drive long distances, live in larger and less energy-efficient homes, and have fewer alternatives to private vehicles.

The relationship is further complicated by what kind of urbanization is happening. Dense European cities produce very different emissions patterns than sprawling car-dependent American suburbs. 

**Industry Share of GDP**

Manufacturing, construction, and heavy industry are among the most carbon-intensive economic activities. A country where 40% of GDP comes from steel, cement, chemicals, and mining will emit significantly more than a service economy of similar wealth where most GDP comes from finance, software, and retail.

This is also related to global supply chains. When wealthy countries outsource their manufacturing to developing nations, they effectively export their emissions. The goods are consumed in rich countries but the carbon is counted in the producing country.

The best example that I can give is data centres. This industry requires huge fresh water to cool down its machinery. In the United States the companies like google faced protests against building them from local activits to internationally renowed environmentalists. The companies now started moving to countries like India which is eyeing investments to boost it's economy. This might heavily damage sensitive Indian eco-system. 
So what might be seen as development by one may not really be development for others.
 
**Fossil Fuel Percentage**

The most direct measure of dependence on carbon-emitting energy. A country where 95% of energy comes from coal, oil, and gas will emit significantly more than one with a diversified clean energy mix, all else being equal.

What the model found, however, is that this factor ranked last in importance among the five. This is counterintuitive but explainable. Fossil fuel percentage is highly correlated with wealth and development level. Poor countries often have high fossil fuel percentages not because they choose to, but because clean alternatives require capital investment they cannot afford. Rich countries have more renewable energy partly because they can afford to build it. Once the model accounts for GDP and renewables separately, the additional information provided by the raw fossil fuel percentage is smaller than expected.

---

## The Global Results

**SHAP Feature Importance Rankings:**

| Feature | SHAP Score | Interpretation |
|---------|-----------|----------------|
| GDP per capita | 2.57 | By far the strongest driver |
| Renewable % | 0.54 | Second most important |
| Urban % | 0.52 | Nearly as important as renewables |
| Industry % | 0.38 | Moderate effect |
| Fossil fuel % | 0.33 | Weakest of the five |

The gap between GDP (2.57) and everything else (all below 0.55) is striking. Wealth is not just the most important factor. It is five times more important than the next factor.

The global visualization below shows SHAP contributions for all 40 countries. Each bar shows how much GDP, renewables, and fossil fuels are pushing emissions up or down for each country. Countries are sorted from lowest to highest CO₂ per capita.

<div style="width:100%; height:700px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article4_chart_global.html"
        width="100%" height="700px"
        style="border:none; border-radius:8px;">
</iframe>
</div>

**How to read this chart:**

Countries at the bottom (Ethiopia, Kenya, Bangladesh) have the lowest emissions. Their bars are almost entirely on the negative side, meaning GDP is pulling their predictions down strongly. They are poor, so they emit little.

Countries on the top (UAE, Australia, USA, Canada) have the highest emissions. Their bars extend far into the positive side, with GDP being the dominant orange bar pushing emissions up.

The green bars (renewables) are visible as negative contributions for countries like Norway, Sweden, Brazil, and Denmark. These are the countries where investment in clean energy is actually measurably reducing emissions relative to what their wealth level would predict.

The red bars (fossil fuels) are relatively small across the board, confirming the model's finding that fossil fuel percentage adds limited explanatory power once GDP and renewables are accounted for.

One country worth noting is Canada. It appears in both the high-GDP high-emissions cluster and has significant green renewable bars. Canada has 67% renewable electricity but still emits 14.2 tonnes per capita because its GDP effect is so strong. Wealth is overwhelming the renewable benefit.

---

## The India Deep Dive

India deserves its own section because it is the most analytically interesting case in the dataset and because it is where I live and where these numbers are not abstract.

**The headline number: India emits 1.9 tonnes of CO₂ per capita.**

The world average is 6.16 tonnes. India is 69% below the world average. For the world's most populous country, the fifth largest economy, and a nation that is industrializing rapidly, this is a genuinely remarkable statistic. Everyday we hear news on pollution, AQI, heatwaves but atleast our emissions are far below the global average. (I am not dismissing that AQI and others things are not important but we are performing relatively better)

<div style="width:100%; height:520px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article4_chart_india.html"
        width="100%" height="520px"
        style="border:none; border-radius:8px;">
</iframe>
</div>

**Reading the India chart:**

Green bars pull emissions downward. Red bars push emissions upward. The longer the bar, the stronger the effect.

**GDP per capita: SHAP value of -3.687**

This is the single most powerful force keeping India's emissions low. India's GDP per capita is approximately $2,100. That places it well below the global average in the dataset. The model has learned that low income strongly predicts low emissions, and it applies that learning to India with full force.

What this means practically: most Indians cannot afford the emission-heavy lifestyle that drives numbers in wealthy nations. Private car ownership is significantly lower than in Western countries. Air travel remains inaccessible to most of the population. Home sizes are smaller. Consumption patterns are inherently more modest. This is not an environmental achievement. It is an economic condition.

**Urban population: SHAP value of -0.844**

Only 36% of India's population is urban. The model treats this as an emissions-reducing factor for the reasons explained earlier: dense urban environments are more energy-efficient per capita than dispersed rural ones. As India urbanizes over the coming decades, this factor will shift.

**Renewable energy: SHAP value of +0.122**

This is the counterintuitive result. India's renewable percentage is pushing its predicted emissions slightly upward. This seems wrong until you understand what the model has learned. In the training data, countries with higher renewable percentages tend to be wealthier European nations that still emit more than India despite their clean energy. The model has partially learned a correlation between renewables and development level. India's 19% renewable share is modest, but its poverty effect is so dominant that the renewable contribution barely registers.

**Fossil fuels: SHAP value of +0.074**

India gets 81% of its energy from fossil fuels. This is high by any measure, and it pushes the prediction upward. But the GDP effect is so strong in the opposite direction that this barely shows up in the final prediction. India burns a lot of coal, but it burns it to generate modest amounts of energy for a large population with low per-capita consumption.

**Industry share: SHAP value of -0.060**

A small negative contribution, suggesting India's industrial structure is not particularly more emission-intensive than what the model would predict given its other characteristics.

**India compared to similar nations:**

| Country | CO₂/capita | GDP/capita | Renewable % | Fossil % |
|---------|-----------|-----------|------------|---------|
| India | 1.9t | $2,100 | 19% | 81% |
| China | 8.0t | $12,500 | 29% | 71% |
| Brazil | 2.2t | $7,500 | 83% | 17% |
| Indonesia | 2.3t | $4,200 | 14% | 86% |
| Bangladesh | 0.5t | $1,800 | 3% | 97% |

The Brazil comparison is the most instructive. Brazil has 3.5 times India's GDP per capita but only slightly higher emissions, because 83% of its energy is renewable (primarily hydropower). This shows what is possible: a developing economy can grow its wealth significantly without proportionally growing its emissions, if it invests in clean energy infrastructure.

The China comparison shows what happens without that investment. China has 6 times India's GDP per capita and 4 times the emissions.

The path India takes over the next 20 years, as it inevitably grows wealthier, will be one of the most consequential climate decisions in human history. The math is clear about the options.

---

## My Take

Honestly I have spoken pretty much on everything from factors to maths and models. Howvwer I personally want to talk about India. In the visualization section I mentioned on factors and how our gdp is driving emission low but if you really understand the Indian way of living which is rooted in sustainable living its not really tough to see the math.

The SHAP model explains India's low emissions through GDP, urbanization, and energy mix. These are measurable variables. But there is a fifth factor that no dataset captures, and I think it deserves to be said explicitly.

Indians, as a culture, have been practicing low-waste sustainable living for thousands of years. Not because of climate policy. Not because of government mandates. Because of how we were raised.

Think about what happens in a typical Indian household:

A steel dabba that belonged to your grandmother is still being used to pack lunch today. Leftover rice from last night becomes tomorrow's breakfast. Old sarees become dusters. Broken plastic buckets become plant pots. Worn-out clothes are cut into rags before they are ever thrown away. Nothing leaves the house until it has been used completely, repurposed at least once, and exhausted of every possible function.

This is not nostalgia. This is a deeply embedded cultural philosophy that has no direct translation in Western economics. The closest term is the Japanese concept of *mottainai*, the feeling of regret over waste. In India it is not even a feeling. It is just how things are done.

The concept of **jugaad** — frugal innovation, making something work with whatever you have is not just an engineering principle. It is a way of life that extends from village workshops to middle-class kitchens to construction sites. Where a Western household might discard and replace, an Indian household finds a way to repair, repurpose, and extend.

Consider what this means at scale. India has 1.4 billion people. If even a fraction of daily consumption decisions involve choosing to reuse over replace, to repair over discard, to share over own individually, the cumulative carbon impact is enormous and completely invisible to any emissions dataset.

The joint family system itself is a carbon-efficient structure. Multiple generations sharing one home, one kitchen, one set of appliances. Compare that to the Western model of individual households, a single person living alone in a large apartment with their own car, their own refrigerator running half empty, their own washing machine used twice a week.

None of this appears in GDP data. None of it appears in renewable energy percentages. The model assigns India's low emissions entirely to poverty. And poverty is certainly part of the explanation. But to reduce India's relationship with resources entirely to economic necessity is to miss something important about how 1.4 billion people actually live.

India's per capita emissions are low partly because India is poor. They are also low because India, at the level of culture and daily practice, has always understood something that the wealthy world is only now beginning to relearn.

The planet does not need everyone to become poorer. It needs everyone to stop treating disposal as the default.

---

## What This All Means

The machine learning model did not discover anything that economists did not already know. But it quantified it clearly and honestly.

Wealth drives emissions. Renewables reduce them. Everything else is secondary.

That means the climate problem is fundamentally an economic problem. It is about who gets to consume how much, how that consumption is powered, and who bears the cost of changing it. Countries that industrialized early, became wealthy on cheap fossil fuels, and now have the capital to build clean energy systems are in a very different position than countries that are still developing and need energy to do it.

This is why it is very important on how nations when they meet for climate action summit discuss their agendas. It's unfair to propose a common solution to tackle this problem when each nation has it's own needs and disadvantages. 
Economically well nations can spend money on renewable energy, recycle things but countries in Africa cannot invest same.

The data does not assign blame. But it does make the picture harder to look away from.

---

*Article 5 publishes on June 5, World Environment Day. The final piece brings everything together.*

*What does this analysis change about how you think about climate responsibility? Drop it in the comments.*

---

*Data: World Bank, IRENA, IEA, Our World in Data (2022)*
*Code: [Jupyter Notebook on GitHub](https://github.com/NIKHCODE/climate-signal-lab/blob/main/notebooks/article4_shap.ipynb)*
*Series: Climate Signal Lab, 5 articles for World Environment Day 2026*
