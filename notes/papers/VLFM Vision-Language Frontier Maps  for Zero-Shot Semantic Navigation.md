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
## Key Contributions

1.
2.
3.

## Experiments


## Limitations




## Questions


