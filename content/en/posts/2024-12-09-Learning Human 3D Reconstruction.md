---
title: "(Learning) Human 3D reconstruction"
date: 2024-12-09T00:07:13-08:00
author: Chenghao Wen
slug: first-post
draft: false
toc: true
categories:
  - test
tags:
  - article
  - English
---


The Task:

Compared to perception tasks, three-dimensional (3D) reconstruction tasks are opposite in terms of input and output. Most 3D reconstruction tasks involve taking semantic understanding of a scene (or person) as input, after already having achieved such understanding, to predict how data will be collected by sensors. If the outcome of training a perception model is akin to a tourist, then the result of training a reconstruction model is akin to a painter.

Taking 3D human body reconstruction as an example, the problem it aims to solve is predicting the position and shape variations of human soft tissues based on known actions. 3D human body reconstruction first decomposes the reconstruction result into a quantifiable mathematical model, which consists of human joints and soft tissues. During human movement, the position and orientation changes of bones are easy to describe because they can be treated as rigid bodies. However, changes in soft tissues are often difficult to describe, and human soft tissues play a decisive role in visual perception.

Among the approaches to modeling human soft tissues, the one I am pre-researching is the blend skin approach, which models soft tissues into multiple vertex patches, and each soft tissue has an attachment relationship with certain joints. Therefore, the task of 3D human body reconstruction is transformed into the task of predicting multiple soft tissues, and the 3D reconstruction model becomes an inferential model that takes a person's motion posture as input, and based on existing human soft tissue parameters and skeletal parameters, predicts the distribution of soft tissues under the given motion posture.

The modeling of this inferential model can be divided into several components, including: