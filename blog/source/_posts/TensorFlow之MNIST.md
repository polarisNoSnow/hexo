# MNIST
## 一.MNIST数据集

1.60000行训练数据集(mnist.train) 
2.10000行测试数据集(mnist.test) 

每一个MNIST数据单元（训练+测试）包含两部分：一张手写的图片（xs）、一个对应的标签（yx），其中训练数据集的图片是 mnist.train.images ,训练数据集的标签是 mnist.train.labels。

## 二.原理简介

1. 每一张图片包含28像素X28像素，展开成一个向量28*28=784


为了得到一张给定图片属于某个特定数字类的证据（evidence），**我们对图片像素值进行加权求和**。如果这个像素具有很强的证据说明这张图片不属于该类，那么相应的权值为负数，相反如果这个像素拥有有利的证据支持这张图片属于这个类，那么权值是正数。其中bi作为偏置量（bias）去中和 因为输入带有的一些无关干扰量(**也可以使输出的曲线平移https://www.jianshu.com/p/3c9c67b6b5dc**)。

$$
evidence_i = ∑W_{i,j} * j + b_i
$$

其中Wi代表权重，bi代表数字 i 类的偏置量，j 代表给定图片 x 的像素索引用于像素求和。
如：输入的一幅图片是9的特征
$$
exevidence_9 = (W_{9,第1个像素} * 第1个像素的索引 + W_{9,第2个像素} * 第2个像素的索引 + ... + W_{9,第784个像素} * 第784个像素的索引)  + b_i
$$



2. 然后用softmax函数可以把这些证据转换成概率 y：
$$
y=softmax(evidence)
$$

softmax简介https://www.w3cschool.cn/tensorflow_python/tensorflow_python-c1ov28so.html

w3cschool的softmax矩阵有误 和 模型图不符合

 **激励函数：将线性函数非线性化的函数，如relu **

**normalize ：归一化处理，如：[-10,10] 调整为[-1,1]**

