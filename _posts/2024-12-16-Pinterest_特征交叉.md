* ```
  layout: post
  title: "Pinterest_特征交叉"
  date: 2024-11-18
  categories: blogging
  ```

## 2024 arxiv

[Improving feature interactions at Pinterest under industry constraints](https://arxiv.org/pdf/2412.01985)

### 问题背景

1. 特征交叉是精排的重要一环，精排模型分成三块：特征预处理（如embedding）、特征交叉、任务预估。
2. 性能和存储的工程性限制：指出引文[11][14][8][17]的工作都是在离线数据集上做的，复杂度等方面在现实的工业系统的吞吐量上来说不适用。
   Lighter and Interpretable Cross Network for CTR Prediction.
   MaskNet: Introducing Feature-Wise Multiplication to CTR Ranking Models by Instance-Guided Mask.
   FinalMLP: An Enhanced Two-Stream MLP Model for CTR Prediction.
   MemoNet: Memorizing All Cross Features’ Representations Efficiently via Multi-Hash Codebook Network for CTR Prediction.

### 本文贡献

1. 列举了工业系统里的约束
2. 通过一系列实验来对挑选合适的特征交叉架构进行了指导
3. 经过工业系统约束下的学习实验，摸索出了一些精排模型提升特征交叉的经验，并进行分享

### Related Works

1. W&D、DeepFM（DeepFI自此而始）
2. MLP是一种隐式的高阶交叉；DCN、DCNv2、xDeepFM是显式的特征交叉；AutoInt使用Attention进行交叉
3. Maksnet、FinalMLP、GDCN、DHEN、SDCNv3后浪花式百出，但已经在耗时、内存上大大增加，不再适合工业部署
4. DeepLight试图通过剪枝兼顾上述后浪的效果提升和模型性能耗费，但是**剪枝可能会导致复现性差（我猜是说一致性和随机性？）**；transformer架构适合做特征交互（fusionfea在做的事情？）[3:Hiformer: Heterogeneous Feature Interactions Learning with Transformers for Recommender Systems.]
   【根据4.4中说明，可复现性主要是指  模型的可复现性非常重要，尤其是在多次训练中，需要确保相同的数据和配置能够生成一致的结果。如果模型的结果不一致，难以判断观察到的性能变化是否真正来源于模型优化，还是仅仅是随机变化。】

* 备注：Ranking打榜可参考[2: Click-Through Rate Prediction on Criteo. (2024). https://paperswithcode.com/sota/click-through-rate-prediction-on-criteo] shows that ClickThrough Rate Prediction on the Criteo dataset has been getting better over the years with better feature interaction architectures.

### 模型基本框架

1. 输入特征：dense_feature(normalized)、sparse_feature（emb size依据特征空间选择)、embedding_feature（如果维度过高，会做降维映射)；用户行为序列使用[15]的方式建模（什么叫user's past engagements as input?)[15: TransAct: Transformer-based Realtime User Action Model for Recommendation at Pinterest.]然后pooling成一个embedding
2. 特征拼接：sparse和embedding feature都做L2 norm，然后和dense进行concat
3. 特征交叉：拼接后的特征，经过4层full-rank DCNv2
4. shortcut：交叉后的特征，拼接未交叉的特征，一起到MLP
5. 多目标：We use a shared MLP with multiple hidden layers and predict 𝐾 outputs corresponding to the 𝐾 tasks.【这里做的好像有点落后】

`<img src="image/2024-12-16-Pinterest_特征交叉/1734424505155.png" width="50%"/>`

### 工业系统的约束
