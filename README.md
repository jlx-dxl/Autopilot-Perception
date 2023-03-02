# 自动驾驶环境感知

## 一、2D视觉感知

### 1. 基本任务

![image.png](https://s2.loli.net/2023/03/02/4DzFGbtHrqLJhy3.png)

单张图像——稀疏输出：物体检测

图像序列——稀疏输出：物体跟踪

单张图像——稠密输出：语义分割

图像序列——稠密输出：实时全景分割

### 2. 数据集和基准测试

1. 通用数据集

   PASCAL-VOC, MSCOCO, MOT

2. 自动驾驶数据集

   KITTI, NuScenes, Waymo

   ![image.png](https://s2.loli.net/2023/03/02/YdWEhsy3gjHPDM8.png)

3. 性能指标

   | 指标                | 含义（p为阈值）                             |
   | ------------------- | ------------------------------------------- |
   | True Positive (TP)  | 预测框与标注框IoU>p                         |
   | False Positice (FP) | 预测框与标注框IoU<p或在无标注框处产生预测框 |
   | False Negative (FN) | 有标注框处没有产生预测框                    |
   | True Negative (TN)  | 没有标注框处没有产生预测框                  |

   - Precision：所有预测框（所有P）里有多少是正确预测的
   - Recall：所有标注框（TP+FN）里有多少被正确的找到了

   $$
   Precision=\frac{TP}{TP+FP}\\
   Recall=\frac{TP}{TP+FN}
   $$

   - 取多个阈值p就可以画出Precision-Recall曲线

     ![image.png](https://s2.loli.net/2023/03/02/5SiPf1dY7jI4kpW.png)

   - AP (Average Precision)：不同Recall下的Precision的平均值（单一标签）

   - mAP (mean Average Precision)：多个标签AP的平均值

### 3. 物体检测算法

#### （1）传统的物体检测算法

![image.png](https://s2.loli.net/2023/03/02/M7PypnOJVCS9KRj.png)

#### （2）R-CNN

##### a. 流程

![image.png](https://s2.loli.net/2023/03/02/gPUlcvRpzCA5ytY.png)

1. 输入图像
2. 产生预选窗口，根据显著性，在显著性明显的区域产生不同大小的预选窗口（每张图片约生成2k个）
3. 将这些窗口放缩到固定大小
4. 送入卷积神经网络进行特征提取（在ImageNet上预训练好的）
5. 窗口特征采用SVM进行分类

##### b. 缺点

1. 窗口数量太多
2. 特征提取冗余
3. 算法运行效率低（10s/img）

#### （3）Fast R-CNN

##### a. 流程

![image.png](https://s2.loli.net/2023/03/02/hRiBZ2vKDXY9Qaw.png)

1. 输入图像
2. CNN进行全图的特征提取
3. 通过selective search进行感兴趣区域（ROI）的提取
4. 对ROI进行ROI Pooling，将ROI压缩成定长的特征向量FCs
5. 用全连接网络进行分类和边框回归

##### b. 缺点

通过selective search来得到候选区域，这个过程依然较慢（2s/img）

#### （4）Faster R-CNN

##### a. 变化

1. 将Fast R-CNN中通过selective search进行ROI的提取改为通过区域候选网络（Region Proposal network，RPN）来在特征图的基础上生成候选框

![image.png](https://s2.loli.net/2023/03/02/FTHdZnjEGrIyhXt.png)

2. 在RPN中引入了Anchor的概念，即根据要检测目标的大小和长宽比的先验知识预定义一系列锚框（Anchor），用这些锚框滚动扫描全图来找ROI，最后再进行回归得到bbb

##### b. 问题

第一个端到端的物体检测网络，速度接近实时（17FPS）

1. ROI Pooling过程比较耗时
2. Anchor需要先验知识，需要人工设计

![image.png](https://s2.loli.net/2023/03/02/A7C4FGB8NoDRtcU.png)

#### （5）采用Feature Pyramid Network（FPN）对特征提取步骤进行优化

![image.png](https://s2.loli.net/2023/03/02/iT2EIKvu7LRqz9N.png)

![image.png](https://s2.loli.net/2023/03/02/oFpDlzf58HOQcAC.png)

#### （6）SSD（Single Shot MultiBox Detector）

##### a. 流程

![image.png](https://s2.loli.net/2023/03/02/hEpD9GgdXvyfwkn.png)

1. backbone是一个VGG-16的CNN进行特征提取
2. 对特征图进行不同级别的下采样，得到不同分辨率的特征图，与FPN不同在于这里只有下采样，没有上采样
3. 对不同分辨率的特征图的每一个像素进行分类和回归，没有候选框的概念

##### b. 缺点

稠密的采样导致正负样本的不平衡，大量负样本会支配损失函数（候选框会过滤掉大量的负样本）

#### （7）YOLO (You Only Look Once)

##### a. 流程

![image.png](https://s2.loli.net/2023/03/02/npXwNqhPzV7B6if.png)

##### b. 缺点

Anchor需要手工设计，Anchor的数量较大时会影响速度

![image.png](https://s2.loli.net/2023/03/02/viOHTfYNKbCs4ID.png)

#### （8）CenterNet（Anchor-free）

##### a. 基本组成

- Backbone与R-CNN/YOLO类似
- 不同之处主要在于Head的设计
  - Center Heatmap：在每个位置上是否出现了物体，属于什么类别
  - Center Offset：中心点偏移
  - Bbox Size：边框大小

##### b. Backbone设计