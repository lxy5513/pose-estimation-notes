# Stacked Hourglass Network

### Keywords

##### ```across all scales```            ```intermediate supervision```



## Hourglass Design 

### Motivations: need to capture information at every scale  

该网络结构能够使同一个神经元感知更多的上下文信息。

local evidence is essential for identifying features like faces and hands, a final pose estimate requires a cohenrent understanding of the full body 

#### set up HG moudles 

- Convolutional and max pooling layers are used to process features down to a very low resolution 
- After reaching the lowest resolution, the network begins the sequence of upsampling and combination of features across scales(跨尺度)
- No Conv layers have filter greater than 3 × 3 

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-27_16-52-51.png)



### Stacked Hourglass with Intermediate Supervision(中间监督)

这种沙漏结构还具备可堆叠性，通过多个沙漏结构的堆叠，来组成新的更具表示能力的沙漏型结构。

- Stacking multiple hourglasses  

- Feeding the output of one as input into the next  

  > expand on a single hourglass by consecutively placing mutiple hourglass moudlues together end-to-end, the allows for repeated bottom-up, top-down inference across scales

- Loss is applied to the predictions of all hourglasses using the same ground truth. 

结合使用中间监督，反复的双向推断是这个网络最后能力的关键 





### Hourglass design

The person's orientation, the arrangement of their limbs, and the relationships of adjancent joints are useful infomation.

The hourglass is a simple, minimal design that has the capacity to **capture all of those features** and bring them together to output pixel-wise predictions



we choose to use a single pipeline with skip layers to perserve spatial information at each resolution. The network reache**r its lowest resolution at 4x4 pixels** allowing smaller spatial filter to be applied taht compare feature across the entire space of the image.



经过解析网络的输出之后，通过连续两轮的1x1的卷积网络，产生网路最后的预测结果。The output of the network is a set of heatmaps where for a given heatmap predict the probability of a joint's presence at each and every pixel 



![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-28_10-51-26.png)

### Layer Implement

Recent work has shown the value of reduction steps with 1x1 convolutions, as well as the benefits of using consective smaller filter to capture a larger spatial context.

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-28_10-42-19.png)

we experienced an increase in network performance after swithing **from** standard convolutional layers with large filter and no reduce **to** newer methods like the residual learning modules



### Stacked Hourglass with Intermediate Supervision

​	we take our network architecture further by stacking multiple hourglasses end-to-end, feeding the output of one as input into the next. the provides the network with a mechanism for repeated bottom-up, top-dowm inference allowing for **reevaluation of initial estimates and features** across the whole image. 

The key to the approch is the prediction of intermediate heatmaps upon which we can **apply a loss.** 

prediction are generated after passing through **each hourglass** where the network 

​	if you want the network to best refine predictions, there predictions cannot be exclusively at a local scale. The relationship to other joints and the general context and the understanding of full image is crucial.

​	local and global cues are intergrated whthin each hourglass module.