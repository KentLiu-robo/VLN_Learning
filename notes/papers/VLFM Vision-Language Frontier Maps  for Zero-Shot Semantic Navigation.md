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
### 三步走：
1.Initialization: robot旋转建模frontier map和value map
2.Exploration:robot 根据value maps开始选择最具价值waypoint巡航，并不断迭代更新frontier & value map，重复选择和巡航，直到视野中出现target object
3.goal navigation: robot每次导航到目标物体方向最近的点，并在到达物体附近时STOP

#### Frontier Map的构建
使用深度图和里程计构建top-down 2D obstacle map:- 将当前深度图转换为点云；
- 过滤掉过低或过高的点，只保留可能构成障碍物的点；
- 将点云从相机坐标系变换到全局坐标系；
- 投影到 2D 网格地图中；
- 根据已探索区域和未探索区域的边界提取 frontier；
- 将每段 frontier 的中点作为候选 waypoint

#### Value Map的构建
Value Map和Frontier Map基本对齐，但是包含两个channel，一个是value分数，一个是confidence分数
VLFM 使用 BLIP-2 计算当前 RGB 图像与文本 prompt 的 cosine similarity。BLIP-2 输出图像和文本之间的相似度分数。分数越高，说明当前视觉观测越像是“前方可能有该目标物体”的场景。然后将这个分数投影到机器人当前视野对应的 top-down map 区域中

机器人从不同角度、不同距离看到同一区域时，VLM 给出的语义分数可能不同。VLFM 不直接用新分数覆盖旧分数，而是根据观察角度给每个像素一个 confidence。论文设定中，越靠近相机光轴的区域 confidence 越高；越靠近视野边缘，confidence 越低。原因很直观：图像中心区域通常更清晰、更可靠，而边缘区域可能畸变更大、语义信息更弱。当同一个地图区域被多次观察时，VLFM 使用带 confidence 的加权平均更新 semantic value。这样可以避免一次低质量观测或边缘视角观测直接破坏地图，也可以融合多帧信息。论文消融实验表明，加权平均优于直接替换和普通平均

#### 目标检测
对于 COCO 类别中的目标，使用 **YOLOv7**；对于非 COCO 或开放词汇类别，使用 **Grounding-DINO**。检测到目标后，再用 **Mobile-SAM** 从检测框中分割出目标轮廓，并结合深度图估计目标上距离机器人最近的点，作为最终导航 waypoint

#### Waypoint Navigation
在仿真实验中，VLFM 使用一个预训练的 **PointNav policy** 来执行到 waypoint 的低层导航。该 PointNav policy 使用 VER 算法训练，输入是深度图以及相对目标距离和方向，不使用 RGB 图像

在真实 Spot 机器人部署中，论文没有使用仿真中的 PointNav policy，而是使用 Boston Dynamics API 执行 waypoint navigation
## Key Contributions
1.Frontier Map转为Value Map-Based，更加符合人类寻找目标时的探索策略
2.使用VLM而非LLM，绕开视觉转为检测文本d
3.

## Experiments


## Limitations




## Questions


