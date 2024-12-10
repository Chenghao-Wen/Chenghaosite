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

## Understanding the Task

Compared to perception tasks,3D reconstruction tasks are opposite in terms of input and output. Most 3D reconstruction tasks involve taking the semantic understanding of a scene or person as input, to predict how data will be collected by sensors or eyes as the output. If the outcome of training a perception model is to regress the model to a **viewer**, then the result of training a reconstruction model is to regress a **painter**.

Taking 3D human body reconstruction as an example, the problem it aims to solve is predicting the position and shape variations of human soft tissues based on known actions. It decomposes the reconstruction result into a quantifiable mathematical model, which consists of human joints and soft tissues. During human movement, the position and orientation changes of bones are easy to describe because they can be treated as rigid bodies. However, changes in soft tissues are often difficult to describe, and human soft tissues play a decisive role in visual perception.

## Blend Skinned Approach

Among the approaches to modeling human soft tissues, the one I am pre-researching is the blend skin approach, which models soft tissues into multiple vertex patches, and each soft tissue has an attachment relationship with certain joints. Therefore, the task of 3D human body reconstruction is transformed into the task of predicting multiple soft tissues, and the 3D reconstruction model becomes an inferential model that takes a person's motion posture as input and is based on existing human soft tissue parameters and skeletal parameters, predicts the distribution of soft tissues under the given motion posture.


From a quantitative perspective, taking the SMPL model as an example, the human body is divided into 6,890 vertices and 23 joints. In other words, the model predicts the coordinates of these 6,890 vertices, represented as a tensor of size 6890×3.

Next, the challenge is to build a model that outputs a tensor of size 6890×3. A natural approach would be to use a sensor device worn by the same person, which has 6,890 vertices, to capture data from various actions, resulting in, for example, 2,000 samples. The model input would then consist of the 23 joint geometry data and the actions performed by the model, followed by constructing a fully connected neural network. After training for 100 epochs, a reasonably accurate model could be obtained.

However, this intuitive and end-to-end solution has a problem: for each individual model, the trained model would be different, meaning it lacks generalizability.

## SMPL
Therefore, researchers introduced prior knowledge to decompose this end-to-end inference process into multiple networks, allowing some of the model's networks to have generalizability:

For example, all models result in 6,890 vertices, and if all the models are standing in a T-pose, the distribution of these vertices will be very similar.
If all models perform the same action, the soft tissue deformation will be similar, and the trajectory of vertex displacement will also be similar.
When performing the same action, the changes in the skeletal positions are similar, leading to comparable rotation and offset effects on the vertices.
After decomposition, the output of the 3D reconstruction can be considered as multiple offsets applied to a base T-pose mesh model. These offsets include:

The displacement of vertices due to the model's body shape.
The displacement of vertices caused by soft tissue deformation from the model’s actions.
The displacement of vertices due to the rigid body rotation of the skeleton caused by the model’s actions.

The following figure illustrates the impact of these three offsets on the reconstruction results:

<figure style="text-align: center;">
    <img src="\researchimages\Breakdownthemodel.png" alt="Pic demonstrate offset" style="width: 400px; height: 240px;">
    <figcaption>Pic Demonstrate Offset</figcaption>
    <figcaption style="font-size: 5px;">Pictrue from SMPL: A skinned multi-person linear model. In Seminal Graphics Papers: Pushing the Boundaries, Volume 2 (pp. 851-866).</figcaption>
</figure>
