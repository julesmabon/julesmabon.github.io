---
layout: post
title:  "Mycobacteria tracking on time-lapse imaging"
subtitle: "My internship at EMBL-EBI with Virginie Uhlmann's group"
background: '/img/cells.png'
date:   2020-09-03 14:08:00 +0200
published: true
hidden: true
---

During summer 2019 I had the pleasure to do an internship with [Virginie Uhlmann](http://www.virginieuhlmann.com)'s team at the [European Bioinformatics Institute (EBI)](https://www.ebi.ac.uk) . [The team](https://www.ebi.ac.uk/research/uhlmann) aims at developing tools that blend mathematical models and image processing algorithms to quantitatively characterize the content of bioimages.

## Goal

My goal during this 6 months internship was to further develop techniques to track individual _Mycobacteria smegmatis_ cells in sequences of time-lapse microscopy images.
Since these images are challenging because of division events, compact cell colonies and little prior on individual cell shapes, classical segment-then-track methods are
ineffective. We relied on a graphical model solution to solve the tracking and segmentation problem jointly at once on the whole sequence. 
To build the graphical model, cell candidates must be identified in each individual image. To do so, we explored the use of several convolutional neural network models from U-net to discriminative losses for instance segmentation.

### _Mycobacteria smegmatis_
We were working on time lapse images of _Mycobacteria smegmatis_ that is a non pathogenic bacterial species that strongly resembles the _Mycobacterium tuberculosis_, making it a great tool to study pathogens in a safe environment. Several works {% cite Santi2013 Santie01999-14 %} study the replication mechanism of these cells and how it is linked to antibiotics resistance

<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/mycobacteria.png" alt="Mycobacteria smegmatis" style="width:75%"/>
  	<figcaption>Mycobacteria smegmatis in phase microscopy overlayed with fluorescence data {% cite ginda2017studies %}</figcaption>
  	</center>
</figure>


### The data
The data provided by Prof. John McKinney's Laboratory of Microbiology and Microtechnology (LMIC, EPFL, Lausanne, Switzerland), consists of 9 videos of 170 frames each.
Each video shows the growth of an M. smegmatis from one or two cells. 
Each frame is composed of two channels: 
one phase channel corresponding to the phase shift of the light entering the microscope,
and one fluorescence channel reporting the protein Wag31, present in the tips of the cells, which has been fused to a fluorescent dye. 
We are, in addition,provided with a set of annotation on the first 130 images on each videos. 
These annotations correspond to the skeleton (i.e., medial axis) of each cell.

### Ambiguity in single frame segmentation
The main issue with this data is that the detection relies mainly on the temporal data since on a given frame it is not obvious to the human eye where the boundaries of the cells are since cells are often cluttered.
<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/detection.png" alt="Mycobacteria smegmatis" style="width:100%"/>
  	<figcaption>Example of different detection solutions for a single image (the input image in this figure is a substitute of the raw data)</figcaption>
  	</center>
</figure>
Here an input frame with fews cells can lead to multiple hypothesis on the number and placement of cells in the frame. This shows how there is ambiguity on individual cells detection even in a few cell setup. Later in the time-lapse there can be thousands of cells at once. Therefore, because of the in-frame ambiguity, we cannot simply segment each frame and then track between frames.

From that example one can intuit that some solutions are more likely than others (i.e. solution C looks a bit off), also by using the data from the previous and next frame we may have more insight on the most probable solution. For that matter we used a graphical model to make use of all of the data available in order to solve the detection and tracking.


## Graphical models for joint Segmentation and tracking 
Schiegg et al propose a model based on factor graphs to represent the multiple detection hypothesis and jointly tracking and segmenting on whole videos at once {% cite schiegg2014graphical %}. To put it in a nutshell, given that we provide a set of **detection candidates** and **transition candidates** , along with **associated probabilities** , this method uses a graph representation to compute the most likely candidates for detections and transitions within some **constraints** . 

* **detection candidates** are potential cells (for instance the region labeled 1 in the figure, or the union of 2 and 3 labeled 23)
* **transition candidates** represent how these candidates would transition from one frame to the next. Here one could expect 12 (at frame t)to transition into 4 (at frame t+1)
* **constraints** are what makes the solution physically sound, no cell candidates can overlap (cell candidate 12 could not co-exist with 23), cells should not appear and disappear in the middle of the video etc...

<figure>
	<center>
  	<img src="{{site.url}}/img/posts/tracking_cells/graphical_model_complete.png" alt="Mycobacteria smegmatis" style="width:100%"/>
  	<figcaption>Sample graphical model for only two frames, each <b>detection candidate</b> is represented by a green node. Yellow nodes correspond to <b>transition candidates</b> between frames. <b>Constraints</b> are represented by the square nodes</figcaption>
  	</center>
</figure>




(how )
(rating how probable a cell candidate or transition is based on shape and position. i.e. cells that appear too long would have a low probability)
(i.e. cells do not appear or disappear mid-video)

To explain the technical functioning of a graphical model would take another presentation, for this time we will focus on how to generate these segmentation proposals.



<!-- 
## Previous approach

## U-net for class prediction
### what is U-net ?
### candidates generation

## Instance segmentation

## Image augmentation

## Using recurrent network to track cells trough time ?

## Acknowledgments
To the French Embassy in London for funding this internship, to EMBL for making it
possible, to José for his technical help,
And mostly, warm and sincere thanks to the whole Uhlmann Lab : Virginie, Soham, Yoann,
Johannes, James and Maria.
 -->

References
====================


{% bibliography --cited%}