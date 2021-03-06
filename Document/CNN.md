1. 什么是CNN

   卷积神经网络（Convolutional Neural Networks, CNN）是一类包含卷积计算且具有深度结构的前馈神经网络（Feedforward Neural Networks），是深度学习（deep learning）的代表算法之一。CNN最常用于CV领域，但是在NLP等其他领域也有应用，如用于文本分类的TextCNN。

   下面是一个CNN的经典网络结构（LeNet）：

   [![sHYs0A.png](https://s3.ax1x.com/2021/01/24/sHYs0A.png)](https://imgchr.com/i/sHYs0A)

   CNN一般具有以下结构：

   - 输入层

     输入CNN的图片数据通常会进行以下处理之一：

     - 对每个像素点求均值，得到均值图像，当训练时用原图减去均值图像。
     - 对所有输入在三个颜色通道R/G/B上取均值，只会得到3个值，当训练时减去对应的颜色通道均值。
     - 减去均值后除以标准差进行归一化。
     - 白化（用PCA降维，对数据每个特征维度除以标准差进行归一化）。

   - 卷积层

     对图像（不同的数据窗口数据）和卷积核（一组固定的权重：因为每个神经元的多个权重固定，所以又可以看做一个恒定的滤波器 filter）做内积（逐个元素相乘再求和）的操作就是所谓的『卷积』操作，也是卷积神经网络的名字来源。

     卷积所用到的卷积核实际上就是数字图像处理中所用到的滤波器，用于提取一组固定特征（颜色深浅、轮廓）。所不同的是，传统的滤波器的算子是固定好的，而卷积核的权重是通过训练得到的。

     卷积核在图像上不断平滑移动，直到计算完所有卷积核大小的窗口，将输出按原顺序排列即为一个输出通道。如果输入为多通道数据，则每个输入通道需要一个卷积核，输出通道上一个像素的值为对应位置多个卷积核的结果之和。

     卷积过程中有如下参数：

     - 卷积核大小 kernel_size 。
     - 步长 stride ，即卷积核每次移动的像素数。
     - 输入尺寸 input_size（只针对某一维）。
     - 输出尺寸 output_size（只针对某一维）。
     - 填充数 padding，即在边缘填充像素的数量，可以填0，也可以填边缘像素取值。
     - 输入通道数 input_channel，如RGB图像的输入通道数为3。
     - 输出通道数 output_channel，每个输出通道对应一组卷积核。因为一组卷积核可以理解为抽取一个特征，所以输出通道数可以理解为抽取的特征数。

     CNN每一层输出的特征图（feature map）上的像素点在原始图像上映射的区域大小称为感受野（Receptive Field），其计算公式为：
     $$
     r_n=r_{n-1}+(kernel\_size_n-1)*\prod_{i=1}^{n-1}stride_i
     $$
     感受野计算公式推导可参考：[https://zhuanlan.zhihu.com/p/28492837](https://zhuanlan.zhihu.com/p/28492837)。需要指出的是，感受野针对的是特定的维度，比如对于输入二维图像的高和宽，各有一个感受野。同时后面所要介绍的池化层的感受野计算与卷积层相同。

     卷积的结果需要通过非线性的激活函数如ReLU、Tanh、Sigmoid进行变换。

     卷积层输出尺寸与输入尺寸的对应关系为：
     $$
     output\_size=\frac{input\_size-kernel\_size+2*padding}{stride}+1
     $$
     设卷积层的输入尺寸为$W*H*C$，输出尺寸为$W'*H'*C'$，卷积核大小为$K*K$，则计算量为：
     $$
     W'*H'*C'*K*K*C
     $$
     参数量为：
     $$
     K*K*C*C'
     $$
     达到相同感受野时，使用小卷积核往往比使用大卷积核需要更少的参数，因此一般使用小卷积核。

   - 池化层

      池化层（pooling）也叫做下采样层，它能够对输入的特征图进行压缩，一方面使特征图变小，简化网络计算复杂度；一方面进行特征压缩，提取主要特征。

     常用的池化操作有最大池化（max pooling）和平均池化（average pooling）。

     - 最大池化（max pooling）

       取出每个窗口内的最大值作为输出，可以理解为进行了特征选择，选出了分类辨识度更好的特征，提供了非线性。 

       最大池化反向求导时把梯度直接传给前一层窗口内值最大的那一个像素，而窗口内其他像素不接受梯度，也就是为0。

     - 平均池化（average pooling）

       取出每个窗口内的平均值作为输出，更强调对整体特征信息进行一层下采样，在减少参数维度的贡献上更大一点，更多的体现在信息的完整传递这个维度上。

       平均池化反向求导时窗口内每个像素的梯度为对应输出像素梯度的平均值。

     最大池化的效果一般比平均池化更好。池化的主要作用一方面是去掉冗余信息，一方面要保留feature map的特征信息。例如在分类问题中，我们需要知道的是这张图像有什么object，而不大关心这个object位置在哪，在这种情况下显然max pooling比average pooling更合适。在网络比较深的地方，特征已经稀疏了，从一块区域里选出最大的，比起这片区域的平均值来，更能把稀疏的特征传递下去。

   - 全连接层

     接在网络最后用于将特征映射到输出空间。

2. CNN的优点
   - 局部连接，权值共享，大大减少了参数量。
   - 卷积可以并行运算，加快计算速度。
   - 层级卷积池化操作，由低级特征逐步组合为高级特征。
   - 平移不变性，由于卷积核是在输入图像上不断移动进行卷积，因此即使输入图像中某个物体位置进行了平移仍然能够被检测到。同时由于有池化层的存在，比如最大池化，如果最大值被移动了，但是仍然在这个感受野中，那么池化层也仍然会输出相同的最大值。

