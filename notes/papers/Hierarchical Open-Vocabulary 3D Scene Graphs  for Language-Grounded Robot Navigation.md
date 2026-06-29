# Hierarchical Open-Vocabulary 3D Scene Graphs  for Language-Grounded Robot Navigation

## Metadata

- Paper: Hierarchical Open-Vocabulary 3D Scene Graphs  for Language-Grounded Robot Navigatio
- Authors: Abdelrhman Werby et al.
- Year: 2024 June
- Link: https://arxiv.org/abs/2403.17846
- Area: 3D Scene Graph
- Tags: HOV-SG

## One-Sentence Summary
构建分层的开放词汇3D场景图，解决抽象对于Robot Navigation带来的阻碍，实现对于多层实景楼房的空间记忆和机器人可回退导航

## Problem
可扩展大型环境和物体目标层面抽象的要求存在限制：
- 语义3D地图：
	1.因为受到预定义的目标设置或者训练的语义推测模型的限制，往往工作在固定目录（fixed category）
	2.为了突破固定目录的限制，则刚需对每个几何物体存储 visual-language embedding，提高了存储代价
- 过往工作的局限:
	1.ConceptGraphs 仅仅在小场景验证且为人工验证


## Method
1.3D segment-level open-vocabulary map建立
从 RGB-D 视频中得到一批**物体级/片段级 3D segments**，并给每个 segment 分配一个开放词表视觉语言特征。论文强调，它没有给每个点都存一个特征，而是利用“相邻点通常语义一致”的假设，只在 segment 层面存特征，以减少存储开销
具体流程图类似：
RGB-
2D mask + depth + camera pose
        ↓SAM & 深度反投影
局部 3D segment
        ↓
变换到全局坐标系
        ↓
全局 3D segment candidate
2.依据开放词汇地图建立场景图scene graph

## Key Contributions

1.
2.
3.

## Experiments


## Limitations


## Connections


## Questions


