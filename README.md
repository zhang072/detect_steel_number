## detect_steel_number
DCIC 钢筋数量识别 baseline 0.98+
## 比赛地址
[智能盘点—钢筋数量AI识别](https://www.datafountain.cn/competitions/332/details)
## 环境依赖
```ubuntu、tensorflow、keras、skimage、opencv-python、numpy、pandas、matplotlib等```
## 我的方案
#### 关于检测/分割模型选择
尝试了retinanet、faster rcnn、fpn和msak rcnn，其中mask rcnn目前得分0.998，从kaggle上得知使用U-Net全卷积网络进行语义分割可能效果比较好，目前还没有尝试。
#### 关于优化器选择
+ 前期选择默认SGD优化器，后来在60epoch后选择用Adam优化器。
+ I found that the model reaches a local minima faster when trained using Adam optimizer compared to default SGD optimizer。
#### 关于学习率策略
每隔25epoch，学习率下降10倍比较好。
#### 关于训练策略
Train in 3 stages: on 512x512 crops containing ships, then finetune on 1024x1024, and finally on 2048x2048. Inference on full-sized 2000x2666 images(由于时间关系没有尝试)
#### 关于图像尺寸
图像尺寸越大越好，但是注意至少要为2^6倍数，受限于硬件条件我这里是2040*2048。
#### 关于数据增强
我不确定数据增强是否有很大效果，下面是我的数据增强方式：
```python
augmentation = iaa.Sometimes(0.6,
                             iaa.Noop(),
                             iaa.OneOf(
                                 [
                                     iaa.Fliplr(0.5),
                                     iaa.Flipud(0.5),
                                     iaa.GaussianBlur(sigma=(0.0, 3.0)),
                                     iaa.AdditiveGaussianNoise(scale=(0, 0.02 * 255)),
                                     iaa.CoarseDropout(0.02, size_percent=0.5),
                                     # iaa.Add((-40, 40), per_channel=0.5),
                                     # iaa.WithChannels(0, iaa.Affine(rotate=(0, 45))),
                                     iaa.Multiply((0.8, 1.5)),
                                     # iaa.Superpixels(p_replace=0.1, n_segments=(16, 32))
                                 ]
                             ))
```
