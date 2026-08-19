# GA-VLN: Geometry-Aware BEV Representation for Efficient Vision-Language Navigation

## Metadata

- Paper: GA-VLN: Geometry-Aware BEV Representation for Efficient Vision-Language Navigation
- Authors: Jiahao Yang, Zihan Wang, Xiangyang Li, Xing Zhu, Yujun Shen, Yinghao Xu, Shuqiang Jiang
- Year: 2026 May
- Link: https://arxiv.org/pdf/2605.22036
- Area: Continuous VLN / MLLM-based Navigation
- Tags: BEV, VLN, MLLM, RGB-D, VGGT, geometry-aware representation

## One-Sentence Summary

GA-VLN 提出 **Geometry-Aware BEV (GA-BEV)**：把历史 RGB-D patch features 通过深度投影到 agent-centric BEV grid，并融合 VGGT 这类 3D foundation model 的隐式几何特征，用紧凑 BEV tokens 替代 dense video tokens，从而提升 MLLM 导航的空间推理和效率。

## Problem

现有 MLLM-based VLN 多直接输入历史 RGB 视频 patch tokens：

- token 数量大，历史帧越多计算越重；
- patch tokens 是平铺的 image-centric 表示，缺少显式 3D 空间结构；
- 多视角历史观测之间没有几何对齐，容易造成冗余和空间不一致；
- 仅靠 MLLM 从视频 token 中隐式学空间关系，效率和泛化都不稳定。

核心问题：**如何把历史视觉观测压缩成既省 token、又有空间结构的导航表示？**

## Method

### 1. Explicit Depth-Guided BEV Projection

对每帧 RGB-D：

1. 用 SigLIP 提取 RGB patch features；
2. 将 depth resize 到 patch 分辨率；
3. 根据相机内参、agent pose 和 depth，把每个 patch center 反投影到 3D；
4. 再投影到地面 BEV 平面。

直观理解：不再把历史图像当作一串无结构 patch，而是把 patch feature 放回它在 3D 空间中的位置。

### 2. Implicit 3D Geometry Priors

论文额外引入 frozen **VGGT-1B**：

- 输入历史图像序列；
- 提取带有多视角几何先验的 features；
- 通过 2-layer MLP 对齐到 SigLIP feature dimension；
- 使用同样的 depth-guided projection 投影到 BEV。

作用：显式深度投影提供局部几何，VGGT 提供从大规模 3D reconstruction 预训练中学到的结构先验。

### 3. Grid-Based BEV Aggregation

将所有 RGB visual features 和 VGGT geometry features 投到 agent-centric BEV grid：

- BEV range: `[-10m, 10m]`
- grid size: `0.25m x 0.25m`
- 同一 grid cell 内 features 做 mean pooling；
- 只保留 non-empty cells；
- 加 2D sinusoidal position embedding。

这样把 dense historical visual tokens 压缩成 sparse BEV tokens。

### 4. GA-VLN Policy

GA-BEV tokens + 当前 front-view visual tokens + language instruction 输入 LLaVA-Video-7B，输出离散动作：

```text
{forward, turn left, turn right, STOP}
```

论文采用 two-round dialogue generation：

- 第一轮：用 instruction + current view + 最多 8 个历史观测聚合出的 BEV，预测 4 个动作；
- 第二轮：复用上一轮 BEV，只加入新的 front-view，继续预测 4 个动作；
- BEV 每 8 个动作更新一次，降低计算开销。

## Key Contributions

1. 提出 GA-BEV：用 depth projection + BEV aggregation 把历史视觉信息变成 3D-grounded、紧凑的空间表示。
2. 融合 VGGT 隐式 3D geometry priors，补充深度投影在噪声、稀疏、多视角结构上的不足。
3. 将 GA-BEV 接入 MLLM-based VLN，在减少 visual tokens 的同时提升 R2R-CE / RxR-CE 等连续 VLN benchmark 表现，且不依赖 DAgger 或 VQA 混合训练。

## Experiments

### Main Results

在 val_unseen 上，GA-VLN 取得：

| Benchmark | NE | OSR | SR | SPL |
| --- | ---: | ---: | ---: | ---: |
| R2R-CE | 4.80 | 67.6 | 61.0 | 55.2 |
| RxR-CE | 5.88 | 67.0 | 55.4 | 45.2 |
| NavRAG-CE | 7.88 | 46.4 | 22.2 | 18.2 |

和 StreamVLN / InternVLA-N1 等 image-based MLLM 方法相比，GA-VLN 的核心优势是：**更少 token + 更强空间结构**。

### Ablation

R2R-CE val_unseen：

| Method | BEV | VGGT Geo | SR | SPL | Latency |
| --- | --- | --- | ---: | ---: | ---: |
| Baseline | no | no | 51.49 | 46.18 | 342.9 ms |
| GA-VLN w/o VGGT | yes | no | 59.21 | 53.87 | 212.9 ms |
| GA-VLN | yes | yes | 60.96 | 55.19 | 258.7 ms |

结论：

- BEV projection 是主要增益来源：SR +7.72，且 latency 更低；
- VGGT geometry prior 进一步提升 SR / SPL，但带来少量额外计算；
- full GA-VLN 仍比 dense-video baseline 更快。

### Token Efficiency

在无 SRDF 的受限训练设置下：

- Baseline: 4003 visual tokens；
- GA-VLN w/o VGGT: 394 tokens；
- GA-VLN: 514 tokens。

也就是说，引入 VGGT 后 token 仍远少于 dense video baseline。

### Design Findings

- `0.25m x 0.25m` BEV grid 最均衡；
- `0.125m` 太细，token 多且冗余；
- `0.5m` 太粗，空间细节损失；
- 历史范围从 32 steps 增到 48 steps 有小幅收益，但更长会饱和或下降。

### Real Robot / Robustness

论文在 Hello Robot Stretch 3 上做了真实房间零样本测试。噪声实验显示，depth / pose / rotation noise 下 SR 下降小于约 2%，说明 BEV aggregation 和 VGGT 几何先验对轻度传感器噪声有一定鲁棒性。

## Limitations

1. **依赖 RGB-D 和位姿质量**：方法需要 depth、camera intrinsics、agent pose；如果 SLAM/pose drift 严重，BEV 对齐会受影响。
2. **不是 open-vocabulary object memory**：GA-BEV 是 dense feature map，不像 3D scene graph 那样显式维护 object node、relation edge、长期语义记忆。
3. **VGGT 引入额外模块**：虽然整体 latency 下降，但仍需要 3D foundation model 特征提取，部署成本比纯 image MLLM 高。
4. **BEV 是 2.5D 表示**：适合地面导航，但对多层结构、台面/柜子/遮挡层次的表达有限。
5. **动作层仍是离散 low-level action**：更多是跟随指令导航，不直接解决开放词汇目标搜索、frontier exploration 或 lifelong memory。

## Questions

1. GA-BEV 能否作为 MSGNav / M3DSG 的底层 metric memory？
   - GA-BEV 负责连续空间和几何压缩；
   - M3DSG 负责 object-level 语义记忆和 image-edge 关系。

2. 能否把 GA-BEV cell tokens 和 3D scene graph tokens 融合？
   - BEV cell token：适合局部可达性、空间布局；
   - object node token：适合目标语义；
   - image-edge token：适合对象关系和视觉证据。

3. 对 SoftNav 的启发：
   - SoftNav 注入 object/frontier soft tokens；
   - GA-VLN 注入 BEV spatial tokens；
   - 二者可以统一成：`BEV tokens + object/frontier tokens + graph relation tokens`。

4. 对 learning-based 3D Scene Graph 的启发：
   - 不一定只学 graph node/edge，也可以学 graph 与 BEV 的 cross-attention；
   - object graph 提供 sparse semantic anchors；
   - BEV 提供 dense geometry context；
   - 学习模块负责根据 instruction 选择最相关的 BEV region、object node、image edge 和 frontier。

## Takeaways

GA-VLN 的价值在于说明：**对 MLLM-based navigation 来说，减少 dense video tokens 不只是效率优化，也会提升空间推理。**

对当前研究方向，GA-VLN 可以作为 MSGNav/SoftNav 的互补模块：

```text
GA-BEV: compact metric geometry
SoftNav: 3D entity soft-token injection
MSGNav: object-centric M3DSG + image-edge visual evidence
```

一个自然后续方向是构建 mixed representation：用 GA-BEV 维护局部几何，用 M3DSG 维护长期语义关系，再通过 learned adapter / graph-BEV transformer 注入 VLM 或导航 policy。
