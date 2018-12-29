# RMPE Regional Multi-Person Pose 

### 1 .主要三个组件：

- #### Sysmmetric Spatial Transformer Network(SSTN)

  > include: 
  >
  > Spatial Transformer Network (STN)
  >
  > Single Person Pose Estimation (SPPE)
  >
  > Spatial De-Transformer Network (SDTN) 

- #### Parametric Pose Non-Maximum-Suppression (PPNMS)

  > To address the problem of redundant detection

- #### Pose-Guided Proposals Generator (PGPG)

  > augment images

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-21_14-51-31.png)





### 2 .Two Frames for pose estimation

- #### the two-step framework 

  first detects human bounding boxes and then estimates the pose within each box independently 

  > The accuracy of pose estimation highly depends on the quality of the detected bounding boxes.

- #### The part-based framework

  first detects body parts independently and than assembles the detected body parts to form mutiple humans pose 

  > part-based framework loses the capability to recognize body parts from a global pose view due to the mere utilization(仅仅用到) of second-order body parts dependence 



### 3 . Symmetric STN and Parallel SPPN

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-21_15-16-26.png)

#### SPPN(single person pose network)

> 用于单人检测的主要网络 
>
> SPPE is seecifically trained on single person images and is very sensitive to licalisation errors, small translation or cropping of human proposal can significantly affect performance of SPPE

the **aim** of symmetric STN and Parallel SPPN is to enhance SPPE when given imperfect human proposals 

We use the stacked hourglass model as the single person pose estimator 

#### STN and SDTN

to extract high quality dominant human proposals. The spatial transformer network has demonstrated excellent performance in selecting **region of interests** automatically. 

After extracting high quality dominant human proposal regions, we can utilize 现成的 SPPE for accurate pose estimation.

SDTN is an inverse procedure of STN

#### Parallel SPPN

> help the STN focus on the correct area and extract high quality human-dominant regions
>
> the STN is trained to move the human to the center of the extracted region to facilitate accurate pose estimation by SPPE

In this branch , the SDTN is omitted 。  

the output of the branch is directly compared to the labels of this center-located GT poses. 

The **weights of this branch are fixed** and its purpose is to back-propagate center-located pose errors to the STN module

> if the extracted pose of the STN is not center-located, the parallel branch will back-propagate large error

In the testing phase, the parallel SPPE is discarded. 



### 4 . Parametric Pose NMS 

#### NMS scheme

Firstly, the most confidenct pose is selected as reference. and some poses close to it arw subject to elimination by applying elimination criterion 

#### Elimination Criterion

We define a pose distance metric $d(P_i,P_j|Λ)$ to measure the pose similarity, and a threshold  η as elimination criterion

 where Λ is a parameter set of function $d(·)$. Our elimination criterion can be written as follows: 

​	$$ f( Pi,Pj|Λ,η) =1[d(Pi,Pj|Λ,λ) ≤ η] $$  

> If d(·) is smaller than η, the output of f(·) should be 1, which indicates that pose Pi should be eliminated



### 5 . Pose-guided Proposals Genetator

#### Data Augmentation

> 在训练过程中增加proposal的数量，虽然每一张图片都只有K个人，每个人只会产生一个bbox，但是可以根据ground truth的proposals，生成和其分布相同的多个proposals一起训练。

During the training phase of the SSTN+SPPE, for each annotated pose in the training sam-
ple we first look up the corresponding atomic pose a. Then we generate additional offsets by dense sampling according to $P (δB|a)$ to produce augmented training proposals.





### 结果

VGG-base (SSD-512) as human detector, RetNet18 as localization network can achieve **mAP 76.7** on MPII

ResNet152 based Faster-RCNN ( PyraNet ) as human detector can achieve **mAP 82.1** on MPII 

### 对比

#### PPN ResNet-18 72.8  ResNet-50 75.9  ResNet-101 76.6  

#### CUM PAF 75.6 

#### Associative Embedding 77.5