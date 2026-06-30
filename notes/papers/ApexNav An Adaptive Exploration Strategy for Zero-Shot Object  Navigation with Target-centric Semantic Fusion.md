# ApexNav An Adaptive Exploration Strategy for Zero-Shot Object  Navigation with Target-centric Semantic Fusion

## Metadata

- Paper: ApexNav: An Adaptive Exploration Strategy for Zero-Shot Object  Navigation with Target-centric Semantic Fusion
- Authors: Mingjie Zhang et. al.
- Year: 2025 Sep
- Link:https://arxiv.org/abs/2504.14478
- Area: VLN
- Tags:Vision-Based Navigation, Autonomous Agents, Search and Rescue Robot

## One-Sentence Summary


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





## Key Contributions

1.
2.
3.

## Experiments


## Limitations


## Connections


## Questions


