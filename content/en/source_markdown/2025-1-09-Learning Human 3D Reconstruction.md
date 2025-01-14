---
title: "(Learning) Human 3D reconstruction"
date: 2025-01-09T00:07:13-08:00
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





### 3D Human Body Reconstruction Task

Compared with perception tasks, 3D reconstruction tasks are opposite in terms of input and output. In most cases, for 3D reconstruction tasks, after having a semantic understanding of the scene (or person), the data that visual sensors will collect is predicted.



Take the 3D human body reconstruction task as an example, which predicts the shape of human soft tissues based on known actions. First, the reconstruction result is decomposed into a quantifiable mathematical model. This mathematical model consists of human joints and soft tissues. The movement of joints is driven by bones and can be regarded as the movement of a rigid body. However, the shape of soft tissues changes with the change of human body postures, which is a complex prediction process.



### Skinning Reconstruction Method

What I am pre - researching is the skinning reconstruction method. It models soft tissues as many vertices, and these vertices have an attachment relationship with joints. Therefore, the 3D human body reconstruction task is transformed into predicting the 3D positions of multiple vertices. And the 3D reconstruction model becomes an inference model that takes a person's movement posture and body shape as input, and based on the weights obtained through training, predicts the distribution of soft tissues under the movement posture.



Take the work of SMPL as an example. The human body is divided into 6890 vertices and 23 joint points. The output of the model is the 3D coordinates of 6890 vertices, represented as a tensor with a size of 6890×3.



#### Linear Blend Skinning


$$
\overline{\mathbf{v}}_{c}=\sum_{i = 1}^{n} w_{i} M_{i,c} M_{i,d}^{-1} \mathbf{v}_{d}
$$



$V_d$ is the vertex coordinates in the resting pose, which is a 6890×3 matrix.,

$M_{i,d}^{-1} $is the transformation matrix that converts vertex coordinates to joint in the static state _

$M_{i,c}$is the transformation matrix of the joint point relative to the rest - pose under the influence of the pose.

$w_{i}$is the weight of each joint point for vertex movement.

## SMPL
Therefore, researchers split this end - to - end inference process into multiple networks by introducing prior experience, making some networks of the model universal:
① For example, the results of all modeling are 6890 vertices. If all models stand in the rest pose, the distribution of these vertices is very similar.
② For example, if all models perform the same action, the soft tissues of the models will have similar deformations, and the vertex offsets are also similar.
③ When performing the same action, the skinning weights are similar.



After decomposition, the output of 3D reconstruction can be regarded as multiple offsets applied to a basic T - shaped skinning model. These offsets include:
① The offset of vertices caused by the model's body shape.
② The vertex offset brought about by the deformation of soft tissues caused by the model's action.
③ The vertex offset brought about by the rigid - body rotation of bones caused by the model's action.



The following figure shows the impact of adding three offsets on the reconstruction result:

<figure style="text-align: center;">
    <img src="\researchimages\Breakdownthemodel.png" alt="Pic demonstrate offset" style="width: 600px; height: 240px;">
    <figcaption>Pic Demonstrate Offset</figcaption>
    <figcaption style="font-size: 10px;">Pictrue from SMPL: A skinned multi-person linear model. In Seminal Graphics Papers: Pushing the Boundaries, Volume 2 (pp. 851-866).</figcaption>
</figure>


#### Input Parameters of the Model



$\beta$ is the body shape parameter of the model. After applying PCA to the scan data of samples of various shapes in the multi - shape dataset, ten principal component eigenvectors are obtained. And based on this, the data of new models is projected onto the eigenvectors.、



$\theta$ is the difference in rotation angles between joint coordinate systems, consisting of 72 numbers. 69 of them are the offsets of each joint point coordinate system relative to the parent joint point coordinate system, and the other 3 numbers are the offsets of the root node coordinate system relative to the world coordinate system. Each node is connected to a body part.



#### Intermediate Variables



$B_S(\vec{\beta})$ is the vertex offset caused by the body shape parameter, also Principal Component Scores

$B_P(\vec{\theta})$ represents the vertex offset brought about by the pose.，

$R_{n}(\vec{\theta})$represents the rotation matrix of each node relative to the parent node. Its size is 3×3×k, and when flattened, it is 9×k.

$R_{n}(\vec{\theta}^{*})$represents the rotation matrix at rest pose

$T_P$represents the position of vertices on the "corrected rest pose" after being corrected by the blend shape parameter and pose, which is also the reconstructed data after PCA processing.



#### Weights of the Model

$S_n$ is the eigenvector obtained through PCA on the multi_shape dataset.

$P_n$ is used to weight the offset of the pose on the blend shape. The matrix size is 3N×|R| = 3N×9×k.

$J$ is used to regress from the vertex position to the joint point position.

$W$  is the skinning weight.

$T$  is the vertex distribution in the rest pose and is also the mean value in PCA.



#### Inference Pipeline

##### **Step 1: Reconstruct the rest pose of SMPL**

First, calculate the Principal Component Scores. The result can be regarded as the offset of the body shape to the vertices in the rest pose.


$$
B_{S}(\vec{\beta}) = \sum_{n = 1}^{|\beta| = 300} \beta_{n} S_{n}, \text{ where } S_{n} \in R^{3N\times300}
$$


The vertices in the rest pose after body - shape correction are


$$
T_{S} = T + B_{S}(\vec{\beta}), \text{ where } T \in R^{3N}
$$


At the same time, according to the corrected rest pose, the positions of joint points can be inferred.


$$
J(\vec{\beta}; \mathcal{J}, \overline{\mathbf{T}}, \mathcal{S}) = \mathcal{J}(\overline{\mathbf{T}} + B_S(\vec{\beta}; \mathcal{S}))
$$



##### **Step 2: Calculate the impact on vertices under a new pose. However, this is the vertex offset made on the rest pose.**


$$
\overline{\mathbf{t}}_i + \sum_{m = 1}^{|\vec{\beta}|} \beta_m \mathbf{s}_{m,i} + \sum_{n = 1}^{9K} (R_n(\vec{\theta}) - R_n(\vec{\theta}^*)) \mathbf{p}_{n,i}
$$



##### **Step 3: Skinning according to weights**


$$
\mathbf{t}'_i = \sum_{k = 1}^{K} w_{k,i} G'_k(\vec{\theta}, J(\vec{\beta}; \mathcal{J}, \overline{\mathbf{T}}, \mathcal{S})) \mathbf{t}_{P,i}(\vec{\beta}, \vec{\theta}; \overline{\mathbf{T}}, \mathcal{S}, \mathcal{P})
$$

### Training Strategy

JWP is trained using the Multi - Pose dataset. T and S are obtained by performing PCA on the Multi - Shape dataset.



#### Training of J, W, P

At this point, T and S are not learned. Therefore, for each sample in the dataset, the vertex position in the rest pose is directly measured and recorded as $\hat{\mathbf{T}}_{i}^{P}$. In addition, the joint points of each sample are also measured and recorded as $\hat{\mathbf{J}}_{i}^{P}$



The first loss is the end - to - end loss, that is, the sum of the squares of the Euclidean distances between the vertex positions after SMPL reconstruction and the actual vertex positions.
$$
\sum_{j = 1}^{P_{\text{reg}}} \left\lVert \mathbf{V}_j^P - W(\hat{\mathbf{T}}_{s(j)}^P + B_P(\vec{\theta}_j; \mathcal{P}), \hat{\mathbf{J}}_{s(j)}^P, \vec{\theta}_j, \mathcal{W}) \right\rVert^2
$$



The second loss is the sample - balance loss. If the left - right symmetry of vertices and joint points is good, the loss will be lower.



$$
E_Y(\hat{\mathbf{J}}^P, \hat{\mathbf{T}}^P) = \sum_{i = 1}^{P_{\text{subj}}} \lambda_U \left\lVert \hat{\mathbf{J}}_i^P - U(\hat{\mathbf{J}}_i^P) \right\rVert^2 + \left\lVert \hat{\mathbf{T}}_i^P - U(\hat{\mathbf{T}}_i^P) \right\rVert^2
$$


The third loss is for the J weight.


$$
E_J(\hat{\mathbf{T}}^P, \hat{\mathbf{J}}^P) = \sum_{i = 1}^{P_{\text{subj}}} \left\lVert \mathcal{J}_I \hat{\mathbf{T}}_i^P - \hat{\mathbf{J}}_i^P \right\rVert^2
$$


The last two losses are regularization losses.


$$
E_P(\mathcal{P}) = \|\mathcal{P}\|_F^2
$$

$$
E_W(\mathcal{W}) = \|\mathcal{W} - \mathcal{W}_I\|_F^2
$$

### Obtaining S and T Using PCA

The author normalized the results of the Multi - pose dataset to the rest pose and calculated the average vertex position $\hat{\mathbf{T}}_{\mu}^{P}$ and the average joint position $\hat{\mathbf{J}}_{\mu}^{P}$ .



For each sample in the multi - shape dataset, the author input $\hat{\mathbf{T}}_{\mu}^{P}$ and $\hat{\mathbf{J}}_{\mu}^{P}$, and took $\theta$ that minimizes the vertex offset as the input.



$$
\arg\min_{\vec{\theta}} \sum_{e} \left\|W_e(\hat{\mathbf{T}}_{\mu}^{P}+B_{P}(\vec{\theta};\mathcal{P}),\hat{\mathbf{J}}_{\mu}^{P},\vec{\theta},\mathcal{W})-\mathbf{V}_{j,e}^{S}\right\|^2
$$


After confirming the input $\theta$, the optimal vertex position in the rest - pose can be calculated.


$$
\hat{\mathbf{T}}_{j}^{S} = \arg\min_{\hat{\mathbf{T}}} \left\|W(\hat{\mathbf{T}} + B_{P}(\vec{\theta}_{j};\mathcal{P}),\mathcal{J}\hat{\mathbf{T}},\vec{\theta}_{j},\mathcal{W}) - \mathbf{V}_{j}^{S}\right\|^2
$$


Subsequently, perform principal component analysis on the rest - pose vertex positions of all samples in the Multi - Shape dataset, take the mean value as the final result $T$, and calculate the eigenvector $S$.



### Reference

Loper, M., Mahmood, N., Romero, J., Pons-Moll, G., & Black, M. J. (2015). SMPL: A Skinned Multi-Person Linear Model. *ACM Transactions on Graphics*, *34*(6).
