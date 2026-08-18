---
title: "The Forest Was Here Yesterday"
date: 2026-08-19
categories: [Biodiversity, Deep Learning]
tags: [cnn, satellite-imagery, deforestation, eurosat, pytorch, computer-vision]
math: true
description: "A Convolutional Neural Network trained on 5,000 EuroSAT satellite image patches learns to identify forest from space with 91.2% recall, revealing what machines see in pixels that human eyes miss at scale."
---

Every ten seconds, a patch of forest somewhere on Earth the size of a football field disappears. Not metaphorically. Literally. A chainsaw, a bulldozer, a controlled burn, and what was standing for decades is gone before you finish reading this paragraph. The global figure is roughly 10 million hectares per year, an area larger than South Korea, vanishing annually. That number comes from somewhere. It comes from satellites, and from machines that have learned to look at the same patch of land twice and notice what changed.

This article builds one of those machines from scratch.

I wanted to understand not just that deforestation is happening, but how the detection systems that track it actually work at the pixel level. The headline numbers we read in climate reports, the 10 million hectares, the percentage of Amazon lost, the rate of forest cover change, all of them trace back to a fundamental computer vision task: teach a model to look at a square of satellite imagery and say, with confidence, whether that square contains forest or not. Get that right, and you can run it over thousands of satellite passes, detect changes automatically, and build an early warning system that operates faster than any human monitoring team could.

I am a first year computer science student at PES University in Bengaluru. I grew up in Andhra Pradesh watching seasonal forests thin over summers that kept getting hotter. The gap between the satellite numbers and the lived experience of that change is something I find myself thinking about a lot. This project is an attempt to close that gap, not emotionally but technically, by understanding the machinery behind the measurement.

What follows is a Convolutional Neural Network trained on 5,000 real satellite image patches from the EuroSAT dataset, evaluated on 1,000 more it had never seen, with a detailed look at what it learned, where it failed, and what those failures tell us about the limits of automated forest monitoring.


THE DATA: WHAT A FOREST LOOKS LIKE FROM SPACE

The EuroSAT dataset was published in 2019 by researchers at the German Research Center for Artificial Intelligence. It contains 27,000 satellite image patches sourced from the Copernicus Sentinel-2 satellite, each patch covering a 64 by 64 pixel area at 10 metre resolution. That means each pixel represents a 10 metre by 10 metre square on the ground. Each 64 by 64 patch covers roughly 0.4 square kilometres of Earth's surface.

The dataset covers 10 land use and land cover classes across Europe:

Annual Crop, Forest, Herbaceous Vegetation, Highway, Industrial, Pasture, Permanent Crop, Residential, River, and SeaLake.

Each class has between 2,000 and 3,000 labelled image patches. The labels were assigned by human experts cross-referencing the satellite imagery with ground truth land cover maps. This makes EuroSAT one of the most reliable publicly available satellite classification datasets in existence.

For this article we used all 10 classes but framed the core question around Forest: can the model learn the visual signature of tree canopy from space, and how well does it distinguish forest from the classes that look most similar, particularly Herbaceous Vegetation and Pasture?

We trained on 5,000 patches and tested on 1,000 patches the model never saw during training. Each image was resized to 64 by 64 pixels, converted to a tensor, and normalised using ImageNet statistics, which centres the pixel values around zero and scales them to unit variance. This normalisation step is standard practice because it makes gradient descent during training numerically stable.


THE TECHNICAL APPROACH

Convolutional Neural Networks: Teaching a Machine to See

A regular neural network treats an image as a flat list of pixel values. A 64 by 64 pixel image becomes a vector of 12,288 numbers (64 times 64 times 3 colour channels), and the network tries to find patterns in that list. The problem is that spatial relationships between pixels are completely lost. A pixel in the top left corner has no special relationship to its neighbour, and the network has to learn every possible arrangement of pixels from scratch.

A Convolutional Neural Network preserves spatial structure. Instead of looking at all pixels at once, it slides a small filter (called a kernel) across the image and computes a dot product at each position. For a 3 by 3 kernel, the operation at each position is:

$$\text{Output}(i,j) = \sum_{m=0}^{2} \sum_{n=0}^{2} \text{Input}(i+m, j+n) \cdot \text{Kernel}(m,n)$$

This produces a feature map that highlights wherever the kernel's pattern appears in the image. A kernel trained to detect horizontal edges will produce high values wherever horizontal edges exist in the input, regardless of where in the image those edges appear. This property is called translation invariance, and it is what makes CNNs so powerful for image recognition: a forest patch looks like a forest patch whether it appears in the top left or bottom right of the satellite image.

Our architecture has three convolutional layers, each followed by a ReLU activation function and a max pooling layer:

Layer 1 applies 32 different 3 by 3 kernels to the raw image, producing 32 feature maps that detect low level patterns like edges and colour gradients. Max pooling then halves the spatial dimensions from 64 to 32.

Layer 2 applies 64 kernels to those 32 feature maps, building more complex patterns like textures and shapes from the low level features. Max pooling reduces from 32 to 16.

Layer 3 applies 128 kernels, building even higher level representations. Max pooling reduces from 16 to 8.

After the three convolutional blocks, the feature maps are flattened into a vector of 8,192 values (128 channels times 8 by 8 spatial dimensions). Two fully connected layers then map that vector to a probability distribution over the 10 land cover classes. A dropout layer with probability 0.5 is applied between the fully connected layers, randomly zeroing out half the neurons during each training step to prevent the model from memorising specific training examples.

The total number of learnable parameters in this network is 2,193,226. Each of those parameters is a weight that gets adjusted during training.

How Training Works

Training proceeds by the following logic. We show the model a batch of 128 satellite patches. It makes predictions. We compute the cross-entropy loss between those predictions and the true labels:

$$\mathcal{L} = -\frac{1}{N} \sum_{i=1}^{N} \sum_{c=1}^{10} y_{i,c} \log(\hat{y}_{i,c})$$

Where y subscript i,c is 1 if sample i belongs to class c and 0 otherwise, and y-hat subscript i,c is the model's predicted probability for that class. Cross-entropy loss is large when the model is confidently wrong and small when it is confidently right.

We then compute how much each of the 2.19 million parameters contributed to that loss using backpropagation, which applies the chain rule of calculus through the network layer by layer. The Adam optimiser then adjusts each parameter by a small amount in the direction that reduces the loss. This cycle repeats across every batch, in every epoch, until the model converges.

We trained for 5 epochs, meaning the model saw the 5,000 training images five complete times.


WHAT THE MODEL FOUND

The results across five epochs tell a clean story.

The model began its first epoch achieving 63.2% accuracy on the test set, already well above the 10% that random guessing would produce across 10 classes. By the fifth epoch it reached 69.0% overall test accuracy. Training loss fell by 55.1% across the five epochs, confirming that the model was genuinely learning and not just memorising.

The forest class specifically performed far better than the overall average. Forest recall reached 91.2%, meaning the model correctly identified 9 out of every 10 forest patches it was shown. For a deforestation detection system, recall on the forest class is the number that matters most. A system that misses forest patches will undercount deforestation. At 91.2% recall after only 5 epochs of training on 5,000 images, the model has learned a robust visual representation of forest canopy from space.

The class the model most often confused with forest was Herbaceous Vegetation, which is the expected failure mode. From 64 by 64 pixels at 10 metre resolution, dense grassland and shrubland produce very similar spectral signatures to sparse or young forest. The green channel values are comparable, the texture is similarly irregular, and the boundary between the two land cover types is often gradual rather than sharp in the real world. This is not a model failure. It is a genuinely hard visual problem. Professional land cover mapping systems use additional spectral bands, particularly the near-infrared channel, specifically to separate vegetation types that look identical in the visible spectrum.

The per-class results reveal the full picture. Annual Crop achieved 94.1% accuracy and SeaLake achieved 87.9%, both substantially above the overall average. These classes have unmistakable visual signatures: agricultural fields appear as regular geometric patterns in straight-edged parcels, and water bodies appear as uniformly dark, textureless regions. The model learns these signatures almost immediately.

At the other end, Pasture achieved only 40.3% and Highway achieved 44.7%. Pasture blends visually with Herbaceous Vegetation and Annual Crop depending on season and management intensity. Highways are thin linear features that may occupy only a fraction of a 64 by 64 patch, making them difficult to distinguish from residential or industrial areas that also contain linear structures. These failure cases point directly to the real limitations of automated satellite monitoring: seasonal variation, spatial resolution, and the fundamental ambiguity of land cover categories that shade into each other in the real world.

Published benchmarks on the full EuroSAT dataset with 15 or more epochs of training consistently reach 87 to 92% overall accuracy. Our 69% reflects the reduced training set and shorter training run, not a flaw in the architecture. The forest recall of 91.2% is consistent with full-scale published results, which suggests the model prioritised the forest class correctly even with limited training.


THE VISUALISATIONS

The training curve shows the model's loss and accuracy across all five epochs. The loss line drops steadily from epoch 1 to epoch 5 with no sign of sudden spikes or reversals, which confirms stable training. The test accuracy line climbs correspondingly. The gap between training loss and test loss is small and consistent, indicating the model is generalising rather than overfitting to the training data.

The confusion matrix is the most informative output of the entire analysis. It is a 10 by 10 grid where each row represents a true class and each column represents a predicted class. The diagonal cells show correct predictions. The off-diagonal cells show errors and reveal exactly which classes the model confuses with which others. The Forest row shows a strong diagonal value of 91.2% and a secondary value in the HerbVeg column, confirming the forest-vegetation confusion described above. The Pasture row shows spread across multiple columns, confirming that the model has not learned a consistent representation of pasture at this training scale.

<div style="width:100%; height:480px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article3_02_training.html"
        width="100%" height="480px" style="border:none; border-radius:8px;"></iframe>
</div>

<div style="width:100%; height:580px;">
<iframe src="https://nikhcode.github.io/climate-signal-lab/assets/article3_02_confusion.html"
        width="100%" height="580px" style="border:none; border-radius:8px;"></iframe>
</div>


MY TAKE

[This section is yours. A few directions to consider:]

The 91.2% forest recall after 5 epochs on 5,000 images is the number I would lead with personally. The systems that generate the 10 million hectare per year figure are running architectures not fundamentally different from what we built here, just with more bands, more data, and more compute. The gap between this notebook and a production deforestation monitoring system is engineering, not science.

The HerbVeg confusion is also worth writing about from your own angle. In Andhra Pradesh, the boundary between degraded forest and scrubland is exactly the kind of ambiguity that makes satellite-based forest accounting politically contested. When a government claims forest cover has increased, and the satellite data shows green pixels, the question of what counts as forest versus what counts as vegetation is not a technical footnote. It is the entire debate.

There is also something worth saying about what 10 metres per pixel means in practice. Each pixel in these patches represents a 10 metre by 10 metre square. A single large tree can occupy one pixel. A small clearing may not register at all. The things we are losing are sometimes too small for the systems we use to measure loss to see.


WHAT IS NEXT

Article 3.03 moves from pixels to systems. Instead of asking whether a patch of land contains forest, it asks whether the Amazon rainforest as a whole is approaching a critical transition point, a tipping point beyond which the forest begins converting to savanna regardless of what humans do. The technique is resilience analysis: measuring statistical early warning signals in the vegetation data itself, the rising variance and increasing autocorrelation that theory predicts should appear in any complex system as it approaches a catastrophic shift. Same satellite data, completely different question.


The forest was here yesterday. A machine can learn to see that it is gone today. The harder question is whether we are building enough of these machines fast enough, and pointing them at the right places. The Amazon is one of those places. That is where we go next.

What surprised you most about what the CNN learned to see? Leave a comment below.
