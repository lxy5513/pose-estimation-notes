# 3D human pose estimation in video with temporal convolutions and semi-supervised training



## 概括

​	在这篇paper中， 我们展示了视频中的3D姿态可以通过基于 **在2D关键点上扩张的时域卷积** 的全卷积模型来进行有效的预测 我们引进一种反响投影的简单、有效的半监督训练方法。这种方法使用的数据是未标注的视频数据。

​	首先我们使用已经预测好的2D关键点视频（未标注）作为输入，然后预测3D姿态， 最后反向投影回输入的2D关键点。