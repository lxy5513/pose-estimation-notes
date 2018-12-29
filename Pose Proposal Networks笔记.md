### Pose Proposal Networks笔记



​	**本文要解决的是从2D的静态图像中检测出人体姿的速度问题， 达到实时状态**

​	

先来看下之前的解决方法，之前的方法可以分为两大类：

- 一类是首先检出人的位置，然后在人的区域内去估计姿态，这是top-down的思想；

- 另一类则是先检出关键点，再将它们合并到一个人上，这是bottom-up的思想。

  > top-down的方法计算代价更高一些，因为它和图片中人的数量成正比，这限制了这类方法在一些实时场景中的应用。而bottom-up的方法需要将关键点进行关联，并合并到单个人上的后续处理流程。



本文根据之前方法的缺点，将**姿态检测的复杂度**与**CNN的特征图分辨率**分离开来，借鉴单阶段目标检测(single-shot)的思想，将**人体姿态估计**转换为**目标检测问题**，从图像中==直接回归==出人和关键点的位置。另外不同于逐像素的关节连接方法，本文**直接从CNN输出关节连接和生成候选姿态**。



 **Extract ==grid-wise== bject confidence maps rather than ==pixel-wise== part confidence map** 



![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-04_11-37-15.png)

### 步骤

1. 将输入图像规范化，resize到统一大小；
2. 经过CNN的前向传播直接输出人的位置，人体各个部分位置，四肢的检测；
3. 对上述的候选区域使用非极大值抑制(NMS)和偶匹配（bipartite matchings）算法；
4. 将候选区域与单个人对应起来并生成候选姿态。



**our method does not need time-consuming, pixel-wise feature extraction or parsing steps**



### 一、PPN（pose proposal network）

 	The PPNs are constructed from a single CNN and produce a ==fixed-size collection of RPs==  for each detection target (person instances or each part) over the input image. 

输入图像经过CNN后被分成了H x W大小的网格，每个网格都对应着输入的**一个区域**，对于每一个网格都输出一组检测结果

### 检测结果包括两部分：

​	==$ \{B_k^i\} \ \ 和\ \ \{C_{k_1,k_2}\} $==



##### $\big\{ B_k^i \big\}	$，其中i代表网格的索引，k代表需要检测的目标索引(比如k=0代表头部)。

$$B_k^i = \big\{ p(R|k, i), p(I|R, k, i), o^i_{x,k}, o_{y,k}^i, w_k^i, h_k^i \big\}$$



> 首先将输入图像分割为![H×W](https://math.jianshu.com/math?formula=H%C3%97W)个 grid cell ，生成一系列的 bounding boxes : 　　
>  　　　　 ![\lbrace {B^i_k}\rbrace_{k\in {\cal K}} = \lbrace p(R|k,i), p(I|R, k, i), o^i_{x, k} , o^i_{y, k} , w^i_k , h^i_k \rbrace](https://math.jianshu.com/math?formula=%5Clbrace%20%7BB%5Ei_k%7D%5Crbrace_%7Bk%5Cin%20%7B%5Ccal%20K%7D%7D%20%3D%20%5Clbrace%20p(R%7Ck%2Ci)%2C%20p(I%7CR%2C%20k%2C%20i)%2C%20o%5Ei_%7Bx%2C%20k%7D%20%2C%20o%5Ei_%7By%2C%20k%7D%20%2C%20w%5Ei_k%20%2C%20h%5Ei_k%20%5Crbrace)
>  　　　　　　　　　　　　![i\in G = \lbrace 1, ..., H\times W\rbrace](https://math.jianshu.com/math?formula=i%5Cin%20G%20%3D%20%5Clbrace%201%2C%20...%2C%20H%5Ctimes%20W%5Crbrace)　　　　
>  　　　　　　　　　　　　　　![{\cal K} = \lbrace 0, 1,..., K \rbrace](https://math.jianshu.com/math?formula=%7B%5Ccal%20K%7D%20%3D%20%5Clbrace%200%2C%201%2C...%2C%20K%20%5Crbrace)
>
> > ![\lbrace {B^i_k}\rbrace_{k\in {\cal K}}](https://math.jianshu.com/math?formula=%5Clbrace%20%7BB%5Ei_k%7D%5Crbrace_%7Bk%5Cin%20%7B%5Ccal%20K%7D%7D) —— 预测的一系列 **Regional Proposal**    ( i 个grid cell 对 k+1 个parts进行预测)   
> >
> >  ![i](https://math.jianshu.com/math?formula=i) —— gird cell 的个数
> >
> >  ![{\cal K}](https://math.jianshu.com/math?formula=%7B%5Ccal%20K%7D)——  要检测的目标数，![K](https://math.jianshu.com/math?formula=K) *is the number of parts*, ![k = 0](https://math.jianshu.com/math?formula=k%20%3D%200) 代表一个完整的人
> >
> >  ![R, I](https://math.jianshu.com/math?formula=R%2C%20I) —— 二进制随机变量
> >
> >  ![p(R|k,i)](https://math.jianshu.com/math?formula=p(R%7Ck%2Ci)) —— grid cell ![i](https://math.jianshu.com/math?formula=i) 负责检测肢体部位![k](https://math.jianshu.com/math?formula=k)的概率， 如果ground truth bounding box of k 的中心落在第![i](https://math.jianshu.com/math?formula=i)个grid cell中，则第![i](https://math.jianshu.com/math?formula=i)个grid cell就负责![k](https://math.jianshu.com/math?formula=k)肢体的检测
> >
> >  ![p(I|R, k, i)](https://math.jianshu.com/math?formula=p(I%7CR%2C%20k%2C%20i)) —— 第![i](https://math.jianshu.com/math?formula=i)个cell预测的第![k](https://math.jianshu.com/math?formula=k)个bounding box与ground truth的 IoU
> >
> >  ![(o^i_{x, k}, o^i_{y, k})](https://math.jianshu.com/math?formula=(o%5Ei_%7Bx%2C%20k%7D%2C%20o%5Ei_%7By%2C%20k%7D)) —— bounding box的中心相对于grid cell的边界的距离，并根据对应网格归一化[0-1]之间
> >
> >  ![w^i_k , h^i_k](https://math.jianshu.com/math?formula=w%5Ei_k%20%2C%20h%5Ei_k) —— bounding box的宽、高，根据图像的尺寸归一化[0-1]之间

如果gird i 对k 负责的话， **p(I | R, k, i)这个条件概率是检测框的置信度。**通过 (IoU) between the
predicted bounding box and the ground truth bounding box.

> 实际的标签集，就是关节对应的点坐标。GT-bbox是 如果从标签集中取到的?? 
>
> gtbbox 是正方形 
>
> k=0  边长长度是头部长度的两倍 
>
> 其他部位的长度是头部大小的一半
>
>  Therefore, all ground truth boxes can be computed from two given head keypoints

> 如果GT bbox 横跨多个cell怎么办？



k=0:

对于每个人的位置可以通过一个包含整个身体或者头部的矩形框进行描述。





##### 对于位于x的grid cell i , CNN还输出keypoints关节连接概率的集合${C_{k_1, k_2}}  \ (k_1, k_2) \in L$ 

==$L$ 可以==表示成组成肢体关键点的索引。

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-04_16-16-25.png)

>
>
> $每一个C_{k_1,K_2} 有 H^‘ W^‘ 种 $
>
> > ![C_{k_{1}k_{2}} = {\lbrace p(C|k_1, k_2, x, x + \Delta x) \rbrace}_{\Delta x \in \chi }](https://math.jianshu.com/math?formula=C_%7Bk_%7B1%7Dk_%7B2%7D%7D%20%3D%20%7B%5Clbrace%20p(C%7Ck_1%2C%20k_2%2C%20x%2C%20x%20%2B%20%5CDelta%20x)%20%5Crbrace%7D_%7B%5CDelta%20x%20%5Cin%20%5Cchi%20%7D)  
> > ![\chi = \lbrace \Delta x = (\Delta x, \Delta y) | |\Delta x| \leq W^\prime \wedge |\Delta y| \leq H^\prime\rbrace](https://math.jianshu.com/math?formula=%5Cchi%20%3D%20%5Clbrace%20%5CDelta%20x%20%3D%20(%5CDelta%20x%2C%20%5CDelta%20y)%20%7C%20%7C%5CDelta%20x%7C%20%5Cleq%20W%5E%5Cprime%20%5Cwedge%20%7C%5CDelta%20y%7C%20%5Cleq%20H%5E%5Cprime%5Crbrace)
> >
> > > ![{\lbrace C_{k_{1}k_{2}} \rbrace}_{(k_1, k_2) \in \cal L}](https://math.jianshu.com/math?formula=%7B%5Clbrace%20C_%7Bk_%7B1%7Dk_%7B2%7D%7D%20%5Crbrace%7D_%7B(k_1%2C%20k_2)%20%5Cin%20%5Ccal%20L%7D) —— ![\cal L](https://math.jianshu.com/math?formula=%5Ccal%20L) 代表能被检测到的肢体，![C_{k_{1}k_{2}}](https://math.jianshu.com/math?formula=C_%7Bk_%7B1%7Dk_%7B2%7D%7D)表示关节![k_1k_2](https://math.jianshu.com/math?formula=k_1k_2)的连接是肢体的概率
> > > ![C](https://math.jianshu.com/math?formula=C) —— 二进制随机变量
> > > ![x](https://math.jianshu.com/math?formula=x) —— 第![i](https://math.jianshu.com/math?formula=i)个grid cell的位置
> > > ![H^\prime \ \ W^\prime](https://math.jianshu.com/math?formula=H%5E%5Cprime%20%5C%20%5C%20W%5E%5Cprime) —— 文中假设位于![x](https://math.jianshu.com/math?formula=x)的肢体仅能到达以![x](https://math.jianshu.com/math?formula=x)为中心的![H^\prime \times W^\prime](https://math.jianshu.com/math?formula=H%5E%5Cprime%20%5Ctimes%20W%5E%5Cprime) 区域
>
> ​	

其中p(C|k1 , k2 , x, x + ∆x)表示从x处的k1到x + ∆x处的k2之间**有关节连接的概率**，如图所示。在这里我们假设位于x处的关节只能与一个**局部区域H’ x W’** 内的关节相连。

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-04_16-21-21.png)



#### 最后CNN输出的Tensor 维度为:

​	==$$ HW\big\{ 6(K+1) + H'W'|L| \big\} $$==



> For limb detections, the two head keypoints
> are defined as being connected to person instances and the other connections are defined
> similar to those in [1]. Therefore
>
> |L| is set to 15.







#### 训练时的损失为：

​	![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-04_16-25-04.png)



>  $$H'W'|L| \big\} $$ 计算损失值原理 ？
>
> $\delta_k^i$ 到底怎么算 ？？ 





## 二、Pose proposal generation

有了上述的候选区域，接下来就是**如何连接关节生成姿态**了。通过NMS生成**一组离散的关键点候选位置**. 这些关键点候选位置由于多人或者误检的情况，每个关键点存在多个候选位置。从这些关键点中生成最优的姿态，这是一个K部图匹配的NP-hard问题。

针对这个场景，可以进行2点简化。

首先选择**最少的边**可以得到==关节姿态的生成树==，它的节点和边代表节点之间有直接的连接，而不是使用整体图。

其次**匹配的问题**可以分解为一组二部图匹配的问题，如图4所示。另外作者考虑到使用的CNN模型较小，其感知野相对较小，提出了考虑到**非邻接节点关系**的匹配方法.

​	![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-04_16-31-00.png)



























