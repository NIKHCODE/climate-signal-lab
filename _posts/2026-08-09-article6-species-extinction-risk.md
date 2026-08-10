---
title: "The Traits That Decide Who Survives"
date: 2026-08-09
categories: [Biodiversity, Machine Learning]
tags: [iucn, random-forest, shap, extinction-risk, biodiversity, conservation]
math: true
description: " I am back with my Climate files series. In this new article we will be discussing on a Random Forest classifier trained on 104 mammal and amphibian species reveals that the number of simultaneous threats a species faces predicts extinction risk three times more powerfully than its geographic range, and ten times more powerfully than its body mass."
---

We talk about extinction here and then especially dinos one of our favourite animal since our childhood. We have movies, animations and different stories about how their extinction happened and there-after. So I personally decided to go into the list of the current age animals and birds and see if something similar is going on. Something that happens, something we read about, something that feels far enough away to be someone else's problem but the numbers sitting inside the IUCN Red List, the world's most comprehensive inventory of species conservation status, tell a different story entirely. Of the 142,500 species formally assessed for extinction risk, 40,084 are threatened. That is roughly one in three and those are only the species anyone has bothered to look at closely, which amounts to about 1.5 percent of the estimated 8 to 10 million species that share this planet with us.

Lakes I knew as a child have become seasonal. Forests that used to feel dense now feel thin. The calls of certain birds have simply stopped appearing in mornings that used to be loud with them. You do not need a dataset to know something is wrong. But you need one to know exactly what, and exactly how fast, and which species are standing closest to the edge right now.

This article is an attempt to answer a question that sits beneath the surface of every conservation headline. Not which species are threatened, because the IUCN has already told us that. The deeper question is what makes a species vulnerable in the first place. Is it body size? Range? The number of habitats it can tolerate? Whether it is a mammal or an amphibian? And can a machine learning model learn that fingerprint from species we already know are in trouble, and use it to say something meaningful about the ones we have not looked at yet?

To answer that, I built a Random Forest classifier trained on 104 real species across mammals and amphibians, each described by eight ecological and environmental traits sourced from published literature and the IUCN Red List. Then I used SHAP values to crack the model open and read exactly which traits drove each prediction. Finally, I layered an Isolation Forest on top to flag species that are statistically anomalous, regardless of what their official threat category says. The results upended what I expected to find.

The goal is not to replace the IUCN's rigorous species assessments, which involve field surveys, population counts, and decades of expertise. The goal is to demonstrate that the ecological fingerprint of extinction risk is learnable from data, and that the feature that matters most is not the one conservation campaigns spend most of their time talking about.


### THE DATA AND THE SPECIES

The dataset covers 104 species, 65 mammals and 39 amphibians, ranging from the blue whale to the axolotl, from the house mouse to the black rhinoceros. Every species carries eight features that describe its ecological situation.

Body mass tells us how large the animal is, measured in grams. It spans an extraordinary range in this dataset, from 2 grams for a poison dart frog to 100 million grams for the blue whale, which is why we apply a logarithmic transformation before feeding it to the model. Geographic range size tells us how much of the Earth's surface the species can call home, measured in square kilometres. Again, this spans orders of magnitude, from 10 square kilometres for the axolotl's last remaining lake habitat in Mexico City to 78 million square kilometres for the house mouse, which has followed human civilisation to every corner of the planet.

Population trend is simpler: a score of 0 means the population is decreasing, 1 means stable, 2 means increasing. Habitat breadth counts how many distinct major habitat types a species uses, which is a measure of ecological flexibility. A species that can live in forests, grasslands, and wetlands is far less vulnerable than one that can only survive in a single very specific environment. Threat sum counts the number of documented threat categories recorded against each species in the IUCN database, covering things like hunting, habitat loss, disease, invasive species, and climate change. Human density measures the average human population density within the species' geographic range. Cropland coverage measures what fraction of that range has been converted to agriculture. Finally, taxonomic group encodes whether a species is a mammal or an amphibian.

The target variable is binary. Species classified as Least Concern or Near Threatened are labelled Not Threatened. Species classified as Vulnerable, Endangered, or Critically Endangered are labelled Threatened. This gives us 55 species in the not threatened group and 49 in the threatened group, a near-balanced split that makes the classification task meaningful without requiring heavy artificial resampling.


### THE TECHNICAL APPROACH

Three methods working together, each answering a different version of the same question.

# Random Forest: Learning the Pattern

A single decision tree learns by asking a sequence of yes or no questions about a species' traits, splitting the data at each branch until it reaches a prediction at the leaves. The problem with one tree is that it memorises its training data. It learns the noise as well as the signal, and performs poorly on species it has never seen.

A Random Forest solves this by growing 500 different trees simultaneously, each trained on a random subset of the species and a random subset of the features. Every tree votes on whether a species is threatened, and the final prediction is the majority:

$$\hat{y} = \frac{1}{500} \sum_{i=1}^{500} T_i(x)$$

Where T subscript i of x is the prediction of the i-th tree for species x. Because each tree has seen different data and used different features, their errors are uncorrelated. Averaging across 500 trees cancels out individual mistakes and leaves behind the genuine signal in the data. This is the same intuition behind polling: one person's opinion is unreliable, but the average of 500 independent opinions is surprisingly accurate.

We trained with class weights balanced, which tells the model to penalise misclassifying a threatened species more heavily than misclassifying a safe one. This prevents the model from taking the lazy route of predicting everything as not threatened simply because that is slightly more common in the training set.

# SHAP Values: Opening the Black Box

A Random Forest makes good predictions but does not explain itself. SHAP, which stands for SHapley Additive exPlanations, fixes this by borrowing a concept from cooperative game theory called the Shapley value.

Imagine each feature as a player on a team, and the prediction is the prize the team wins. The Shapley value asks: if we consider every possible order in which the players could have joined the team, how much does each player's arrival change the team's performance on average? That average contribution across all possible orderings is the player's fair share of the prize.

The formula is:

$$\phi_i = \sum_{S \subseteq F \setminus \{i\}} \frac{|S|!(|F|-|S|-1)!}{|F|!} \left[ f(S \cup \{i\}) - f(S) \right]$$

In practice this means: for every possible subset of features that does not include feature i, calculate how much the prediction changes when you add feature i. Average all those changes, weighted appropriately. The result tells you exactly how much each feature pushed the prediction toward threatened or away from it, for every individual species in the dataset.

The SHAP bar chart in this article shows the mean absolute SHAP value across all species for each feature. A higher value means that feature, on average, changed the model's prediction more. This is not just feature importance in the traditional sense. It is a measure of causal contribution, grounded in mathematical fairness.

# Isolation Forest: Finding the Outliers

The Random Forest and SHAP tell us which species are predicted to be threatened and why. The Isolation Forest asks a completely different question: which species are simply strange, regardless of their threat status?

It works by randomly selecting a feature and a random split point, then isolating data points into progressively smaller partitions. A species with very unusual trait combinations, say, an extremely tiny range and an extremely large body, gets isolated from the rest of the data very quickly. One that sits comfortably in the middle of the distribution takes many more splits to isolate. The anomaly score is the average depth at which a species gets isolated across many random trees. A score near -0.5 flags a genuine outlier. A score near 0 is unremarkable.

The key insight is that this method knows nothing about threat categories. It learns only the structure of the trait data. When it flags a species as anomalous and that species also turns out to be threatened, the two independent methods are agreeing with each other without coordination. That convergence is meaningful.


### WHAT THE ANALYSIS FOUND

The model achieved a ROC-AUC of 0.9545 on species it had never seen during training, and a test accuracy of 95.2 percent across 21 held-out species. These are strong numbers for a dataset of this size, and they confirm that the ecological fingerprint of extinction risk is genuinely learnable from the eight features we used.

But the model's performance is secondary to what SHAP revealed about which features drove it.

The number of documented threats against a species is the single most powerful predictor of extinction risk, with a mean SHAP value of 0.234. The second most important feature, habitat breadth, has a SHAP value of 0.078. Threat count outweighs habitat breadth by a factor of three. It outweighs geographic range size by a factor of four. It outweighs body mass by a factor of eight.

This is a finding that cuts against the most common conservation narrative. The public conversation about extinction focuses heavily on large, charismatic animals. Elephants, tigers, rhinos. The implicit assumption is that being big makes you vulnerable. But the data says something more precise: what makes a species vulnerable is not its size. It is the accumulation of simultaneous pressures bearing down on it at once. A species facing hunting pressure, habitat loss, invasive competitors, disease, and climate stress simultaneously is far more at risk than a large species facing only one of those things. The axolotl weighs 150 grams. The blue whale weighs 100 million grams. Both are critically endangered or endangered. Their size is not what they have in common. Their threat profiles are.

Habitat breadth came second, which is intuitive once you think about it. A species that can use nine different major habitat types has nine fallback positions when one of them degrades. A species that can only survive in a single very specific microenvironment has none. When that environment disappears, so does the species. Specialists are structurally more fragile than generalists, and the SHAP values confirm this at scale.

Geographic range size came third, slightly ahead of population trend. This makes biological sense. A species with a tiny range is one habitat loss event, one epidemic, or one extreme weather event away from extinction. A species spread across millions of square kilometres can absorb localised disasters without the whole population collapsing. Range size is the ecological equivalent of diversification.

Body mass came fifth, well behind the four features above it. This does not mean size is irrelevant. Large animals tend to have slower reproduction rates, which means populations take longer to recover from shocks. But pure size is a weak predictor on its own. A large animal with a wide range, multiple habitat options, and few documented threats is safer than a tiny animal facing relentless pressure from every direction.

The Isolation Forest results added a second layer of confirmation. Among the 20 most anomalous species in the dataset, 55 percent are threatened, compared to 47 percent in the full dataset. The method that knows nothing about IUCN categories still finds threatened species disproportionately among the outliers. The most anomalous species in the entire dataset is Ambystoma mexicanum, the axolotl, which has a geographic range of approximately 10 square kilometres in a single lake in Mexico City, a body mass of 150 grams, and a Critically Endangered classification. Its trait combination is so extreme relative to the rest of the dataset that the model isolates it almost immediately. The second and third most anomalous species are Mus musculus and Rattus norvegicus, the house mouse and brown rat, which are outliers in the opposite direction: ranges approaching 78 million square kilometres, adaptable to virtually any habitat, and facing essentially no extinction pressure. The model correctly identifies both types of extreme, threatened and unthreateable, as statistically unusual.


### THE VISUALISATIONS

The SHAP bar chart shows the mean absolute SHAP value for each of the eight features, sorted from most to least important. The dominance of threat count is visually immediate. The bar for number of threats extends roughly three times further than the next longest bar. This is not a close race. The model is telling us, as clearly as a bar chart can, that the accumulation of simultaneous pressures is what separates threatened species from safe ones in this dataset.

The ROC curve shows how the classifier performs across all possible decision thresholds. The curve climbs steeply from the bottom left toward the top left corner before flattening along the top edge. The area under the curve is 0.9545, which means that if you presented the model with one randomly chosen threatened species and one randomly chosen safe species, it would correctly identify the threatened one as higher risk 95.45 percent of the time. The grey diagonal line represents a random classifier with no skill. Our curve stays well above it across the entire range.

Both charts are interactive. Hovering over the SHAP bars shows the exact values. The ROC curve shows the true and false positive rates at each threshold point.

<div style="width:100%; height:500px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article3_01_shap.html"
        width="100%" height="500px" style="border:none; border-radius:8px;"></iframe>
</div>

<div style="width:100%; height:500px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article3_01_roc.html"
        width="100%" height="500px" style="border:none; border-radius:8px;"></iframe>
</div>


### WHAT IS NEXT

The next article in this cluster turns from species-level risk to landscape-level change. Instead of asking which animals are closest to the edge, we will ask whether we can watch the forest disappearing in real time, pixel by pixel, using satellite imagery and a convolutional neural network trained on paired before-and-after images. The technique changes completely. The underlying question, how fast are we losing what we cannot get back, stays the same.


There is a number I keep coming back to. Not 40,084, the count of threatened species. Not 0.9545, the model's AUC. It is 98.5 percent: the fraction of life on Earth we have not yet formally assessed. The model in this article is one small attempt to say something useful about that silence. The silence itself is the real finding.

What do you think we are missing in the 98.5 percent we have not looked at yet? Leave a comment below.
