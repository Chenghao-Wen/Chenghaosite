---
title: "LiDAR Panoptic Segmentation"
date: 2024-12-03 T18:37:57-08:00
author: Chenghao Wen
slug: second-post
draft: false
toc: true
categories:
  - test
tags:
  - article
  - English
---






# LiDAR Panoptic Segmentation


## What makes LiDAR Panoptic Segmentation Important

When drivers operate vehicles, their decisions are shaped by the driving environment. This environment can be broadly categorized into two parts: one is the static road conditions, such as terrain, vegetation, and traffic lights, which are referred to as 'Stuff' in panoptic segmentation tasks; the other comprises dynamic elements , such as pedestrians, trucks, bicycles, and so on, designated as 'Thing' in the same context.

<div style="display: flex; justify-content: center; align-items: center; gap: 50px;">
  <figure style="text-align: center;">
    <img src="\images\Bus.png" alt="Pic Bus" style="width: 300px; height: 200px;">
    <figcaption>Bus is Dynamic 'Thing'</figcaption>
  </figure>
  <figure style="text-align: center;">
    <img src="\images\Trunk.png" alt="Pic Trunk" style="width: 300px; height: 200px;">
    <figcaption>Trunk is Static 'Stuff'</figcaption>
  </figure>
</div>


Perception tasks in autonomous driving scenes encompass semantic segmentation and instance segmentation (object recognition). Let us illustrate the focal points of these two tasks from the perspective of supervised labels, using the perception of point cloud data in autonomous driving scenarios as an example. 



Semantic segmentation labels for points denote the semantic categories of 'Thing' and 'Stuff' points, with common categories on road including vehicles, pedestrians, vegetation, traffic lights. This implies that completing this task merely requires identifying the boundaries of each category. 

<figure style="text-align: center;">
    <img src="\images\Semantic3.png" alt="LiDAR Semantic Segmentation">
    <figcaption>LiDAR Semantic Segmentation</figcaption>
</figure>







In comparison, instance segmentation labels are only valid for dynamic 'Thing' points, indicating both the categories of 'Thing' and assigning unique identifiers to different objects, such as Car A, Car B, and Bike C, which might be labeled as one, two, and three, respectively. Therefore, instance segmentation, initially regarded as a downstream task of object recognition, focuses on locating different 'Things', or their centers.

<figure style="text-align: center;">
    <img src="\images\Instance3.png" alt="LiDAR Instance Segmentation">
    <figcaption>LiDAR Instance Segmentation</figcaption>
</figure>





Panoptic segmentation can be viewed as a multi-task learning approach combining instance and semantic segmentation, capable of perceiving the roadway and distinguishing vehicles with varying speeds. Utilizing a unified feature extraction network to accomplish both tasks in a single inference ensures real-time processing, which is the significance of panoptic segmentation.
## What Is the Problem Remaining to be Solved

## Our Proposed Method