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
Object node：
- 物体唯一 ID；
- 类别；
- 3D 坐标；
- 2D bounding box；
- 语义 mask；
- 点云；
- CLIP visual feature；
- room location
使用 **YOLO-World** 做开放词汇检测，使用 **SAM** 生成实例 mask，使用 **CLIP** 提取视觉特征。然后根据空间相似度和视觉相似度，把当前帧中的物体与全局场景图中的已有物体进行匹配和合并

Image edge:
空间距离小于阈值的物体对，如果它们在同一帧中共同出现，则把这一帧图像加入它们之间的 edge image set; 避免频繁调用 VLM 生成关系文本，同时保留更丰富的关系信息

### 2.MSGNav
四个关键模块
#### KSS：Key Subgraph Selection
为了避免大量graph的引入VLM，论文中使用了KSS办法，对于每条边的检索只需要平均4张sub graph.实现从越来越大的 M3DSG 中选择与当前目标最相关的一小部分子图，让 VLM 只看关键内容。
如何实现？
- 压缩：先把完整图压缩成简单邻接表，只保留物体 ID、类别和邻接关系
- 聚焦：VLM 根据当前导航目标，从压缩图中选择 top-k 相关物体
- 剪裁：据选中的关键物体和关系边，从 M3DSG 中挑选最能覆盖相关边的图像。论文使用 greedy dynamic allocation，使少量图像覆盖尽可能多的重要关系


## Key Contributions

1.
2.
3.

## Experiments


## Limitations




## Questions


