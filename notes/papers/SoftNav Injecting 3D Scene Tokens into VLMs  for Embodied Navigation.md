# SoftNav Injecting 3D Scene Tokens into VLMs for Embodied Navigation

## Metadata

- Paper: SoftNav: Injecting 3D Scene Tokens into VLMs for Embodied Navigation
- Authors: Yi Wu, Junjie An, Xiao Liu, Yiqun Zhou, Yuechen Wu, Xiaoqing Guan, Shuyang Yu, You Wang, Guang Li
- Year: 2026
- Link: https://arxiv.org/pdf/2607.14586
- Area: Embodied Navigation / Open-Vocabulary Object Navigation
- Tags: VLN, VLM, 3D scene token, soft token, frontier navigation, PQ3D, representation gap

## One-Sentence Summary

个人理解：SoftNav 的核心不是发明新的多模态注入范式，而是在 embodied navigation 中证明了一件具体的事：把 object/frontier 级别的 3D 连续表示直接投影成 VLM hidden space 里的 soft tokens，比把同样的 3D 信息序列化成文本 prompt 更适合导航决策，尤其能改善路径效率和跨 benchmark 泛化。

更短地说：SoftNav = frozen PQ3D 3D scene encoder + frozen Qwen2.5-VL + trainable MLP projector + LoRA，用少量 SFT 数据把 3D entity tokens 注入 VLM，让 VLM 基于视觉记忆、3D scene tokens 和文本目标选择 frontier waypoint。

## Problem

论文主要关注的是 3D 场景信息进入 VLM 决策模块时的 **representation gap**，而不是单纯提出一个新的导航 pipeline。

### 1. 现有导航范式的断裂

论文把已有方法分成几类：

1. **Modular / learned policy methods**
   - 代表：MTU3D / PQ3D。
   - 优点：有结构化 3D 感知，可以把 object 和 frontier 编成统一 query embeddings。
   - 问题：通常接一个专门训练的 policy，不能充分利用 VLM 的开放世界知识和语言推理能力。

2. **VLM-based navigation methods**
   - 代表：NavGPT、NaVid、VLM frontier selection 等。
   - 优点：有大模型推理能力，适合开放词汇、语言目标、多模态目标。
   - 问题：往往缺少稳定的 3D scene memory，或者只能看到当前图像/历史图像，缺少 object/frontier 级别的全局 3D 结构。

3. **3D-to-text prompting methods**
   - 代表：把坐标、scene graph、frontier、object list 转成文本 prompt。
   - 问题：3D 几何、空间关系、语义聚类和不确定性会被离散文本压扁。论文认为这个转换本身会产生可测量的性能损失。

### 2. 关键问题

SoftNav 要回答的问题是：

> 如果 3D encoder 已经产生了连续、结构化、任务相关的 entity embeddings，为什么还要先把它们翻译成文本，再让 VLM 从文本里重新理解空间结构？

这也是论文最有价值的切入点：不是问“有没有 3D 信息”，而是问“3D 信息以什么接口进入 VLM”。

### 3. 对 3D Scene Graph / MSGNav 的关联

这对 MSGNav-style M3DSG 很直接。MSGNav 用 image edges 保留视觉证据，但最终仍然依赖 VLM 查看被选中的图像和 prompt 做推理。SoftNav 的启发是：可以进一步把 object node、image edge、frontier 等实体编码成 soft graph tokens，让 VLM 不只是“看图 + 读文本”，而是能在 hidden space 里直接 attention 到 learned 3D graph memory。

## Method

SoftNav 的整体结构是两条输入流汇入 VLM：

1. **Visual stream**：当前/历史 RGB 图像通过 DINOv3 提取视觉 tokens，并压缩成固定数量的 visual memory tokens。
2. **3D scene stream**：RGB-D 流经 PQ3D，得到 object 和 frontier 的 3D query embeddings；这些 embeddings 经过 MLP projector 变成 VLM hidden space 中的 soft tokens。

最终输入形式近似为：

```text
visual memory tokens + projected 3D soft tokens + text instruction -> VLM -> target coordinate -> snap to frontier
```

### 1. Visual Memory

论文使用 frozen DINOv3 提取每帧图像特征，并用 CompressionHead + PatchMerger 压缩到每帧 15 个 tokens。

滚动窗口保存最近 16 帧：

```text
15 tokens/frame x 16 frames = 240 visual memory tokens
```

这部分提供 egocentric visual context，但它本身不是 SoftNav 最核心的创新，更像是给 VLM 保留最近视觉观测。

### 2. PQ3D 3D Scene Encoder

SoftNav 采用 MTU3D 中使用的 PQ3D 作为 frozen 3D scene encoder。PQ3D 从 streaming RGB-D 中维护：

- object instances；
- persistent object bank；
- occupancy map；
- exploration frontiers。

PQ3D 为每个 object 或 frontier 输出一个 768-d query embedding：

```text
q_i in R^768,  i = object or frontier
```

这里所谓 **entity-level 3D navigation token** 指的是：每个 token 对应一个导航实体，而不是一个图像 patch 或一个点云点。实体可以是 object，也可以是 frontier。每个 embedding 里包含几何、语义、空间关系和导航相关信息。

### 3. Soft-Token Injection

Soft token 不是词表中的离散 token，而是一组连续 embedding。SoftNav 用两层 MLP 把 PQ3D 的 768-d query embedding 映射到 Qwen2.5-VL-3B 的 2048-d hidden space：

```text
q_i: 768-d -> MLP Projector -> q_hat_i: 2048-d
```

MLP 结构：

```text
Linear(768, 1408) -> GELU -> Linear(1408, 2048) -> LayerNorm
```

中间维度 1408 是 768 和 2048 的中点，属于简单的维度渐进扩张 heuristic。

论文选择 MLP 而不是复杂 cross-attention 的原因：

- 训练数据很少，只有 1,187 个 SFT samples，复杂 adapter 容易过拟合；
- PQ3D query embedding 已经包含较强语义和空间信息，剩下主要是对齐到 VLM hidden space；
- 计算开销极低，projector 每步只有约 0.07 ms。

### 4. Scatter Injection and Token Ordering

实现上，文本模板中会插入 3D placeholder tokens。进入 embedding 层后，这些 placeholder 会被 projected 3D embeddings 替换。

输入序列为：

```text
H_input = [visual memory tokens; 3D soft tokens; text tokens]
```

soft-token block 里，论文采用 frontier tokens 在前、object tokens 在后的顺序。默认注入全部 3D tokens，最多 128 个；后续消融说明，hard top-k token selection 没有超过 full-token 注入。

### 5. Frontier Decision

VLM 不直接输出 low-level action，而是自回归生成一个目标坐标字符串，例如：

```text
"0.5, 3.2"
```

然后系统把这个坐标 snap 到 occupancy map 中最近的 eligible frontier：

```text
predicted coordinate -> nearest valid frontier -> local navigation
```

为了避免 VLM 预测一个过远 frontier，论文加入 distance-ratio guardrail，经验值为 r=2。如果 frontier detector 返回空集合，会触发 rescue fallback。

### 6. Training Strategy

训练时冻结：

- DINOv3 visual encoder；
- PQ3D 3D scene encoder；
- Qwen2.5-VL-3B backbone。

只训练：

- MLP projector，约 4M 参数；
- LoRA adapters，约 13M 参数；
- 总可训练参数约 17M。

训练数据来自 MTU3D 成功 episodes 中的 frontier selections。也就是说，SoftNav 本质上有一部分 teacher distillation：把 MTU3D 的 frontier coordinate 决策作为 VLM 的 SFT 标签。

训练目标是标准 autoregressive loss，让 VLM 生成目标 frontier coordinate string。

## Key Contributions

1. **量化了 text serialization 的 representation gap**
   - 论文控制 PQ3D backend 和 VLM 不变，只改变 3D 信息输入接口。
   - 结果显示 soft tokens 明显优于 Text-Hint / Rich-Text / Text-Hint + LoRA，说明问题不只是 prompt 不够详细，也不是 LoRA 参数量不足，而是文本接口本身损失了连续 3D 表示。

2. **提出 entity-level 3D soft-token injection for navigation**
   - 以前多模态注入并不新，LLaVA、BLIP-2、Flamingo、3D-LLM、LEO 都做过类似 bridge。
   - SoftNav 的具体价值在于把 PQ3D 产生的 object/frontier 级别 3D scene embeddings 注入 VLM，用于 online frontier-based embodied navigation。

3. **低训练成本地结合 3D perception 和 VLM reasoning**
   - 只用约 1,187 个 SFT 样本和约 17M trainable parameters。
   - 同时保留 PQ3D 的 3D grounding 能力和 VLM 的语言/开放世界推理能力。

4. **跨 benchmark 和真实机器人迁移**
   - policy 只在 OVON 上训练，但 zero-shot 迁移到 GOAT-Bench、SG3D 和 Unitree Go2 实机测试。
   - 这说明 PQ3D query embeddings + soft injection 捕获了较通用的导航语义，而不是只拟合一个 benchmark。

## Experiments

### 1. HM3D-OVON

SoftNav 在三个 split 上都取得最高 SR 和 SPL：

| Split | SR | SPL |
| --- | ---: | ---: |
| val_seen | 74.2 | 33.9 |
| val_seen_synonyms | 68.3 | 28.6 |
| val_unseen | 66.7 | 25.7 |

相对最强 prior MTU3D：

- val_seen SR: 55.0 -> 74.2，提升 +19.2 pp；
- val_seen_synonyms SR: 45.0 -> 68.3，提升 +23.3 pp；
- val_unseen SR: 40.8 -> 66.7，提升 +25.9 pp。

这里最值得注意的是 val_unseen 提升最大，说明 VLM reasoning + 3D soft tokens 对开放词汇和未见环境有帮助。

### 2. Zero-Shot Generalization

SoftNav 的 navigation policy 只在 OVON 上训练，然后直接迁移：

#### GOAT-Bench

| Split | SR | SPL |
| --- | ---: | ---: |
| val_seen | 67.2 | 29.0 |
| val_synonyms | 63.2 | 25.8 |
| val_unseen | 64.2 | 24.7 |

SoftNav 的 SR 超过 MTU3D 和 MSGNav，但 SPL 在部分 split 上低于最强 baseline。这说明它更容易成功，但路径不一定最短。

#### SG3D

| Metric | SoftNav |
| --- | ---: |
| s-SR | 47.2 |
| t-SR | 16.6 |
| SPL | 14.7 |

SG3D 是 scene-graph-grounded sequential navigation，SoftNav 在 s-SR 和 t-SR 上提升明显，但 SPL 低于 MTU3D。这和 GOAT-Bench 一样，说明成功率收益比路径效率收益更稳定。

### 3. Representation Gap Ablation

最关键的消融是 Table V：

| Input to VLM | val_seen SR | val_seen SPL | val_synonyms SR | val_synonyms SPL | val_unseen SR | val_unseen SPL |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Text-Hint | 66.7 | 24.7 | 66.7 | 27.0 | 63.3 | 28.3 |
| Rich-Text | 65.0 | 24.9 | 62.5 | 26.1 | 70.0 | 27.9 |
| Text-Hint + LoRA | 68.3 | 26.9 | 60.8 | 23.6 | 64.2 | 23.1 |
| Soft tokens | 74.2 | 33.9 | 68.3 | 28.6 | 66.7 | 25.7 |

关键结论：

- 在 val_seen 上，soft tokens 把 SPL 从 24.7 提到 33.9，提升 +9.2 pp；
- 加 LoRA 到 Text-Hint 只带来很小收益，说明不是“多训练一点 VLM”就能解决；
- Rich-Text 加入距离、方向、置信度后效果不稳定，说明文本 channel 对空间信息的利用不可靠；
- val_unseen 上 Rich-Text SR 名义上高于 SoftNav，但统计上不可区分，且 SPL 仍不占优。

这部分是论文最有说服力的实验：它把 SoftNav 的 claim 从“我设计了一个 adapter”变成“embedding-level 3D transfer 确实比 text interface 更合适”。

### 4. Token Selection Ablation

论文尝试 Instruction-Conditioned Token Selector，用 goal instruction 和 PQ3D tokens 做 cross-attention，再选择 top-k tokens。

结果：

| Variant | K | SR | SPL |
| --- | ---: | ---: | ---: |
| Full model | 128 | 74.2 | 33.9 |
| selector | 32 | 73.3 | 32.3 |
| selector | 96 | 68.3 | 30.6 |

结论：当前训练规模下，注入全部 tokens 最好。可能原因：

- PQ3D tokens 对导航是 dense informative，硬剪枝会丢信息；
- top-k 选择不可微/梯度流受限；
- 1,187 个 samples 不足以同时训练 selector、projector 和 LoRA。

这对 MSGNav/M3DSG 很重要：如果要做 learned subgraph selection，需要更好的训练信号和更平滑的 selection 机制，不能简单 top-k。

### 5. Efficiency and Real-World Testing

单步推理约 1 s：

- PQ3D 3D perception: 约 0.45 s；
- VLM inference: 约 0.49 s；
- MLP projector: 约 0.07 ms。

瓶颈不在 projector，而在 3D perception 和 VLM inference。

真实机器人测试：

- 平台：Unitree Go2 + Intel RealSense D435i；
- 场景：indoor corridor、building hall、outdoor area；
- 结果：30 次中成功 19 次，整体 63.3%；
- outdoor 环境不在室内训练分布中，仍能有一定成功率。

## Limitations

1. **多模态注入范式本身不新**
   - MLP projector + LoRA + soft token injection 已经在 LLaVA、BLIP-2、Flamingo、3D-LLM、LEO 等工作中出现。
   - SoftNav 的新意应限定为 navigation-specific 3D entity token injection 和 representation-gap ablation，不能过度表述为架构创新。

2. **依赖 PQ3D 表示质量**
   - 如果 PQ3D 没有检测到目标，或者 object/frontier embedding 质量差，SoftNav 也无法补救。
   - 论文 qualitative failure 中也指出，3D encoder cue 不可靠时，SoftNav 和 Text-Hint 都会失败。

3. **训练数据很少，空间规划能力可能不足**
   - 1,187 个 SFT samples 足以学习基本对齐，但可能不足以学到细粒度路径效率。
   - GOAT/SG3D 中 SR 高但 SPL 不一定最好，说明模型更偏“找到目标”，而不是“高效规划”。

4. **token selection 没有真正解决长期记忆压缩**
   - Full 128 tokens 效果最好，但长期探索、lifelong navigation 或更大场景中，object/frontier tokens 会持续增长。
   - 论文的 selector 消融失败，说明 learned selection 仍是开放问题。

5. **不是端到端 learning 的 3D scene graph**
   - SoftNav 使用 frozen PQ3D，学习的是 3D-to-VLM interface。
   - 它没有学习如何构建 scene graph、如何维护 memory、如何更新 object relation，也没有直接学习 image-edge relation。

6. **真实部署规模有限**
   - 实机测试只有 30 个 trials，且 perception/planning offboard。
   - 真实长期部署中的延迟、SLAM 漂移、动态障碍、光照变化、语义检测错误仍未充分评估。

## Questions

1. **SoftNav 的 representation gap 是否适用于 M3DSG image-edge？**
   - MSGNav 用 image edge 替代 text edge，已经避免了一部分文本化损失。
   - 但推理时仍然让 VLM 看少量图像和 prompt。是否可以把 image-edge 编码成 learned relation soft tokens，进一步减少图像输入和 prompt 依赖？

2. **M3DSG 中哪些实体应该被 token 化？**
   - object node token；
   - image-edge relation token；
   - frontier token；
   - room/region token；
   - visibility/viewpoint token；
   - uncertainty token。

3. **如何做比 SoftNav 更强的 learned selection？**
   - SoftNav 的 top-k selector 没有超过 full tokens。
   - 对 M3DSG 来说，不能只学 object/frontier relevance，还要学 edge relevance 和 visual evidence coverage。
   - 可能需要 soft attention / differentiable retrieval / contrastive subgraph ranking，而不是 hard top-k。

4. **能不能把 MSGNav 和 SoftNav 结合成 learning-based M3DSG？**

一个可能方向：

```text
M3DSG object nodes + image edges
  -> frozen CLIP/DINO/SigLIP edge image encoder
  -> graph transformer over object-edge-frontier tokens
  -> projector to VLM hidden space
  -> VLM/policy predicts target object, frontier, or viewpoint
```

训练信号可以来自：

- MSGNav 原始 VLM decision 的 distillation；
- oracle shortest path 的 frontier label；
- 成功 episode 的 final target object；
- view visibility / last-mile success label；
- 对比学习：goal text 与正/负 subgraph 的匹配。

5. **Soft token 和 explicit graph reasoning 如何结合？**
   - soft tokens 可提高连续表示和 VLM attention；
   - explicit graph 可保留可解释结构、可检索性、长期 memory 更新。
   - 比较合理的方案不是二选一，而是 explicit M3DSG 负责 memory，learned graph tokens 负责 VLM/policy interface。

## Takeaways for My Work

对当前想做的 learning-based 3D Scene Graph / MSGNav 扩展，SoftNav 的启发可以总结为：

1. 不要只把 3D scene graph 转成文本 prompt；这很可能是信息瓶颈。
2. 可以先冻结现有图构建系统，只训练 graph-to-VLM adapter，降低实验成本。
3. image-edge 不应只是被 VLM 查看的一张图，也可以编码成 task-conditioned relation token。
4. 学习重点应放在 interface、relation scoring、subgraph selection、frontier/viewpoint decision，而不是一开始端到端重训全部模块。
5. 论文真正值得复用的是 ablation 设计：固定 encoder 和 VLM，只比较 text / image / soft token / learned graph token 接口，才能证明“learning-based M3DSG”到底带来了什么。
