﻿基于Tensorflow的图像风格迁移
#
>>主要参考文献为李飞飞的《Perceptual Losses for Real-Time Style Transfer and Super-Resolution》<br> 
#
图像风格转换<br>
>>以目前的深度学习技术，如果给定两张图像，完全有能力让计算机识别出图像具体内容。而图像的风格是一种很抽象的东西，人眼能够很有效地的辨别出不同画家不同流派绘画的风格，而在计算机的眼中，本质上就是一些像素，多层网络的实质其实就是找出更复杂、更内在的特性(features)，所以图像的风格理论上可以通过多层网络来提取图像里面可能含有的一些有意思的特征。<br> 
>>快速风格迁移的网络结构包含两个部分。一个是“生成网络”（Image Transform Net），一个是“损失网络”（Loss Network）。生成网络输入层接收一个输入图片，最终输出层输出也是一张图片（即风格转换后的结果）。模型总体分为两个阶段，训练阶段和执行阶段。模型如图所示。 其中左侧是生成网络，右侧为损失网络。<br> 
>>训练阶段：选定一张风格图片。训练过程中，将数据集中的图片输入网络，生成网络生成结果图片y，损失网络提取图像的特征图，将生成图片y分别与目标风格图片ys和目标输入图片（内容图片）yc做损失计算，根据损失值来调整生成网络的权值，通过最小化损失值来达到目标效果。<br> 
>>执行阶段：给定一张图片，将其输入已经训练好的生成网络，输出这张图片风格转换后的结果。
#
生成网络<br>
>>对于生成网络，本质上是一个卷积神经网络，这里的生成网络是一个深度残差网络，不用任何的池化层，取而代之的是用步幅卷积或微步幅卷积做网络内的上采样或者下采样。这里的神经网络有五个残差块组成。除了最末的输出层以外，所有的非残差卷积层都跟着一个空间性的instance-normalization，和RELU的非线性层，instance-normalization正则化是用来防止过拟合的。最末层使用一个缩放的Tanh来确保输出图像的像素在[0,255]之间。除开第一个和最后一个层用9x9的卷积核(kernel)，其他所有卷积层都用3x3的卷积核。
#
损失网络<br>
>>损失网络φ是能定义一个内容损失(content loss)和一个风格损失(style loss)，分别衡量内容和风格上的差距。对于每一张输入的图片x我们有一个内容目标yc一个风格目标ys，对于风格转换，内容目标yc是输入图像x，输出图像y，应该把风格ys结合到内容x=yc上。系统为每一个目标风格训练一个网络。<br> 
>>为了明确逐像素损失函数的缺点，并确保所用到的损失函数能更好的衡量图片感知及语义上的差距，需要使用一个预先训练好用于图像分类的CNN，这个CNN已经学会感知和语义信息编码，这正是图像风格转换系统的损失函数中需要做的。所以使用了一个预训练好用于图像分类的网络φ，来定义系统的损失函数。之后使用同样是深度卷积网络的损失函数来训练我们的深度卷积转换网络。 <br>
>>这里的损失网络虽然也是卷积神经网络(CNN)，但是参数不做更新，只用来做内容损失和风格损失的计算，训练更新的是前面的生成网络的权值参数。所以从整个网络结构上来看输入图像通过生成网络得到转换的图像，然后计算对应的损失，整个网络通过最小化这个损失去不断更新前面的生成网络权值。<br>
 