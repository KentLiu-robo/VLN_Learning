# 从 ApexNav、HOV-SG、VLFM、MSGNav 看具身大小脑 VLN 系统

## 关联论文笔记

- [[../papers/ApexNav An Adaptive Exploration Strategy for Zero-Shot Object  Navigation with Target-centric Semantic Fusion|ApexNav]]
- [[../papers/Hierarchical Open-Vocabulary 3D Scene Graphs  for Language-Grounded Robot Navigation|HOV-SG]]
- [[../papers/VLFM Vision-Language Frontier Maps  for Zero-Shot Semantic Navigation|VLFM]]
- [[../papers/MSGNav Unleashing the Power of Multi-modal 3D Scene Graph for Zero-Shot Embodied Navigation|MSGNav]]

## 一句话总结

这四篇工作可以看作 zero-shot / open-vocabulary 具身导航系统的四块拼图：VLFM 提供面向未知环境的语言价值 frontier 探索，ApexNav 强化探索策略切换和目标级语义融合，HOV-SG 提供分层开放词汇 3D 场景图与可执行导航图，MSGNav 进一步把 3D 场景图升级为保留图像关系证据的多模态长期记忆，并补足最后一段可见视角选择。

## 四篇工作的核心定位

| 工作      | 核心问题                                              | 核心表示                                                                | 工作重点                                                           | 系统角色          |
| ------- | ------------------------------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------- | ------------- |
| VLFM    | 未知环境中的 zero-shot semantic navigation 如何选 frontier | 2D frontier map + language-grounded value map                       | 用 BLIP-2 给 frontier / free space 赋语义价值，引导探索                    | 在线探索前端        |
| ApexNav | 语义线索不稳定、目标误检污染、局部导航安全性                            | semantic score map + object cluster + safe waypoint                 | 在语义探索和几何探索间自适应切换，并做 target-centric semantic fusion             | 探索策略与目标确认     |
| HOV-SG  | 大型真实环境中如何构建可查询、可导航、开放词汇的 3D 语义记忆                  | root-floor-room-object 分层 3D scene graph + Voronoi actionable graph | 从 RGB-D 构建分层开放词汇 3D 场景图，支持语言查询和跨楼层导航                           | 长期语义地图 / 空间记忆 |
| MSGNav  | 传统 scene graph 文本边丢失视觉关系，完整图输入 VLM 成本高            | object node + image edge 的 M3DSG                                    | 用图像边保留关系证据，结合 KSS、AVU、CLR、VVD 完成 zero-shot embodied navigation | 多模态记忆与高层推理    |

## 按 VLN 系统模块比较

### 探索：从最近 frontier 到语义驱动 frontier

VLFM 的关键贡献是把传统 frontier exploration 从“去最近未知边界”升级为“去最可能包含目标语义线索的未知边界”。它用 RGB-D 和里程计构建 2D obstacle map，提取 frontier，再用 BLIP-2 根据目标 prompt 给可观测区域打 value 分数，最后选择 value 更高的 waypoint。

ApexNav 接着补上一个重要问题：语义价值不总是可靠。目标可能还没出现，VLM 分数可能很平均，也可能被房间先验误导。因此 ApexNav 用 max-to-mean ratio 和 standard deviation 判断语义线索是否有区分度。线索强时走 semantic-based exploration，线索弱时回退到 geometry-based exploration。

两者可以形成互补：VLFM 负责产生连续的 value map，ApexNav 负责判断当前 value map 是否值得信任，并决定是语义探索、几何探索，还是多 frontier 全局访问规划。

### 目标识别：从单帧检测到长期目标级融合

VLFM 的目标检测流程是 YOLOv7 / Grounding-DINO + Mobile-SAM，再结合深度估计目标点。这种流程简洁有效，但对单帧检测、遮挡、相似物体和观察角度仍然敏感。

ApexNav 的 target-centric semantic fusion 更适合作为目标确认模块。它把物体实例作为 object cluster 维护，而不是只相信某一帧检测；每个 cluster 保留多个候选标签和置信度，通过点云体积加权、多帧融合、未再次检测惩罚，以及 LLM 给出的相似物列表和自适应阈值，区分 reliable target 和 suspected target。

这说明一个完善 VLN 系统不应只问“当前帧有没有目标”，而应维护“这个 3D 位置上的长期物体实例是否稳定支持目标判断”。

### 场景记忆：从任务特定 value map 到可复用 3D scene graph

VLFM 和 ApexNav 的语义地图更偏任务特定：目标变了，value map 或 semantic score map 也要重新围绕新目标更新。它们适合当前任务探索，但不一定天然适合作为 lifelong memory。

HOV-SG 则把环境组织成 `root -> floor -> room -> object` 的层级图。它的价值在于支持开放词汇查询，用 floor、room、object 层级贴近人类空间描述，用 Voronoi navigation graph 把语义查询结果转成可执行路径，并通过 segment / object / room 级存储降低长期记忆成本。

HOV-SG 更像是“大脑中的长期空间记忆”：它不只是为了当前目标服务，而是支持之后反复查询、跨房间推理、跨楼层导航。

### 多模态关系推理：从文本边到图像边

HOV-SG 的层级结构强，但关系表达仍然容易被压缩成类别、房间、空间归属等较粗粒度信息。MSGNav 的 M3DSG 把边从文本关系升级为 image edge：两个物体如果在同一帧共同出现，就把这张图像作为边的视觉证据保存下来。

这个设计解决了传统 scene graph 的一个关键问题：把视觉关系转成文字会丢失外观、遮挡、布局、材质、房间风格和上下文。MSGNav 让 VLM 在后续推理时重新看见关系证据，而不是只读“cup on table”这种简化文本。

同时，MSGNav 通过 KSS 解决大图推理成本，通过 AVU 扩展词汇，通过 CLR 引入闭环推理记忆，通过 VVD 选择能看清目标的最终视角。因此它更接近一个可查询、可推理、可行动的多模态大脑。

## 四篇工作的互补关系

| 能力 | VLFM | ApexNav | HOV-SG | MSGNav |
| --- | --- | --- | --- | --- |
| 未知环境主动探索 | 强 | 强 | 弱，偏建图后查询 | 中，可结合探索过程 |
| 语义 frontier 选择 | 强 | 强，且能切换策略 | 弱 | 可由图推理辅助 |
| 目标检测与确认 | 中 | 强 | 中 | 强，依赖 object node 和图像证据 |
| 长期语义记忆 | 弱 | 中，维护目标 cluster | 强 | 强 |
| 开放词汇能力 | 中 | 中 | 强 | 强 |
| 房间 / 楼层层级 | 弱 | 弱 | 强 | 中，可与 HOV-SG 结合 |
| 多模态关系保真 | 弱 | 弱 | 中 | 强 |
| 最后一段视角选择 | 中 | 中，强调安全 waypoint | 中 | 强，VVD 明确解决 last-mile |
| 真实机器人部署意识 | 中 | 强 | 强 | 中，仍偏 benchmark 系统验证 |

## 面向具身大小脑 VLN 系统的整合设想

一个更完善的 VLN 系统可以分成“大脑”和“小脑”：

```text
用户语言目标
    ↓
语义大脑：任务理解、长期记忆、图推理、探索意图
    ↓
行动小脑：局部建图、frontier 选择、路径规划、安全控制、视角调整
    ↓
机器人执行与观察反馈
    ↓
在线更新 value map / object clusters / 3D scene graph / decision memory
```

### 语义大脑：理解、记忆和推理

语义大脑应融合 HOV-SG 和 MSGNav 的优势：

1. 用 HOV-SG 的层级结构保存长期空间记忆：floor、room、object。
2. 用 MSGNav 的 image edge 保存物体关系的视觉证据，避免把关系过早压缩成文本。
3. 用 CLIP / VLM embedding 支持开放词汇查询。
4. 用 KSS 从大图中检索与目标最相关的子图，降低 VLM 推理成本。
5. 用 AVU 在探索过程中更新词汇，让系统能学习环境中的细粒度对象和新表达。
6. 用 CLR 保存历史探索结果，避免重复搜索已经排除的区域。

这个大脑回答的问题不是“下一步怎么走”，而是：

- 目标最可能在哪个楼层、房间、区域？
- 当前 scene graph 中有没有可靠目标？
- 如果没有目标，应该优先探索哪些未知区域？
- 哪些相似物体容易误导检测？
- 过去哪些区域已经查找失败？

### 行动小脑：探索、规划和安全执行

行动小脑应融合 VLFM 和 ApexNav 的优势：

1. 用 VLFM 的 frontier map 维护未知边界和候选 waypoint。
2. 用 VLFM / ApexNav 的 VLM semantic score map 给 frontier 打分。
3. 用 ApexNav 的线索强弱判断机制决定语义探索还是几何探索。
4. 当多个高价值 frontier 存在时，用 ApexNav 的全局访问顺序规划减少折返。
5. 用 ApexNav 的 safe waypoint navigation，在局部控制中同时考虑效率和 ESDF 安全代价。
6. 用 MSGNav 的 VVD 在最后一段选择可见性最高的观察点，而不是只去离目标最近的位置。

这个小脑回答的问题是：

- 当前局部地图中哪些 frontier 可达？
- 语义线索是否足够可靠？
- 下一目标 waypoint 是哪里？
- 如何安全地走过去？
- 到达目标附近后，应该停在哪个视角才能真正看清目标？

## 一个可能的完整系统流程

### 1. 语言目标解析

输入：

```text
Find the mug in the kitchen.
```

大脑解析出目标物体 `mug`、房间先验 `kitchen`、相似或易混淆物 `cup / bowl / bottle / container`，并确定查询优先级：先找 kitchen，再找 mug；若已有 mug 记忆，直接验证。

### 2. 长期图检索

系统先查询 HOV-SG / MSGNav 融合图：

1. 是否有 kitchen room node？
2. kitchen 中是否有 mug 或 cup-like object node？
3. 是否有 image edge 能证明该物体与 kitchen、table、sink、cabinet 等上下文相关？
4. 目标 node 的置信度是否足够可靠？

如果找到可靠目标，直接进入 viewpoint selection 和 navigation。  
如果只找到 kitchen，则导航到 kitchen 附近并局部搜索。  
如果连 kitchen 都没有，进入 frontier exploration。

### 3. 语义探索

小脑维护 frontier map，并根据 VLM value map 给未知边界打分。大脑提供房间和物体先验，例如：

```text
mug likely appears in kitchen, dining room, office, table, cabinet, sink area
```

当 frontier semantic score 差异明显时，使用语义探索；当分数接近、语义线索不可靠时，使用几何探索扩大观察范围。探索中不断更新 2D occupancy map、semantic value map、object clusters、HOV-SG 层级节点、M3DSG image edges 和 decision memory。

### 4. 目标确认

当检测到疑似目标时，不立即停机，而是进入 target-centric fusion：

1. 将检测结果和已有 object cluster 匹配。
2. 更新多标签置信度。
3. 根据观察体积和视角质量加权。
4. 如果重新观察却未检测到目标，降低旧判断置信度。
5. 用 image edge 和 VLM 进行上下文复核。

只有当目标 cluster 置信度超过阈值，且语义上下文合理时，才进入最终导航。

### 5. 最后一段导航与验证

系统不直接选择离目标最近的点，而是使用 VVD：

1. 围绕目标点云采样候选 viewpoint。
2. 计算目标可见性、遮挡、距离、朝向、安全代价。
3. 选择能看清目标且可达的最佳视角。
4. 到达后再次检测和 VLM 验证。
5. 成功后把目标、视角、图像边、房间关系写回长期记忆。

## 关键研究问题

### 如何把 HOV-SG 从离线建图改造成在线增量图？

HOV-SG 当前更偏预扫描后查询。未来系统需要边探索边更新 floor、room、object 和 navigation graph。难点在于 room 节点可能会随观察增加而合并 / 拆分，object node 也会因为遮挡和重复观测而变化。

一个可行方向是先在线维护 object-centric graph，再周期性做 room-level consolidation。也就是说，小脑实时更新 object clusters 和 frontier map，大脑每隔一段时间把局部 object / free-space / wall evidence 汇总成更稳定的 room nodes。

### 如何压缩 MSGNav 的 image edge 长期记忆？

图像边保留了丰富证据，但长期运行会带来存储和检索成本。未来可以为每条 edge 维护代表性图像集合，而不是保存所有共视帧：

- 保留视角差异最大的图像。
- 保留目标清晰度最高的图像。
- 保留能覆盖最多关系的图像。
- 对低价值、重复、模糊图像做遗忘。

这相当于给大脑加入“记忆压缩”和“代表性经验回放”机制。

### 如何让大脑和小脑形成闭环？

大脑不能只在任务开始时规划一次，小脑也不能只盲目执行 waypoint。更好的方式是：

```text
大脑给出探索意图和语义先验
小脑执行并返回观察证据
大脑根据证据修正目标区域
小脑重新规划 frontier / waypoint
```

例如大脑判断 mug 可能在 kitchen，但小脑探索 kitchen 后没有可靠目标，系统应更新 decision memory，并降低 kitchen 的优先级，转向 dining room 或 office。

### 如何统一目标概率、语义相似度和可达性？

VLFM / ApexNav 的 value score 更像语义相关性，HOV-SG / MSGNav 的图检索分数更像语义和关系可信度，而局部规划还需要考虑路径代价、安全性和可见性。

未来系统可以把候选区域评分写成一个统一目标函数：

```text
score(region) =
  semantic_relevance
  + graph_context_confidence
  + expected_information_gain
  + target_detection_confidence
  - path_cost
  - collision_risk
  - revisit_penalty
```

这样大脑和小脑共享同一套候选区域评价，而不是各自维护割裂的分数。

## 组合后的理想系统

理想的具身大小脑 VLN 系统应该具备这些能力：

1. 像 VLFM 一样在未知环境中主动探索，而不是被动等待目标出现。
2. 像 ApexNav 一样知道什么时候相信语义，什么时候回退到几何探索。
3. 像 HOV-SG 一样把环境组织成可查询、可复用、可导航的分层空间记忆。
4. 像 MSGNav 一样保留物体关系的视觉证据，让 VLM 能重新查看场景上下文。
5. 用长期 object cluster 和 image edge 共同确认目标，降低误检和相似物干扰。
6. 用 visibility-based viewpoint decision 解决最后一段“到了附近但看不清”的问题。
7. 用 closed-loop reasoning 把失败探索、错误检测、成功经验都写回记忆。

最终形态可以概括为：

```text
VLFM 负责发现哪里值得去
ApexNav 负责判断怎么探索和如何确认目标
HOV-SG 负责保存可查询的分层空间记忆
MSGNav 负责保存多模态关系证据并驱动高层推理

大脑做语义理解、长期记忆和图推理
小脑做局部地图、frontier 探索、安全规划和视角控制
```

## 我的理解

这四篇工作共同说明，未来的 VLN 不会只靠一个端到端 policy 解决。更可行的方向是模块化但闭环的具身智能系统：上层用开放词汇 3D scene graph 和 VLM 做长期语义推理，下层用 frontier map、occupancy map、ESDF 和 waypoint controller 做可执行、安全、实时的导航。

真正关键的是连接两者的接口：大脑输出的不是抽象文本计划，而应该是带置信度的目标区域、候选对象、关系证据和探索优先级；小脑返回的也不是简单成功 / 失败，而应该是可写回图记忆的观察证据、可达性、可见性和检测置信度。只有这样，系统才能从一次性导航走向 lifelong embodied navigation。
