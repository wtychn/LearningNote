# 深度学习相关知识学习总结
因为毕业课程等要求，还是需要接触一下深度学习相关知识，这里对所学皮毛进行一点总结。
## 学习分类
可以根据学习方式分为几类：
- 监督学习：需要含**有标签**的数据集进行训练
- 无监督学习：需要**不含标签**的数据集进行训练
- 强化学习：自动训练，自对抗等方式
## 神经网络
深度学习之前必不可少的就是学习神经网络的基础知识，先看一张人工智能相关概念的包含关系图：
![包含关系](https://gitee.com/wtychn/ImageBed/raw/master/img/20200621132955.png)
### 参数解析
- **输入层神经元个数**：对应特征维度
- **输出层神经元个数**：对应分类类别数
- **隐层节点**：隐层节点数太少，则网络从样本中获取信息的能力就越差，无法反映数据集的规律；隐层节点数太多，则网络的拟合能力过强，可能拟合数据集中的噪声部分，导致模型泛化能力变差。
- **激活函数**：
  1. `sigmoid`函数![sigmoid](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622093925.png)
  2. `tanh`函数（解决`sigmoid`神经元产生非零均值的问题）![tanh](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622094254.png)
  3. `ReLU`函数![ReLU](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622094354.png)
  4. `PReLU`/`Leaky ReLU`函数（解决`ReLU`在x<0时不被激活的问题）![PReLU/Leaky ReLU](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622094731.png)
  5. `ELU`（融合`sigmoid`和`ReLU`）![ELU](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622094851.png)
- **欠拟合**：训练的特征少，误差较大
- **过拟合**：训练特征维度多，使得拟合函数完美接近训练集，但是泛化能力差，实际预测能力不足。![欠拟合和过拟合](https://gitee.com/wtychn/ImageBed/raw/master/img/20200622100002.png)
- **优化器**：大部分时间可以选择`Adam`，可以又快又好地完成训练