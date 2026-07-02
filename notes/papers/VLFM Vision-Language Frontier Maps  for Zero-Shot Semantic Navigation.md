# VLFM: Vision-Language Frontier Maps  for Zero-Shot Semantic Navigation

## Metadata

- Paper:VLFM: Vision-Language Frontier Maps  for Zero-Shot Semantic Navigation
- Authors:Yokoyama et. al.
- Year:2023 Dec.
- Link:https://arxiv.org/abs/2312.03275
- Area:VLM (Exploration)
- Tags:VLM, Semantic Navigation, Frontier Map

## One-Sentence Summary
个人认为：VLFM是在 occupancy map 获得 Frontiers 的基础上，利用 VLM 作为 foundation model，使用目标物体包含的提示词文本和RGB-D图像，共同得到 language grounded value map , 并借此选择最优探索 frontier，最终实现机器人在未知环境和全新目标无图条件下的 zero- shot Navigation 。

## Problem
1.task-specific 类型的 Obj-Nav 往往依赖固定类别数据训练，较难进行泛化和真实部署。
2.Frontier-Based Exploration Method 往往是贪心算法选择最近frontier直到目标物体出现，但是没有语义判断能力。
3.基于LLM或文本语义的方法，往往将视觉信息强制转换到text，导致视觉线索的丢失，且需要大量的计算资源意味着刚需远程服务器。

## Method


## Key Contributions

1.
2.
3.

## Experiments


## Limitations




## Questions


