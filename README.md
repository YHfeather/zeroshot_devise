# DeViSE
ZeroShot learning is about the combination between NLP and CV which can predict the label of unseen data when it was trained.

## 简介
- 使用CUB鸟类图片数据集和text8英文维基百科数据集实现了DeViSE模型
- DeViSE模型可以对在训练图片分类器时从未出现的label进行预测
- 核心思想是结合图片分类器输出的feature representation和Word2Vect输出的图片label的word embedding representation作为DeViCE核心模型的输入，训练一个线性转换模型。从而可以实现对在训练集合中从未出现的label进行分类

## 流程
1. 预训练一个图片分类器
2. 预训练一个词向量模型
3. 训练一个线性模型，输入为1:对于一个image图片分类器的输出特征表示，2:image对应label的词向量特征表示。然后定义一个trainable的矩阵并定义hinge rank loss进行模型训练。
4. 使用一个从未出现在图片分类器训练过程中label的image测试DeViSE模型，并使用最大余弦相似度选择DeViSE模型预测的label

## 运行方法
直接打开'DEViSE.py'进行运行即可

## 组成
该模型一共由三个部分组成，分别对应三个文件。
- AlexNet构建了一个图片分类器(visual_model.py)
- Skip_Gram构建了一个Word2Vect模型(skip_gram.py)
- 分别使用AlexNet训练生成的image feature representation和skip_gram训练生成的label embedding representation，训练一个DeViSE模型，用以对在训练AlexNet时未出现的label进行分类(DeViSE.py)

### 图片分类器
使用在ImageNet比赛中获得优胜的AlexNet网络结构构建一个CUB图片分类器，
#### CUB数据集
CUB数据集中，共有200种鸟类图片。在训练AlexNet过程中，不同于一般训练过程，只是用1~150个标签的鸟类进行模型训练，而剩余的151~200个标签的鸟类图片作为DeViSE模型的测试数据存在。
#### NN结构
该图片分类器借鉴AlexNet的结构共包含：
1. 三个卷积层
2. 三个最大池化层
3. 三个local response normalization层
4. 一个包含4个隐藏层的全连接层
5. 输出层使用softmax函数计算loss
6. 全连接层使用dropout进行正则化
7. 定义一个tensor用来返回在softmax之前的image representation作为DeViSE调用的输出

### 词向量模型
使用Skip_Gram构建词向量模型，数据集为text8简易英文维基百科数据集
#### 结构
和常规的Skip_Gram模型相同，略

### DeViSE Core

#### 结构
1. 定义一个线性运损模型：一个trainable的矩阵M，size(M) = len(image_representation) * len(label embedding representation).
2. 训练过程：用一个图片的图片分类器的输出representation和该图片对应label的word embedding representation和该矩阵进行线性运算。
3. 定义hinge rank loss（见paper），梯度下降最小化该loss
4. 测试过程: 使用从未在图片分类器中出现过的label的图片进行测试。用图片分类器输出该test image 的 representation，然后用该representation和训练好的矩阵M进行线性运算得到一个vector，该vector即DeViSE的预测输出label对应的word embedding representation，然后从整个word embedding space 找到和该预测向量余弦相似度最大的word 即为DeViSE对应的预测label


