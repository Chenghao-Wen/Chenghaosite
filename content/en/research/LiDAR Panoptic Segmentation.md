---
title: "LiDAR Panoptic Segmentation"
date: 2024-11-24T20:42:27-05:00
author: Chenghao
slug: first-post-cn
draft: false
toc: true
categories:
  - test
tags: article
---
# LiDAR Panoptic Segmentation



- What makes LiDAR Panoptic Segmentation Important
- What is the problem remained to be solved
- Our Proposed Method
  1. 

## What makes LiDAR Panoptic Segmentation Important

When drivers operate vehicles, their decisions are shaped by the driving environment. This environment can be broadly categorized into two parts: one is the fixed road conditions, such as terrain, vegetation, and traffic lights, which are referred to as 'Stuff' in panoptic segmentation tasks; the other comprises elements that may appear and disappear, such as pedestrians, trucks, bicycles, and so on, designated as 'Thing' in the same context.


Perception tasks in autonomous driving scenes encompass semantic segmentation and instance segmentation (object recognition). Let us illustrate the focal points of these two tasks from the perspective of supervised labels, using the perception of point cloud data in autonomous driving scenarios as an example. 

Semantic segmentation labels for points denote the semantic categories of 'Thing' and 'Stuff' points, with common categories including vehicles, pedestrians, vegetation, traffic lights. This implies that completing this task merely requires identifying the boundaries of each category. 



In comparison, instance segmentation labels are only valid for 'Thing' points, indicating both the categories of 'Thing' and assigning unique identifiers to different objects, such as Car A, Car B, and Bike C, which might be labeled as one, two, and three, respectively. Therefore, instance segmentation, initially regarded as a downstream task of object recognition, focuses on locating different 'Thing', or their centers.

Panoptic segmentation can be viewed as a multi-task learning approach combining instance and semantic segmentation, capable of perceiving the roadway and distinguishing vehicles with varying speeds. Utilizing a unified feature extraction network to accomplish both tasks in a single inference ensures real-time processing, which is the significance of panoptic segmentation.

## What Is the Problem Remained to be Solved