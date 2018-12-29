# compute vision base



## 1. SENet (Squeeze-and-Excitation Network)

>  Sequeeze-and-Excitation(SE) block并不是一个完整的网络结构，而是一个子结构，可以嵌到其他分类或检测模型中, 比如resnet Inception    

**SENet的核心思想在于通过网络根据loss去学习特征权重，使得有效的feature map权重大，无效或效果小的feature map权重小的方式训练模型达到更好的结果** 

![](https://raw.githubusercontent.com/lxy5513/Markdown_image_dateset/master/Xnip2018-12-27_19-36-26.png)



