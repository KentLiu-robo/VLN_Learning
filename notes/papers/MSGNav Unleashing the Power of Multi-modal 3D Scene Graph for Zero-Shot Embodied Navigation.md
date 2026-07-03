# MSGNav Unleashing the Power of Multi-modal 3D Scene Graph for Zero-Shot Embodied Navigation

## Metadata

- Paper:MSGNav: Unleashing the Power of Multi-modal 3D Scene Graph for Zero-Shot Embodied Navigation
- Authors:Xun Huang et. al.
- Year:2026 Mar.
- Link:https://arxiv.org/abs/2511.10376
- Area:Embodied Navigation
- Tags:3D Scene Graph, zero-shot embodied navigation

## One-Sentence Summary
个人认为：MSGNav实际上提出了使用图像代替文本作为边的多模态3D Scene Graph (M3DSG), 并借助这个场景记忆图实现了具身导航系统，同时完善了导航最后一段的位姿调整。

GPT5.5总结：MSGNav 是一个基于多模态 3D Scene Graph 的零样本具身导航系统。它不再把 3D 场景图中的物体关系简单压缩成文字边，而是用图像作为关系边来保留视觉证据，并结合关键子图选择、自适应词汇更新、闭环推理和可见性视角决策，实现开放词汇、多模态目标的 zero-shot embodied navigation

## Problem
1.传统3D场景图过度依赖文本转译，丢失视觉信息；主要带来三个核心问题：
- 构建成本高：消耗大量token和时间开销
- 视觉线索丢失：转译过程丢失外观、遮挡、布局、房间风格、目标细节等等视觉线索
- 词汇受限：对于closed-set的词表，对于未定义的物体表达能力不足
2.场景图增大，VLM推理成本太高：机器人不断探索，3D Scene Graph 会不断增长。如果每次都把完整图结构和大量图像交给 VLM，会导致推理慢、token 成本高、图像输入过多
3.依赖固定词表：要做到open-vocabulary
4.知道目标位置，不代表导航到合适的视角

## Method
### 1.M3DSG
object node + image edge: 两个物体之间的关系不是简单写成 “beside”“on”“near”，而是保存它们共同出现的 RGB-D 图像，让 VLM 在后续推理时直接看到视觉证据


## Key Contributions

1.
2.
3.

## Experiments


## Limitations




## Questions


