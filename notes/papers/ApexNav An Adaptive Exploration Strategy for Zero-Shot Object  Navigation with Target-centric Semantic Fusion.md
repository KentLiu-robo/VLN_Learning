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

置信度惩罚：如果某个 object cluster 以前被检测为目标，但机器人后来又从新的视角看到了这个位置，却没有再次检测到目标，那么系统会认为：之前的目标判断可能不可靠，施加惩罚项，这样有效的避免了过去错误的积累。
导航策略：
- 如果 cluster 的最佳标签是目标类别，并且置信度超过 LLM 给出的阈值，那么它被认为是 **reliable target**，机器人会导航过去。
- 如果置信度不足，则只作为 **suspected target**，不会马上前往。只有当没有 frontier 可以继续探索时，系统才会考虑这些可疑目标。

### Semantic Score Map
构建语义分数地图，更好的判断哪些区域更具备探索的潜质，选择frontier。
- 使用 **BLIP-2** 对 RGB 图像和文本 prompt 计算图文匹配分数。为了提升远距离预测能力，ApexNav 不只问“前方是否有目标物体”，还会加入目标可能出现的房间信息。例如目标是 bed，则 prompt 可以包含 bedroom；目标是 toilet，则 prompt 可以包含 bathroom。房间名称由 LLM 推理得到。
- 系统把 BLIP-2 得到的语义相关分数投影到已观测的 free grids 上，并根据视角置信度加权。靠近光轴的区域权重更高，偏离光轴的区域权重更低。多帧之间则采用置信度加权平均更新语义分数

### Safe Waypoint Navigation
不仅要走较短的路径还要安全的走过去，故引入了action evaluation机制：
-  **efficiency cost**：鼓励动作朝向 A* 路径上的局部目标点移动，并惩罚让机器人远离局部目标的动作。
- **safety cost**：通过局部 ESDF map 计算动作路径附近到障碍物的距离。如果路径太靠近障碍物，cost 增大。最终选择总代价最低的动作


## Key Contributions

1.更合理的探索框架，根据线索的自动切换探索策略
2.更鲁棒的目标识别技术，维护目标和相似物体的长期记忆，通过 object cluster、多标签竞争、检测体积加权、多帧置信度更新和未检测惩罚，解决单帧检测和 max-confidence fusion 容易被误检污染的问题。
3.真实可部署性的提高，有safe waypoint的计算，不仅考虑最短路径，还利用 ESDF 安全代价降低碰撞风险，并在真实机器人实验中验证了可部署性。论文在 AgileX LIMO 机器人上进行了 6 次真实环境实验，目标包括 toilet、couch、chair、refrigerator、plant 和 TV，平均 SR 为 100%

## Experiments
论文在 HM3Dv1、HM3Dv2 和 MP3D 三个数据集上评估 ApexNav。
- 结果显示，ApexNav 在 HM3Dv1 上达到 SR 59.6、SPL 33.0；在 HM3Dv2 上达到 SR 76.2、SPL 38.0；在 MP3D 上达到 SR 39.2、SPL 17.8。与已有方法相比，ApexNav 在 HM3Dv1 和 HM3Dv2 上取得最优结果，在 MP3D 上 SPL 也优于 SG-Nav。
- 进一步解释：SR 的显著提升主要来自 target-centric semantic fusion 对误检的修正能力；SPL 的提升主要来自 adaptive exploration strategy 对不同环境探索效率的提升

## Limitations
1.依赖多个外部模型，工程复杂度较高
2.ApexNav 的探索切换机制虽然有效，但还不是一个真正学习到的策略，对阈值和语义分数质量仍较敏感
3.计算开销和实时性

## Questions
如何利用Frontier Ma

