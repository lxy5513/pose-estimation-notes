## personLab 系统

#### PersonLab: Person Pose Estimation and Instance Segmentation with a Bottom-Up, Part-Based, Geometric Embedding Model 笔记



## Abstract

没有边框，自下而上的方法  SSDmodel  

group keypints into person pose instance 

Associate semantic person pixels with their corresponding person instance, delivering instance-leval person segmentation 



## Introduce

对一个混乱、拥挤的图片，我们的目标是确定每一个个体实例，定位他的面部和身体其他部位,并预测他的实例分割掩码。

两种多人检测，姿态评估的方法

- 自上而下 person first

  首先通过边框检测器粗略的界定每个人的位置，然后在每个bbox中分别对每个人进行姿态评估

  **成本较高**

- 自下而上 parts first

  首先定位语义级别的实体（关键点keypoints或者语义级别的标签），然后把它们分组成个体实例。

  **本文中使用后者** 应用没有边框的全卷积系统，它的计算代价与图片中人的个数没有关系，主要是用于backbone的特征提取



我们首先预测所有的关键点，然后学着预测对应keypoints之间的相对位移，一旦我们定位了所有的点，就把它们分组成个体本身，我们从把握最大的点开始，而不是从之前决定好的点（例如鼻子）开始.

​	

除了预测关键点，对于每一个个体，我们也预测与实例无关的个人语义分割mask map。

对于每个像素xi，我们预测xi所属的相应人的所有K个关键点的位置。然后，我们将其与所有候选检测人j（根据平均关键点距离）进行比较，并通过关键点检测概率加权; 如果该距离足够低，我们将像素i分配给人j。

​	

不需要second stage box-based refinement. easy to deploy in mobile phones 



![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-13_13-57-49.png)





## Methods

### 个人检测和姿态预测 

我们开发一种box-free、 bottom-up 方法用于个人检测和姿态预测， 它包含两个连续的步骤，首先检测出所有的关键点，然后将这些关键点分组成一个个的人体实例。使用K=17的coco数据集 



#### 关键点检测

> in an instance-agnostic fashion, all visible keypoints belong to ang person in the image 

我们混合使用分类和回归的方法。

我们生成热图（每个关键点一个通道）和偏移（每个关键点两个通道，用于水平和垂直方向的位移）

> $x_i$是这个图片的2-D位置，i=1, ..., N, N代表图片中的像素个数。
>
> $D_R(y)=\big\{ x: ||x-y|| <= R \big\}$ 是一个y为中心的圆。
>
> 另外让 $y_{j,k}$ 是第 j 个人的第 k 个关键点。j = 1, ...., M.

对于每个关键点类型k = 1,2, ..., K。 我们设置一个二分类: 我们预测一个heatmap  $ p_k(x)$  = 1如果 $x \in D_R(y_{j,k})$.    否则就等于 0. 

> R=32 in this paper, 故意将R不设为和实例大小成比例，是为了在计算分类损失时平等的对待所有不同大小的实例	

这样我们就有 **k 个独立的密集二分类**。

> 我们使用 the average logistic loss 去计算 heatmap loss 



##### PersonLab 预测如下：

- keypoints heatmaps 

  > classfication loss 

- Short-range offsets

  > purpose：to improve the keypoints localization accuracy  
  >
  > vector:  $S_k(x) = y_{j,k} − x$ （x in the keypoints disks). 
  >
  > in train: L1 loss

  aggregate the heatmap and short-range offsets via Hough voting into 2-D hough score maps $h_k(x)$ to generate **Hough arrays**

- Mid-range offsets 

  > 通过直接回归一个关键点的 heatmap keypoints disk 到另一个关键点之间的offset来建模两个关键点之间的关系，这种距离的offsets是较难学习的，在具体回归数值时会有较大的误差，用关键点周围的short-range offsets 去进一步refine对应的offsets

- Person segmentation mask 

- Long-range offsets

前三个通过Pose Estimation Module 去检测人体姿态

后两者通过Instance Segmentation Module 去预测实例分割任务





![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-13_14-45-51.png)



### **Group keypoints into person detection instances** 

**Mid-range pairwise offsets**   最大的$h_k$充当人体关键点的候选点，但是它们没有携带相关的相关于人实例信息。所以我们需要去 group keypoints into person detection instances 

> 这就是Mid-range pairwise offsets 的目的。

$M_{k,l}(x)$ designed to connect pairs of keypoints  we compute 2(K-1) such offset feilds

>  the pairwise offset filed from the k-th to the l-th keypoint is given by $M_{k,l}(x) = (y_{j,l}-x) \ [\ x \in D_R(y_{j,k})\ ]\ $   

对于同一个人对象，从第k个关键点到第l个个关键点



**Associative Embedding**: 通过使用高维向量的向量编码来编码不同个体的不同关键点之间的关系，即同一个人的不同关键点在空间上是尽可能近的。不同人的不同关键点在空间上是尽可能远的，最后可以通过两个关键点在高维空间上的距离来判断两个关键点是否属于同一个人，从而达到聚类的目的。



