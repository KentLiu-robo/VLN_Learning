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

GPT5.5如是说:
HOV-SG 的 3D Scene Graph 不是直接从网络预测出来的，而是一个**几何启发式分层 + SAM 分割 + CLIP 开放词表特征 + DBSCAN 特征聚合 + Voronoi 可导航图**组合起来的系统：先把 RGB-D 观测变成带 CLIP 特征的 3D segments，再用高度直方图分楼层、BEV-Watershed 分房间、overlap 分配物体，最后形成 `root → floor → room → object` 的层级图，并通过导航图把语义目标转化为机器人可执行路径。

## Problem
可扩展大型环境和物体目标层面抽象的要求存在限制：
- 语义3D地图：
	1.因为受到预定义的目标设置或者训练的语义推测模型的限制，往往工作在固定目录（fixed category）
	2.为了突破固定目录的限制，则刚需对每个几何物体存储 visual-language embedding，提高了存储代价
- 过往工作的局限:
	1.ConceptGraphs 仅仅在小场景验证且为人工验证

## Method
### segment-level open-vocabulary map建立
从 RGB-D 视频中得到一批**物体级/片段级 3D segments**，并给每个 segment 分配一个开放词表视觉语言特征。论文强调，它没有给每个点都存一个特征，而是利用“相邻点通常语义一致”的假设，只在 segment 层面存特征，以减少存储开销
**具体流程图**：
**RGB-D图像
	↓SAM
2D mask + depth + camera pose
	↓深度反投影
局部 3D segment
	↓坐标转换+多帧 3D segment 合并
全局 3D segment candidate**

**开放语义特征的获取**：
对于每个 2D mask，论文提取三种图像输入的 CLIP 特征：
1. **整张 RGB 图像的 CLIP feature**：提供全局上下文；
2. **mask bounding box crop 的 CLIP feature**：保留目标和周围背景；
3. **去除背景后的 masked crop CLIP feature**：更聚焦目标本身。
加权求和方式融合这三类特征
对于每个mask提取的特征映射到深度投影的3D点附近（**point-wise feature map**），对于重复观测到的3D点其特征直接平均
使用 DBSCAN 对这些点级 CLIP 特征进行聚类，然后找到多数簇，再选择距离多数簇均值最近的特征作为最终特征。

### 依据开放词汇地图建立场景图scene graph

#### 1.Floor Segmentation
沿高度轴构建 histogram，然后检测高度分布中的峰值，并用 DBSCAN 对峰值聚类，最后选出代表性的高度层。然后用高度向量中两个连续值作为一个floor的边界（floor & ceiling）。
floor节点同时会添加上text embedding"floor {#}",用作语义标识

#### 2.Room Segmentation
对于每个 floor point cloud，将其投影到鸟瞰图 BEV 平面，构造 2D histogram。这样 3D 房间布局被转换为 2D 平面结构。随后从 BEV histogram 中通过阈值提取墙体骨架mask。

**Room masks 的获取：**
得到墙体 mask 后会：
1. 对 wall mask 做膨胀；
2. 计算 Euclidean Distance Field, EDF；
3. 对 EDF 阈值化得到 isolated regions；
4. 以这些 isolated regions 作为 seeds；
5. 使用 Watershed 算法得到 2D room masks

每个 2D room mask 对应 BEV 平面一个区域，再结合该 floor 的高度范围，提取落在该 2D 区域和高度区间内的 3D 点，构成**room point cloud**。论文指出，这些 room point clouds 后续用于把 objects 关联到 rooms

**Room nodes的语义表征**：
通过该房间内的观察，提取这些图像的CLIP特征，使用K-means选出最具代表性的K个特征

**Room的分类**：通过CLIP特征与候选room categories的余弦相似度寻找最为接近的房间类型

#### 3，Object Identification
**物体分配给房间**
给定 room point cloud，首先会检查 object-level 3D segments 与候选 room 在 BEV 平面上的点云重叠部分。如果某个 object segment 与某个 room 有重叠，就把它分配给该 room；如果一个 segment 和所有 room 都没有重叠，则分配给欧氏距离最近的 room。

**合并操作**：
把具有显著 pair-wise partial overlap 且查询得到相同 object label 的 3D segments 合并。也就是说，如果两个 segment 在几何上高度重叠，并且语义上都被识别为同一种物体，就合并成一个 object node

### 可导航图Actionable Graph
每层楼构建 floor-level free space map
1. 将相机轨迹点投影到 BEV；
2. 将 floor point cloud 投影到 BEV；
3. 根据一定高度范围的点生成 obstacle map；
4. 用 floor/pose region 减去 obstacle region，得到 free space；
5. 在 free space 上构建 Voronoi graph。
将相机位姿投影为 BEV 点，并假设一定半径内的相邻节点可通行；同时用 floor-wise points 得到楼层区域，用特定高度范围内的点得到 obstacle map，最后生成 free space map 并构建 Voronoi graph

多楼层环境，HOV-SG 会提取被分类为 stairs 的区域中的相机位姿，形成 stair-wise graph。然后将 stair graph 与对应楼层的 Voronoi graph 中最近节点连接起来，从而得到 cross-floor navigational graph
## Key Contributions

1.**Open-Vocabulary**: 自然语言查询，无需预定义类别
2.**分层级抽象**：分为Root，Floor, Room, Object多层级的逻辑图，方便查找符合直观
3.**actionable map**：不停留在semantic map 而是可以让机器人执行导航任务，包括跨楼层任务

## Experiments
- **benchmark**: 论文在 Replica 和 ScanNet 上比较了 HOV-SG、ConceptFusion、ConceptGraphs 等方法。结果显示，使用 ViT-H-14 作为 CLIP backbone 时，HOV-SG 在 Replica 上取得 **mIoU 0.231、F-mIoU 0.386、mAcc 0.304**，在 ScanNet 上取得 **mIoU 0.222、F-mIoU 0.303、mAcc 0.431**，整体优于其他开放词表方法
- **实机实验**: 论文使用 Boston Dynamics Spot 机器人，在两层办公楼中进行真实环境测试。实验包含 **41 个物体目标、9 个房间目标和 2 个楼层目标**。结果显示，object 查询的图检索成功率为 **70.7%**，最终导航成功率为 **56.1%**；room 查询成功率为 **55.6%**；floor 查询为 **100%**。这说明 HOV-SG 不只是离线语义表示，还能结合 Voronoi navigation graph 支持真实机器人跨楼层导航
- **存储效率**：HOV-SG 用 segment/object/room/floor 节点存储语义特征，而不是给每个点、voxel 或 grid cell 存一个视觉语言 embedding。因此在 HM3DSem 的 8 个场景中，总存储量为 **1493 MB**，而 VLMaps 为 **6068 MB**，平均减少约 **75%** 的存储开销

## Limitations
1.流程复杂，超参较多，部署调参成本高
2.假设环境的静态，很难应对动态场景
3.房间语义容易受视觉相似性影响
## Questions
如何在exploration过程中建立这样的scene graph？
**对于 HOV-SG 这篇论文中的真实机器人导航实验来说，基本上是先需要对整个室内环境进行预扫描/建图，然后再基于建好的层级 3D Scene Graph 进行语言查询和导航**。它不是一个严格意义上的在线 SLAM 式系统
从方法扩展角度看，**边探索边建图完全可以实现**，只是需要把 HOV-SG 改造成增量式框架。其中 object-level 语义 map 最容易在线化，navigation graph 也可以局部更新；真正困难的是room/floor 的稳定在线分割、节点合并/拆分、动态环境更新以及语言目标驱动的主动探索。
举例分析：
用户指令：Find the mug in the kitchen.
1. 在已有 HOV-SG 中查找 kitchen 和 mug
2. 如果找到：
      直接规划到目标附近
3. 如果只找到 kitchen，没找到 mug：
      导航到 kitchen 内继续局部探索
4. 如果 kitchen 也没找到：
      根据 frontier exploration 探索未知房间
5. 每探索一段：
      更新 object nodes、room nodes、navigation graph
6. 一旦目标查询匹配成功：
      停止探索，转入目标导航

