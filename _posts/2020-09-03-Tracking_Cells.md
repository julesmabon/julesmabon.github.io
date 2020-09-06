---
layout: post
title:  "Mycobacteria tracking on time-lapse imaging"
subtitle: "My internship at EMBL-EBI with Virginie Uhlmann's group"
background: '/img/cells.png'
date:   2020-09-03 14:08:00 +0200
published: true
hidden: false
---

During summer 2019 I had the pleasure to do an internship with [Virginie Uhlmann 's team](https://www.ebi.ac.uk/research/uhlmann) at the [European Bioinformatics Institute (EBI)](https://www.ebi.ac.uk) . The team aims at developing tools that blend mathematical models and image processing algorithms to quantitatively characterize the content of bioimages.

In this post I will discuss the methods used during the course of this internship on multiple cell tracking in time lapse microscopy images. Most of the methods described in this post were adapted from the cited publications to our case and implemented by me in Python and Tensorflow. 

Goal
====================================

My goal during this 6 months internship was to further develop techniques to track individual _Mycobacteria smegmatis_ cells in sequences of time-lapse microscopy images.
Since these images are challenging because of division events, compact cell colonies and little prior on individual cell shapes, classical segment-then-track methods are
ineffective. We relied on a graphical model solution to solve the tracking and segmentation problem jointly at once on the whole sequence. 
To build the graphical model, cell candidates must be identified in each individual image. To do so, we explored the use of several convolutional neural network models from U-net to discriminative losses for instance segmentation.

## _Mycobacteria smegmatis_
We were working on time lapse images of _Mycobacteria smegmatis_ that is a non pathogenic bacterial species that strongly resembles the _Mycobacterium tuberculosis_, making it a great tool to study pathogens in a safe environment. Several works {% cite Santi2013 Santie01999-14 %} study the replication mechanism of these cells and how it is linked to antibiotics resistance

<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/mycobacteria.png" alt="Mycobacteria smegmatis" style="width:75%"/>
  	<figcaption>Mycobacteria smegmatis in phase microscopy overlayed with fluorescence data {% cite ginda2017studies %}</figcaption>
  	</center>
</figure>


## The data
The data provided by Prof. John McKinney's Laboratory of Microbiology and Microtechnology (LMIC, EPFL, Lausanne, Switzerland), consists of 9 videos of 170 frames each.
Each video shows the growth of an M. smegmatis from one or two cells. 
Each frame is composed of two channels: 
one phase channel corresponding to the phase shift of the light entering the microscope,
and one fluorescence channel reporting the protein Wag31, present in the tips of the cells, which has been fused to a fluorescent dye. 
We are, in addition,provided with a set of annotation on the first 130 images on each videos. 
These annotations correspond to the skeleton (i.e., medial axis) of each cell.

## Ambiguity in single frame segmentation
The main issue with this data is that the detection relies mainly on the temporal data since on a given frame it is not obvious to the human eye where the boundaries of the cells are since cells are often cluttered.
<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/detection.png" alt="Detection ambiguity" style="width:100%"/>
  	<figcaption>Example of different detection solutions for a single image (the input image in this figure is a substitute of the raw data)</figcaption>
  	</center>
</figure>
Here an input frame with fews cells can lead to multiple hypothesis on the number and placement of cells in the frame. This shows how there is ambiguity on individual cells detection even in a few cell setup. Later in the time-lapse there can be thousands of cells at once. Therefore, because of the in-frame ambiguity, we cannot simply segment each frame and then track between frames.

From that example one can intuit that some solutions are more likely than others (i.e. solution C looks a bit off), also by using the data from the previous and next frame we may have more insight on the most probable solution. For that matter we used a graphical model to make use of all of the data available in order to solve the detection and tracking.


Graphical models for joint Segmentation and tracking 
====================================


Schiegg et al propose a model based on factor graphs to represent the multiple detection hypothesis and jointly tracking and segmenting on whole videos at once {% cite schiegg2014graphical %}. To put it in a nutshell, given that we provide a set of **detection candidates** and **transition candidates** , along with **associated probabilities** , this method uses a graph representation to compute the most likely candidates for detections and transitions within some **constraints** . 

* **detection candidates** are potential cells (for instance the region labeled 1 in the figure, or the union of 2 and 3 labeled 23)
* **transition candidates** represent how these candidates would transition from one frame to the next. Here one could expect 12 (at frame t)to transition into 4 (at frame t+1)
* **constraints** are what makes the solution physically sound, no cell candidates can overlap (cell candidate 12 could not co-exist with 23), cells should not appear and disappear in the middle of the video etc...

<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/graphical_model_complete.png" alt="A graphical model" style="width:100%"/>
  	<figcaption>Sample graphical model for only two frames, each <b>detection candidate</b> is represented by a green node. Yellow nodes correspond to <b>transition candidates</b> between frames. <b>Constraints</b> are represented by the square nodes</figcaption>
  	</center>
</figure>

To explain the technical functioning of a graphical model would take another presentation, for this time we will focus on how to generate these segmentation proposals. Also this part of the graphical inference model had already been implemented by C. Haubold {% cite haubold2017scalable %}.


## Graphical model model limitation
from the previous example we can expect the graph to grow exponentially as the number cell proposal augments. Thus it is important to generate a set of segmentation proposals that is wide enough to contain the ground truth but small enough not to make the complexity explode.


Previous approach
====================================
<!-- TODO -->
By Virginie Uhlmann {% cite uhlmann2017landmark %}, based on splines, detect cell tips, generate all paths between cells. Issue is great amount of detection thus explosion of the graphs complexity.

<figure>
  <center>
    <img src="{{site.url}}/img/posts/tracking_cells/uhlmann-prev-appr.jpg" alt="Uhlmann previous approach" style="width:100%"/>
    <figcaption>1 : identify cell tips with pixel classifier, 2 and 3 : model all possible tip to tip links as a shortest path relying on splines {% cite uhlmann2017landmark %}</figcaption>
    </center>
</figure>

_________________________________


# Exploring candidates generation through deep learning
<!-- TODO -->
Leverage convolutional neural networks to generate cells candidates
## First approach : U-net for pixel class prediction
### what is a U-net ?
<!-- TODO -->
 {% cite RonnebergerFB15 %}

per pixel classification, pixel prediciton is context aware (sees around) but prediction is detailed (ie output class map has same spatial definition as input)

<!-- TODO -->
 conv
 pool (scale down)
 upconv (scale up)
 skip (transfer)
<figure>
  <center>
    <img src="{{site.url}}/img/posts/tracking_cells/u-net-architecture.png" alt="Unet Architecture" style="width:100%"/>
    <figcaption>Structure of a basic U-net {% cite RonnebergerFB15 %}</figcaption>
    </center>
</figure>

<!-- TODO -->
idea detect cells and separate different cell segments using the tips of the cells as delimiters. added core class and outer cell class to enforce separation between different instances of touching cells.

however these are really not cells candidates but rather a bunch of cell segments



### Weighted soft-max cross-entropy loss

We use a Cross entropy loss with soft-max as it is commonly used for classification {% cite Bishop:2006:PRM:1162264 %} and can be writen as

$$
l(I) := - \sum_{x\in I}w(x) \log 
\frac{
\exp \left(    \hat{y}_{y(x)}(x)  \right)
}{
\sum_{k=0}^K\exp \left(    \hat{y}_{k}(x)  \right)
},
$$

with $$x$$ a pixel in the image domain $$I$$, $$\hat{y}_k:I\rightarrow\mathbb{R}$$ the predicted score for class $$k$$, $$K$$ the number of classes, and $$ y:I\rightarrow\{0,\dots,K\}$$ the ground truth segmentation. Thus, $$\hat{y}_{y(x)}(x)$$ corresponds to the predicted score for ground-truth class $$y(x)$$ at position $$x$$. The $$w(x)$$ is a per-pixel weighting term used to handle class imbalance and instance separation, and is defined as

$$
    w:=w_\mathrm{bal} +w_\mathrm{sep}
$$

#### Learning instance segmentation

To learn how to segment instances, labels of different instances must be separated by at least one pixel of background, ensuring that each instance is one single connected component. In order for this gap to be predicted correctly by the network, a weighting term $$w_\mathrm{sep}$$ is applied on the loss to further penalise errors in boundary areas as

$$
    w_\mathrm{sep}(x) := \exp \left( -\frac{(d_1(x)+d_2(x)}{2\sigma^2}\right),
$$

with $$d_1$$ and $$d_2$$ the distances to the two nearest instances. This weight therefore increases at locations where two cells are close together, enforcing a separation. The next figure shows an illustration of  $$w_\mathrm{sep}(x)$$ over an image.

#### Pixel class prediction

Our first objective is to distinguish cells from background, thus we use a background class and a cell class. However individual cells need to be spatially separated by a gap in order to distinguish different entities, hence we introduce an **inner cell** and **outer cell** class in order to have separate cell core between different cells and also not to confuse the classifier by labeling the outer cells as background. Since the cell tips will be a useful information to separate individual cells afterwards we also introduce a **cell tip** class.

<figure>
  <center>
    <img src="{{site.url}}/img/posts/tracking_cells/unet-train-weighted.png" alt="Unet training" style="width:100%"/>
    <figcaption>Training the U-net with our data.</figcaption>
    </center>
</figure>

Once the U-net is trained, the network outputs probability maps for each class.

### Candidates generation

From output of the network we need to produce a set of cell candidates. We devised a method of _over-segmentation_ to get a set of cell parts that could be assembled to make cell candidates. Basically, from the separated **inner cells**, we run a watershed algorithm to propagate labels on the cells. We get the results shown below on a sample image. From the over-segmentation output we can build a set of cell proposals. We build proposals from the cell segments by proposing every segment and every union of touching segments (within set limit amount, so that cells aren't too big).

<figure>
  <center>
    <img src="{{site.url}}/img/posts/tracking_cells/candidates.png" alt="Candidates generation" style="width:100%"/>
    <figcaption>Building candidates from the predicted pixel classes. Left : the predicted pixel classes. Center : cell segments built from the pixel classification. Right : set of cell candidates built from the available cell segments.</figcaption>
    </center>
</figure>


_________________________________

## Instance segmentation with pixel embedding
In the first approach the instance segmentation (differentiating one cells from its neighbor) is done by the proxy of per pixel classification. What if we could directly optimize for instance segmentation within the convolutional network and thus minimize the handcrafting of the cell proposal method.

each pixel is attributed a N dimensional vector, so that every pixel from a same cell has a "similar" vector and pixels from different cells have "non-similar" vectors (the similarity measure may depend on the method used). If N=3 you can consider these vectors to be RGB colors, the objective being so that each cell is colored uniformly but different cells have different colors. Through the different figures we project our N dimensional vectors to a 3D space with a PCA so that we can display these vectors as colors. Using the first two dimensions of the PCA we can plot these pixels as points within this 2D space to better visualize the clustering.

### Semantic InstanceSegmentation with a Discriminative Loss Function
{% cite deBrabandere2017semantic %}
#### discriminative loss
#### clustering

--results--

### Customizing the loss for our purpose ?
<!-- TODO -->
ie cell division, to identify parenting

## Using recurrent network to track cells trough time ?
<!-- TODO -->

Conclusion
====================================

<!-- TODO -->

## Acknowledgments
To the French Embassy in London for funding this internship, to EMBL for making it
possible, to José for his technical help,
And mostly, warm and sincere thanks to the whole Uhlmann Lab : Virginie, Soham, Yoann,
Johannes, James and Maria.


References
====================


{% bibliography --cited%}