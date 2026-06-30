# ApexNav An Adaptive Exploration Strategy for Zero-Shot Object  Navigation with Target-centric Semantic Fusion

## Metadata

- Paper: ApexNav: An Adaptive Exploration Strategy for Zero-Shot Object  Navigation with Target-centric Semantic Fusion
- Authors: Mingjie Zhang et. al.
- Year: 2025 Sep
- Link:https://arxiv.org/abs/2504.14478
- Area: VLN
- Tags:Vision-Based Navigation, Autonomous Agents, Search and Rescue Robot

## One-Sentence Summary
个人认为：本文重点是解决ZSON过程中，探索阶段的可切换方案，还有目标识别阶段的一种新语义融合方法，使得探索策略更加适应环境，目标识别更具鲁棒性。


## Problem
1.现有的Zero-shot Object Navigation(下称ZSON)，往往重度依赖语义推理，但是语义信息在相当一部分情况下，难以获取或无法充分表征
2.在视觉导航过程中，单帧目标检测不可靠，多帧融合时使用max-confidence fusion也会带来标签污染
3.end to end 模型训练消耗大，modular learning-based method 效果不佳

总体来看就是为了解决ZSON的两个痛点：探索阶段要根据语义线索强弱自适应切换策略；识别阶段要用多帧目标中心融合，而不是相信单帧或最高置信度。
## Method
### Adaptive Exploration Strategy
Apex-Nav 会根据语义线索的强弱自动切换两种探索模式，如果语义线索强则使用semantic-based exploration, 如果语义线索弱则使用 geometry-based exploration

#### 语义线索强弱的衡量标准：
1.**max-to-mean ratio**：最高语义分数和平均语义分数的比值；这个比值很高，说明某些 frontier 明显比其他区域更可能包含目标。
2.**standard deviation**，即 frontier 语义分数的标准差；如果标准差较大，说明不同 frontier 的语义相关性差异明显，语义信息具有区分度
当两个指标均超过阈值，则认为语义线索强，切换到语义探索模式，否则使用几何探索，扩大信息获取范围

#### 语义探索阶段的策略：
先筛选出一批高语义分数 frontier，把访问这些 frontier 的顺序建模成一个 Asymmetric Traveling Salesman Problem（非对称旅行商问题），再用 LKH-Solver 求解访问顺序。
这样做的目的是在保证语义相关性的同时，减少重复路径和无效折返，相当于用一组候选区域的全局访问顺序替换局部的贪心策略，解决不可靠的问题。

#### 几何探索阶段的策略：
当语义分数比较均匀、无法判断目标方向时，ApexNav 使用最近 frontier 策略。虽然 TSP 常用于全覆盖探索，但 ObjectNav 的目标不是完整覆盖环境，而是尽快找到目标；在语义线索弱时，最近 frontier 反而更符合目标导航任务

### Target-centric Semantic Fusion
#### LLM 推理相似物列表
不仅仅是检测目标物体本身，而是利用LLM推理出容易混淆的对象列表，并且生成自适应的置信阈值，要求融合后的置信度超过阈值才被认为是可靠目标

#### 检测和分割
对于 COCO 数据集中的常见类别，使用 **YOLOv7** 检测；对于其他开放词汇类别，使用 **Grounding-DINO**；检测框再输入 **Mobile-SAM** 得到实例分割结果；最后结合深度图提取每个实例对应的 3D 点云，并使用 DBSCAN 去除噪声。即每一帧检测的若干对象每一个应该包含点云信息，检测置信度，语义标签；

使用object cluster作为基本单位，对应空间中的一个物体实例，并且包含多个候选标签和对应置信度，保留了多类别的竞争关系，而不是直接锁死类别。

检测体积加权，检测体积近似为点云中的点数。因为一个物体在当前视角中被观察到的区域越大，点云点数越多，说明该检测更可靠；如果只是被遮挡后露出一小部分，检测结果就不应有太大权重。更新时候：
````text
新的融合置信度 = 历史置信度 × 历史观测权重 + 当前检测置信度 × 当前观测权重
````

置信度惩罚：如果某个 object cluster 以前被检测为目标，但机器人后来又从新的视角看到了这个位置，却没有再次检测到目标，那么系统会认为：之前的目标判断可能不可靠，施加惩罚项

## Key Contributions

1.
2.
3.

## Experiments


## Limitations


## Connections


## Questions


