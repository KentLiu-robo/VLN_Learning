# VLN 学习笔记：Awesome Visual-Language-Navigation

## Metadata
- Source: [Paper Survey 之 Awesome Visual-Language-Navigation (VLN)](https://kwanwaipang.github.io/VLN/)
- Author: Kwan Wai-Pang
- Original date: 2025-08-24
- Area: Vision-Language Navigation, Embodied AI, Vision-Language-Action
- Tags: VLN, VLA, Embodied AI, navigation, spatial reasoning, grounding

## VLN核心问题总结
VLN 的核心问题是让具身智能体根据自然语言指令，在真实或仿真的视觉环境中理解空间、识别目标、规划路径并执行动作；研究主线从早期的 seq2seq / speaker-follower，逐步发展到 Transformer 预训练、跨模态对齐、历史记忆、主动感知和 VLM/VLA 驱动的泛化导航。

## 1. VLN 是什么
Vision-Language Navigation (VLN) 关注“语言指令到导航行为”的闭环映射。智能体通常接收一段自然语言指令，例如“走出卧室，经过走廊，在厨房门口右转”，然后根据视觉观测和自身历史轨迹选择动作，最终到达目标位置或定位目标物体。
VLN 与纯视觉导航不同：它不只是找最短路，而是要把语言中的地标、方向、关系、动作顺序和场景常识与视觉观测对齐。它也不同于静态 VQA，因为 VLN 需要连续决策、路径规划和动作执行，错误会在时间上累积。

可以把 VLN 拆成四个子能力：
1. 语言理解：解析路线、目标、地标、空间关系和约束
2. 视觉嵌入：把词语中的对象、房间、方向和场景线索对应到视觉输入
3. 历史记忆与规划：结合过去经过的位置，判断当前状态和下一步动作
4. 动作执行与终止：选择移动、转向、观察、停止，必要时输出目标 bounding box

## 2. 任务类型
| 类型 | 目标 | 典型特点 |
| --- | --- | --- |
| 指令跟随导航 | 按自然语言路线到达目标点 | R2R 是经典设定，强调路径与指令对齐 |
| 远程目标定位 | 根据描述找到并定位目标物体 | REVERIE 同时要求导航和目标 grounding |
| 对话式导航 | 通过问答或对话补充信息 | CVDN、Aerial Vision-and-Dialog Navigation |
| 场景/需求导向导航 | 根据场景需求寻找可满足需求的物体 | 需要常识推理和开放词汇理解 |
| 空中/城市导航 | 无人机或城市尺度导航 | 视角、尺度、地图和路径约束更复杂 |
| 通用具身导航 | 用 VLM/VLA 统一多类导航任务 | 近年重点，强调泛化和多任务统一 |

## 3. 场景类型
VLN 早期主要集中在室内环境，例如 Matterport3D 中的房间、走廊和楼梯。随后逐渐扩展到：
- 室内真实环境：对象密集，语言地标清晰，但遮挡和视角变化明显。
- 室外/城市环境：尺度大，路径长，路标和地图信息更重要。
- 空中导航：无人机视角带来连续运动、俯视观测和高度变化。
- 机器人真实部署：需要考虑传感器噪声、运动控制误差和 sim-to-real gap。

## 4. 数据集与模拟器

### 模拟器

VLN 常用模拟器提供可交互环境、视觉观测、导航图、动作空间和评测接口。早期任务大量基于 Matterport3D / Habitat 等室内环境；近期任务进一步扩展到无人机、城市和真实机器人平台。

### 代表数据集

| 数据集 | 任务侧重 | 备注 |
| --- | --- | --- |
| R2R | 室内指令跟随导航 | 经典 VLN benchmark，目标是按路线指令到达终点 |
| R4R | 更长路线、更复杂指令 | 路径和语言更长，更考验长程依赖 |
| RxR | 多语言、更细粒度路径对齐 | 支持跨语言 VLN，指令更丰富 |
| REVERIE | 导航 + 远程物体定位 | 不仅要到达，还要定位目标物体 |
| SOON | 场景导向目标导航 | 强调 graph exploration 和目标搜索 |
| CVDN | 视觉-对话导航 | 导航过程中利用对话历史 |
| AerialVLN | 空中 VLN | 面向 UAV 视角和空中路线 |
| OpenUAV | 开放无人机导航 | 更贴近无人机开放环境 |
| CityNav | 城市尺度导航 | 关注大规模城市空间 |
| AeroVerse | 空中/开放环境导航 | 用于扩展空中导航和泛化研究 |

## 5. 评估指标

| 指标 | 含义 | 关注点 |
| --- | --- | --- |
| Navigation Error (NE) | 最终位置到目标的距离 | 终点是否接近目标 |
| Success Rate (SR) | 是否在阈值内成功到达 | 最直观的成功率 |
| Oracle Success Rate (OSR) | 路径中曾经接近目标即视为 oracle 成功 | 反映探索过程中是否经过正确区域 |
| SPL | Success weighted by Path Length | 同时考虑成功和路径效率 |
| nDTW / SDTW | 路径与参考轨迹的相似度 | 衡量是否按指令路线行走 |
| CLS | Coverage weighted by Length Score | 衡量路径覆盖参考轨迹的程度 |
| RGS / RGSPL | REVERIE 等任务中目标 grounding 成功率 | 同时评估导航和物体定位 |

阅读指标时 Tips：高 SR 不一定代表遵循了指令路线；高 nDTW/SDTW 更能说明路径与语言描述一致；SPL 会惩罚绕路，因此对高效规划更敏感。

## 6. 模型训练范式

### 6.1 模仿学习

模仿学习把专家轨迹作为监督信号，训练智能体从当前观测和指令预测下一步动作。优点是稳定、易训练；问题是容易出现 exposure bias，测试时一旦偏离专家轨迹，模型可能不知道如何恢复。

常见方法：
- behavior cloning：直接学习专家动作。
- teacher forcing / student forcing：训练时控制使用专家动作或模型动作的比例。
- speaker-follower 数据增强：用 speaker 生成额外指令-轨迹对，缓解数据稀缺。

### 6.2 强化学习
强化学习通过奖励函数优化导航策略。奖励通常与到达目标、缩短距离、遵循路径或完成 grounding 有关。优点是可以直接优化任务指标；难点是奖励稀疏、探索困难、训练不稳定。

VLN 中常见做法是先用模仿学习预训练，再用强化学习微调

### 6.3 跨模态对齐
跨模态对齐是 VLN 的核心。模型需要把语言 token、视觉区域、历史位置和候选动作放进统一表示空间。早期使用 attention 和 seq2seq，后来发展为 multimodal Transformer、CLIP 对齐、VLM 语义推理和大规模预训练。

关键问题：
- 语言中的 landmark 是否能对应到图像区域。
- 动作历史是否能帮助判断当前指令执行到哪一步。
- 视觉特征是否具有足够的开放词汇和场景常识。

### 6.4 辅助监督学习
辅助任务用于让模型学习更稳的中间能力，例如：
- 预测下一步是否接近目标。
- 判断当前视角是否与指令片段匹配。
- 重建路径顺序或历史状态。
- 做 masked language / masked region modeling。
- 用对比学习拉近匹配的图文或轨迹表征。

## 7. 代表论文脉络

### 7.1 阅读主线

这一章可以按下面这条线来读：

1. 先看任务怎么被定义：R2R、REVERIE、SOON、CVDN 这些工作把 VLN 从“按指令走到终点”扩展到目标定位、场景需求和对话导航。
2. 再看模型怎么变强：从 Seq2Seq / Speaker-Follower，到 history memory、Transformer、VLN-BERT、PREVALENT 这类预训练和跨模态对齐方法。
3. 再看泛化怎么处理：REM、FDA、AuxRN 这类工作主要在数据增强、辅助监督和更稳的中间能力上下功夫。
4. 最后看 foundation model / VLA 时代：NavGPT-2、Uni-NaVid、NaVILA、HCM、DDN 这类工作把 VLN 推向更通用的具身导航和机器人落地。

### 7.2 论文梳理总表

| 主线 | 论文 | 解决什么问题 | 记忆点 |
| --- | --- | --- | --- |
| 任务起点 | Vision-and-Language Navigation | 提出 R2R 和 Matterport3D Simulator | VLN 起点：语言指令 + 视觉环境 + 连续动作 |
| 早期经典模型 | Speaker-Follower Models for VLN | 用 speaker 生成指令并辅助 follower 选路 | speaker 可做数据增强，也可做候选路径重排 |
| 目标定位 | REVERIE | 从到达目标点扩展到远程目标物体定位 | 不仅要走到附近，还要找到目标物体 |
| 场景需求 | SOON | 根据场景描述寻找合适物体 | 强调图探索和语义推理 |
| 对话导航 | CVDN | 用多轮问答补充导航信息 | 从静态指令变成对话协作 |
| 泛化增强 | Random Environmental Mixup | 通过环境混合提高 unseen 泛化 | 让模型少记场景分布，多看语言和视觉线索 |
| 历史记忆 | HAMT | 显式建模完整导航历史 | 从当前感知驱动走向记忆驱动导航 |
| 图文预训练 | VLN-BERT | 把网页图文对预训练迁移到 VLN | 先学图文 grounding，再做导航 |
| 通用预训练 | PREVALENT | 用 image-text-action triplets 做 VLN 预训练 | 预训练—微调范式进入 VLN |
| 辅助监督 | AuxRN | 通过自监督辅助任务理解导航状态 | 不只是动作分类，还要理解自己走到哪了 |
| 主动感知 | Active Visual Information Gathering | 不确定时主动探索再决策 | 像人一样先探一探路 |
| 空中导航 | Aerial Vision-and-Dialog Navigation | 把对话导航扩展到无人机视角 | 连续空间、俯视图、多轮对话 |
| 频域增强 | FDA | 用高频细节增强视觉—语言匹配 | 不只看布局，也看边缘、纹理、局部细节 |
| 机器人 VLA | NaVILA | 用上层 VLA + 下层 locomotion policy 做腿式机器人导航 | 中层动作连接语言推理和底层控制 |
| 综述脉络 | VLN today and tomorrow | 总结 foundation model 时代的 VLN 新方向 | world model、human model、VLN agent、behavior analysis |
| LLM 导航推理 | NavGPT-2 | 让 VLM 生成导航推理并接 policy network | VLM/LLM 负责推理，policy 负责选下一步 |
| 连续控制 | HCM / Robo-VLN | 分层处理高层推理和低层控制 | 高层子目标 + 低层速度控制 |
| 统一任务 | Uni-NaVid | 用视频式 VLA 模型统一多类导航任务 | VLN、ObjectNav、EQA、Human-following 放进同一框架 |
| 需求驱动 | DDN | 从“找指定物体”变成“根据需求找合适物体” | 需求文本 -> 属性空间 -> 目标物体 |

### 7.3 逐篇论文笔记

#### 7.3.1 基础任务与任务扩展

##### Vision-and-Language Navigation: Interpreting visually-grounded navigation instructions in real environments

1. 提出 Matterport3D Simulator，这是一个基于真实室内全景图像的交互式强化学习环境。它使用 Matterport3D 数据集中的真实建筑扫描数据，而不是纯合成场景，因此视觉纹理、家具、物体和空间布局更接近真实世界。
2. 提出 Room-to-Room（R2R）数据集。这是第一个面向真实建筑环境的视觉语言导航基准数据集，包含 21,567 条众包自然语言导航指令，平均每条指令约 29 个词，基于 7,189 条路径，平均路径长度约 10 米。
3. 建立了早期 VLN baseline：一个基于 LSTM Seq2Seq + Attention 的导航智能体，用于验证该任务的难度和可行性。

论文比较了两种训练方式。

```text
Teacher-forcing：训练时每一步都使用专家轨迹中的正确动作作为下一步输入。优点是稳定，缺点是模型只见过正确路径上的状态，测试时一旦走偏就容易崩溃。

Student-forcing：训练时下一步动作从模型自己的输出分布中采样。这样模型会进入自己造成的错误状态，更接近测试场景，论文结果显示 student-forcing 整体优于 teacher-forcing
```

优点是开创性非常强。它把视觉、语言和动作统一成一个可评测任务，并提供了 R2R 这一长期使用的标准 benchmark。
局限也很明显：

- 导航是离散图节点跳转，不是真实连续控制；
- 没有低层动力学、碰撞、避障和机器人运动约束；
- 模型只使用语言 attention，没有深入做视觉 attention；
- 指令只要求到达目标，不严格评价是否忠实遵循路径；
- 与真实机器人部署仍有 sim-to-real gap。

##### REVERIE: Remote Embodied Visual Referring Expression in Real Indoor Environments

在 R2R 基础上，要完成更高难度的目标，寻找初始就不在视野之中的物体定位。
目标物体在起点是无法被观测到的，这意味着机器人必须具有“常识”和“推理能力”，以到达目标可能出现的位置。

目标任务：Find the green book on the master bedroom's table
三层维度：
1. 地点导航：Where is the bedroom?
2. 目标定位：Find the green book
3. 跨房间推理：Which bedroom is the master bedroom? 场景推理

Semantic navigation：语义导航，需要模型具备基本的常识推理能力，例如 find the microwave 应该优先前往 kitchen 而非 bedroom。
REVERIE 被认为是第一个真正把“目标物体定位”引入 VLN 的经典基准，也是后来 ObjectNav、Embodied Referring Expression Grounding、VLA 等方向的重要起点。

##### SOON: Scenario Oriented Object Navigation with Graph-based Exploration

SOON（Scenario Oriented Object Navigation） 解决的是：“根据真实场景任务需求，找到一个符合描述的目标物体并完成导航。”
物体没有被预先标注，而是根据实际场景自行判断，所以格外强调 Semantic Reasoning 的引入，需要从场景需求出发，找到对应的物品。
Find the bed -> Find a place where a person could sleep

突出贡献：

1. Graph-Based Exploration (GBE):

```text
G = (V, E)
V: Viewpoint
E: Entrance，可通行连接
```

   智能体会一边探索一边构图，形成 memory graph。决策时参考：
   - 历史地图
   - 当前观察
   - 语言目标

2. SOON 和 REVERIE 任务相同：根据指令在 3D 环境中找到目标物体。
   - 之前的任务指令起始位置是固定的，然后指令是 step-by-step 地指导 agent 导航至某个位置。
   - SOON 不依赖于起始位置，它的指令是针对目标物体的由粗到细的描述，所以可以不依赖于 agent 起始位置。

##### Vision-and-Dialog Navigation （CVDN）

CVDN（Cooperative Vision-and-Dialog Navigation） 可以理解为：在 VLN 中加入“对话求助”机制，让智能体不再只依赖一条固定指令，而是可以通过多轮问答逐步获得导航信息。

1. 数据集中的两个角色：
   - Navigator：在环境中移动，看得到第一人称视觉观察，但不知道完整最优路径。
   - Oracle：不直接移动，但拥有关于最短路径/下一步方向的特权信息，可以回答问题。

Navigator 遇到不确定情况时向 Oracle 提问，Oracle 根据全局路径信息给出帮助。论文将这种设置称为 Cooperative Vision-and-Dialog Navigation。

2. 任务的形式：

NDH（Navigation from Dialog History）

NDH 中模型通常不是主动生成问题，而是利用已有的人类对话历史来预测导航动作。

```text
Navigator: Should I go upstairs?
Oracle: Yes, head up the stairs.
Navigator: I see two rooms, left or right?
Oracle: Go into the right room.
```

例如根据上述对话模型需要据此继续导航至下一目标场景/地点

3. 模型方法：多模态 Seq2Seq 模型
   - 对话历史用 LSTM/序列编码器建模；
   - 当前视觉特征来自 Matterport3D 视角图像；
   - decoder 根据视觉、语言和历史动作预测下一步导航动作。

CVDN 的核心贡献是把 VLN 从“静态指令跟随”扩展为“多轮对话协作导航”，让智能体能够利用人类问答中的动态信息完成导航任务，是交互式具身智能的重要早期基准。

##### Speaker-Follower Models for Vision-and-Language Navigation

不仅训练一个“Follower”根据指令走路，还训练一个“Speaker”根据路径生成指令；Speaker 既可以生成更多训练数据，也可以在推理时帮助 Follower 判断哪条候选路径最符合原始指令。

问题在于，R2R 这种人工标注的路径—指令数据很贵，规模有限。VLN 又要求 agent 在未见过的新环境中泛化，因此只靠少量人工指令很容易过拟合。
Speaker-Follower 的关键想法是：既然我们有路径和对应指令，那么可以反过来训练一个模型给路径写说明。

Follower 就是传统的 VLN-agent，这里由于时代局限性，还是使用的 seq2seq 架构。

Speaker 的关键作用：

1. 数据增强：

```text
采样新路径
↓
Speaker生成对应语言指令
↓
得到synthetic instruction-path pairs
↓
加入训练集训练 Follower
```

2. Pragmatic Reasoning

```text
候选路径 A
候选路径 B
候选路径 C
↓
Speaker 判断：
哪条路径最可能生成原始指令？

筛选路径反向推理
```

3. Panoramic Action Space：

Speaker-Follower 使用 panoramic action space：agent 在每个 viewpoint 可以看到 360° 全景，并直接选择一个相邻可导航 viewpoint 作为下一步动作。

在 foundation model 之前，Speaker-Follower 已经提出了“生成式语言模型可以反向增强导航模型”的思想。

#### 7.3.2 泛化与数据增强

##### Vision-Language Navigation with Random Environmental Mixup

为了应对模型在 seen 和 unseen 的场景中显著的差异，提高泛化性，作者借鉴 CV 中的 MixUp 思路，提出 Random Environmental Mixup (REM)。
本质上，是构造一个环境上的 out-of-distribution，强迫模型学习语言指令与视觉线索，而不是训练集的规律，避免过拟合环境。

操作上：
- 打乱环境分布，生成一个原来没有的新房间布局
- 不改变模型结构，不改变训练方法，改变训练数据

与后续的 Domain Randomization 的思想高度一致。

#### 7.3.3 历史记忆、预训练与辅助监督

##### History Aware Multimodal Transformer for Vision-and-Language Navigation
VLN 领域从 LSTM时代进入Transformer时代 的代表性工作之一，HAMT（NeurIPS 2021） 则首次系统解决了：VLN中长期历史信息（History）无法有效利用的问题。

核心思想：
1. 把整个导航历史保留下来：以往作为单个历史向量 seq2seq，现在作为完整的 memory tokens 然后输入给 Transformer。
2. 核心贡献：History Encoder 将 History Memory 每一步保存 视觉特征 + 位置特征 + 动作特征
3. 主要是解决了三类问题：(1) 长期依赖 (2) 路径回溯 (3) 探索记忆

HAMT 的核心贡献是提出 History-Aware Multimodal Transformer，将整个导航历史作为显式记忆输入 Transformer，通过语言—视觉—历史三模态联合注意力机制解决 VLN 中长期依赖、重复探索和历史遗忘问题，是 VLN 从“当前感知驱动”迈向“记忆驱动导航”的代表性工作。

##### Improving vision-and-language navigation with image-text pairs from the web
VLN-BERT：将 ViLBERT 式的图文预训练迁移到 VLN 中，用 Conceptual Captions 这类网页图文对来增强导航模型的视觉语言 grounding 能力。

- 类似 ViLBERT 的区域特征：先从全景图中生成多个 perspective views，再用 bottom-up attention object detector 提取 image regions 和 region features；这些 image regions 被当成视觉 token，与语言 token 通过 co-attention Transformer 进行融合。
- 可以理解为把语言部分的 BERT 和视觉部分的 ViLBERT 思路接到 VLN 任务里，让导航模型先具备一定图文对齐能力。
- 这里的 web 数据主要指网页图像-文本对，比如 Conceptual Captions，不是直接拿网页本身做导航数据。

补充：

ViLBERT 框架：

```text
Text Stream        Image Stream
    │                  │
    ▼                  ▼
Text Transformer   Visual Transformer
    │                      │
    └──── Co-Attention ────┘
             │
             ▼
     Vision-Language Fusion
```

1. 文本分支负责理解句子；
2. 图像分支负责理解图像区域；图像并非整个图像，而是分割后的区域 token。
3. Co-Attention 负责让文本和图像互相对齐；Co-Attention 中是词和图像区域之间互相看，这样模型就能学习视觉和语言之间的 grounding。

总结：ViLBERT 是一种视觉语言 Transformer 模型，它分别编码文本 token 和图像区域 token，并通过 co-attention 机制学习二者之间的对应关系，从而提升 VQA、图文匹配、视觉 grounding 和 VLN 等任务中的跨模态理解能力。

##### Towards Learning a Generic Agent for Vision-and-Language Navigation via Pre-training

核心意义在于：比较早、也比较系统地把“预训练—微调”范式引入 VLN 任务，试图训练一个可以迁移到不同导航任务中的通用视觉语言导航智能体。
重点在于通过预训练，让模型先学到一些视觉—语言—动作之间的基础对应关系，然后再通过后续训练提高 unseen 环境的准确率：先通过大量 image-text-action triplets 进行自监督预训练，学习通用表示，然后再迁移到具体 VLN 任务中。（PREVALENT）

- VLN 场景的数据：预训练数据主要来自 R2R 训练集和通过 Speaker 模型在训练环境中合成的指令。论文构造了两类预训练数据：第一类是 R2R 训练集中的约 104K image-text-action triplets；第二类是使用 Speaker 模型为训练环境中的最短路径合成约 1,020K 条指令，最终形成约 6.582M image-text-action triplets。
- 输入表示：Image-Text-Action Triplet，图像-文本-动作三元组。
- 整体架构表示：

```text
Instruction Tokens
        ↓
Text Transformer

Panoramic View Tokens
        ↓
Vision Transformer

Text Features + Vision Features
        ↓
Cross-modal Transformer
        ↓
Joint Representation
```

- 两类预训练任务：
  1. Image-attended Masked Language Modeling：根据语言上下文和视觉状态。
  2. Action Prediction：模型根据当前视觉状态和语言指令预测专家动作。

局限性：
1. 它的预训练数据仍然主要来自 R2R 训练环境和 Speaker 合成数据，不是真正大规模开放世界数据。因此它的“通用性”仍然受 Matterport3D / R2R 分布限制。
2. 动作预测不利用完整历史。论文里 action prediction 主要根据当前视觉图像和指令进行预测，不参考完整 trajectory history。这也解释了为什么后来 HAMT 要进一步引入 History Tokens。
3. 它主要学习的是 encoder 表示，仍然需要接入下游任务的导航 decoder / policy。也就是说，PREVALENT 本身不是一个完整的端到端通用机器人控制系统，而是一个用于增强 VLN agent 的预训练表示模块。
4. 它仍然处于离散 VLN 框架中，基于 Matterport3D viewpoint 图导航，没有解决连续控制、动力学、避障和真实机器人部署问题。

##### Vision-Language Navigation with Self-Supervised Auxiliary Reasoning Tasks

AuxRN：核心思想不是换一个更大的主干网络，而是给 VLN 智能体额外设计自监督辅助推理任务。
论文提出 AuxRN 包含四个辅助目标：解释过去动作、估计导航进度、预测下一方向、评估轨迹一致性。
论文中指出，之前方法虽然能编码视觉、语言和历史信息，但仍然存在几个问题：
- 过去动作会影响未来动作；
- 模型不能显式判断轨迹是否和指令对齐；
- 模型不能准确估计当前完成进度；
- 模型对导航图和下一动作后果的理解也不充分；
所以作者的出发点是：
VLN 不应该只训练“动作分类器”，还应该训练一个能理解自身导航状态的智能体。

四个辅助任务如何完成的：
1. Trajectory Retelling，轨迹复述：这个任务让模型根据已经走过的视觉轨迹，重新生成/解释语言指令中的词语，比如 walk away from the bedroom and head upstairs，这种描述能够让模型更好理解自己做过什么。
2. Progress Estimation，进度估计：作者使用基于步数百分比的软标签，比如第 t 步占总步数 T 的比例，并使用 BCE loss 而不是 MSE，可以回答：我到哪一步了
3. Cross-modal Matching，跨模态匹配：以 0.5 的概率随机打乱 batch 中的全局语言特征，然后让模型判断当前视觉语言轨迹表示与语言是否一致。也就是说，模型要区分轨迹和指令的匹配，能够回答：我现在走的路线和指令一致吗？
4. Angle Prediction，角度预测：这个任务让模型预测下一步 teacher action 对应的转向角度，能够回答：我下一步大概朝哪个方向走？

从自动化学科知识来看，AuxRN 有点像给控制器加了几个状态估计器和自监督诊断模块

局限性明显：
1. 仍然是 R2R 离散图导航，不涉及连续控制、真实机器人动力学、避障和低层运动控制。
2. 它的 backbone 仍然是 LSTM + attention，而不是后来更强的 Transformer / BERT / CLIP-ViT 体系
3. trajectory retelling 和 angle prediction 依赖 teacher trajectory，在 RL/student forcing 场景下不一定都能使用。论文也说明 trajectory retelling 不在 RL 场景中训练，angle prediction 由于需要 teacher angle，也不在 RL 中使用。
4. Cross-modal Matching 使用 batch 内 shuffle 构造负样本，这种负样本可能比较粗糙；它能提升轨迹—指令一致性，但不一定能判断好每一步是否忠实执行了某个子指令。

#### 7.3.4 主动感知与空中导航

##### Active Visual Information Gathering for Vision-Language Navigation
当 VLN 智能体对下一步导航不确定时，不应该盲目选择动作，而应该像人一样主动“探一探路”、收集更多视觉信息，再做导航决策。
更像是自己主动 explore 一段，获得更多环境认知信息。
论文摘要中明确指出，该框架要学习一个 exploration policy，决定 “when and where to explore”“what information is worth gathering”“how to adjust the navigation decision after exploration”。

##### Aerial Vision-and-Dialog Navigation
空中无人机的CVDN，多轮对话导航，它的核心是提出了一个新的任务设置、模拟器、数据集和基线模型。
1. 提出了AVDN数据集：一个连续、逼真的航拍导航模拟器。与 R2R 这类离散 viewpoint 图不同，AVDN 的无人机可以在连续空间中移动到任意位置；视觉观察是从 xView 高分辨率卫星图像中裁剪出的 top-down view (无人机俯视视角图像); 数据集包含3,064条航拍导航轨迹，并配有人类—人类异步对话：commander提供初始指令和后续回答，follower操控无人机并在不确定时提问。数据采集过程中还记录了 follower 在图像上的注意力区域
2. 两类任务：
- ANDH（Aerial Navigation from Dialog History）：给定当前轮之前的对话历史和当前视觉观察，预测当前对话轮对应的子轨迹动作。也就是说，它考察的是“在当前对话轮中，根据已有对话继续飞到下一阶段目标”。
- ANDH-Full（Aerial Navigation from Full Dialog History）：给定完整对话历史，让无人机从起点直接预测完整轨迹，飞到最终目标区域。这个任务更难，因为输入对话更长、目标描述更完整，同时需要完成更长距离的导航。

评价标准：最终预测view area与目标区域的IoU是否大于 0.4；评价指标包括 SR、SPL 和 Goal Progress

3. 提出 HAA-Transformer 基线模型
提出 Human Attention Aided Transformer（HAA-Transformer），不仅预测无人机下一步waypoint，还同时预测人类follower在图像中关注的位置。论文实验显示，加入 human attention prediction 后，Transformer 和 LSTM 两类模型性能都有提升，尤其对长轨迹更有帮助
````text
历史对话
   ↓
BERT Encoder
   ↓
Language Embeddings

无人机视觉观察
   ↓
xView-pretrained DarkNet-53 + Attention
   ↓
Visual Embeddings

无人机方向
   ↓
Direction Encoder
   ↓
Direction Embeddings
````
````text 
                三类 Embeddings 拼接
                        ↓
                Multi-layer Transformer
                        ↓
                Multimodal Embeddings

                ↙                 ↘
Navigation Decoder          Human Attention Decoder
        ↓                           ↓
Waypoint + Progress          Attention Mask
````
#### 7.3.5 视觉增强补充

##### Frequency-enhanced Data Augmentation for Vision-and-Language Navigation
提出一种频域视角下的数据增强方法，用于提升现有 VLN 模型的视觉—语言匹配能力。
FDA 认为 VLN 模型不仅要看懂图像中的整体布局，还要更好地利用图像中的高频细节，例如边缘、纹理、物体轮廓等；因此作者通过傅里叶变换在频域中混合图像高频成分，构造增强图像，让导航模型学会根据语言指令筛选真正有用的视觉细节。

关注图像的高频信息
why?：VLN 中真正帮助语言和视觉对齐的，不只是房间的大致布局，也包括物体边缘、纹理、局部细节等高频信息。
how?：
- 保留原图低频信息
- 混合原图和干扰图的高频信息
- 再反变换回图像
模型不能死记某些固定纹理，而必须根据语言指令去选择真正相关的高频线索。
该方法无需修改模型、无需增加参数，却能在 R2R、RxR、CVDN 和 REVERIE 等任务上提升 HAMT、DUET 等模型的泛化性能。

#### 7.3.6 Foundation Model、VLA 与机器人导航

##### NaVILA: Legged Robot Vision-Language-Action Model for Navigation
NaVILA 是一个面向腿式机器人的两层 VLA 导航框架：上层视觉语言模型根据历史图像、当前图像和语言指令生成中层动作，例如向前走 75cm 或右转 30°；下层视觉/激光雷达强化学习 locomotion policy 再把这些中层动作转化为真实机器人关节控制。

离散图导航和连续环境导航，模型输出往往都是中层动作指令，较难真正解决机器人的底层动作精确控制，故推进架构到：
Language + Vision → Mid-level action → Low-level joint control
包含上层 VLA 输出中层动作指令，下层 locomotion policy 则负责底层机器人关节的控制。

- 上层 VLA：建立在 VILA 视觉语言模型家族基础上
        VILA 的三个重要组成部分：
        1. Vision Encoder：编码视觉图像
        2. MLP Projector：视觉 token 投影到语言 token 空间
        3. LLM：处理和推理，自回归生成
NaVILA 的上层输出不是普通文本描述，而是固定动作词加连续参数：eg. move forward 75cm。
推理阶段用正则表达式解析器从 LLM 输出中提取动作类型和参数，例如 forward / turn left / turn right 以及对应距离或角度；作者称在模拟和真实实验中，所有动作都能成功匹配并映射。

- 下层 locomotion policy：论文以 Unitree Go2 为主要平台说明：机器人头部/基座上装有 LiDAR，点云频率 15Hz；Go2 有 18 个自由度，其中 6 个是基座自由度，策略控制四条腿的 12 个关节电机。
输入：
中层速度命令 + 本体感知 + LiDAR height map
输出：
12 个腿部关节目标位置
然后：
通过 PD/torque 控制驱动机器人运动。

- 新的 benchmark：VLN-CE-Isaac：在 Isaac Sim 中构建高保真的虚拟环境，引入机器人身体、关节运动和与环境的物理交互。作者使用 R2R 的相同场景，从 R2R Val-Unseen 的 1,839 条轨迹中选择 1,077 条可通行且 mesh 质量较好的轨迹。

##### Vision-and-language navigation today and tomorrow: A survey in the era of foundation models
利用 foundation models 建立 VLN 新架构。

1. World Model：如何理解、记忆和预测环境
- 历史记忆
- 拓扑地图
- 语义地图
- BEV表示
- 3D/volumetric 环境表示
- 未来视角预测；
- 环境生成与想象

2.  Human models: 智能体如何理解、生成、澄清和利用人类语言
- 导航指令生成
- 指令理解
- 地标发现
- 对话导航
- 提示生成
- LLM 辅助 instruction grounding
- 让智能体能向人类解释或请求帮助

3. VLN Agent: 具身导航智能体
- Traditional Models
Seq2Seq
↓
Speaker-Follower
↓
PREVALENT
↓
VLN-BERT
↓
HAMT
↓
DUET

- VLM / LLM based VLN models:
CLIP-Nav
NavGPT
MapGPT
InstructNav
NavGPT-2
NaVid
NaVILA
Generalist embodied navigation models

VLN 正在从“benchmark-specific model”变成“foundation-model-powered embodied agent”

4. Behavior Analysis：行为分析
如何评价、诊断和理解 VLN agent 的行为
- 模型到底依赖视觉还是语言？
- 是否真正使用地标？
- 是否会走捷径？
- 是否能泛化到 unseen environments？
- 是否能处理长指令？
- 是否能在真实机器人中部署？
- 评估指标是否足够？

##### NavGPT-2: Unleashing Navigational Reasoning Capability for Large Vision-Language Models
试图弥合 VLN-specialized models 和 LLM-based navigation paradigms 之间的性能差距，同时保持 LLM 生成可解释导航推理的能力
构建了一个可以训练的 VLM + 导航策略网络：
多视角图像 + 指令
        ↓
VLM 对齐到 LLM latent
        ↓
生成导航推理文本
        +
提取 LLM hidden states
        ↓
图结构导航策略网络
        ↓
选择下一步节点

总体结构：
NavGPT-2 由一个 Large Vision-Language Model 和一个 navigation policy network 组成；VLM 里用 Q-former 提取 image tokens，这些 image tokens 输入 LLM，用于生成导航推理；动作预测则使用经过 LLM 编码后的 image token 和 instruction token hidden representations。

- 视觉部分：EVA-CLIP ViT + Q-former
NavGPT-2 使用 EVA-CLIP ViT-g/14 作为冻结视觉编码器提取视觉特征，然后使用 Q-former 将每个 view 压缩成固定长度的 image tokens。
- 方向信息：显式输出给 LLM
NavGPT 会把候选方向的角度和方位写入 prompt，可以让 LLM 知道各个 candidate 对应的方向信息。
- LLM Latent 的更强表征力：
图像 token 和指令 token 经过 LLM 编码后，会得到 hidden representations，作者利用这些隐状态，训练 policy network，进行动作预测。
- 拓扑图结构的导航策略：解决长期空间记忆缺失的特点
论文指出，导航图会在线维护，作为记忆机制来追踪 navigation experience；NavGPT-2 可以从整个已构建拓扑图中选择下一步目标，从而支持长程历史记忆和回退。
节点表示经过 Transformer 编码，再与 LLM 编码后的 instruction 做 cross-modal encoding。随后使用 Graph-Aware Self-Attention（GASA）建模节点之间关系。GASA 会同时考虑节点间距离和视觉相似性；最后用两层前馈网络给每个节点打分，选择分数最高的节点作为目标，并沿图中最短路径走过去。

两阶段训练策略：
1. 训练 Q-former 生成导航推理能力：让 VLM 可以生成导航推理文本
从 R2R 训练集中随机选择 10K 个中间步骤，用 GPT-4V 根据当前观察和指令生成 single-step navigation reasoning。然后用这些数据对 Q-former 和 projection layer 做 visual instruction tuning，视觉编码器和 LLM 都保持冻结。论文里第一阶段初始化自 InstructBLIP，只微调 Q-former，frozen LLM 和 vision encoder 不动。
2. 训练导航策略网络：
第二阶段把预训练好的 VLM 接到 downstream navigation policy 上，冻结 VLM，只微调 policy network。训练使用 Behavior Cloning + DAgger loss。
BC 使用 ground-truth action，DAgger 使用 agent on-policy 采样产生的 partial graph，再根据到目标的 shortest path 生成 pseudo label。

实现了 VLM + LLM + policy net 三层次的架构设计，整体效果达到和 VLN-specialized model 接近的水平。

##### Hierarchical cross-modal agent for robotics vision-and-language navigation

Robo-VLN：VLN 从离散图导航提升到更接近真实机器人导航的连续控制环境。Robo-VLN 相比传统 VLN 具有更长轨迹、连续动作空间和障碍物挑战。
Hierarchical 的体现：高层推理模块和底层控制模块。

- 高层模块：
  1. 编码语言指令。
  2. 编码 RGB + Depth 视觉输入。
  3. Cross-modal Transformer 进行对齐。
  4. LSTM 模块作为短时记忆。
  5. 输出离散的高层子目标。

- 低层模块：
  1. 输入：

     ```text
     当前视觉特征
     +
     高层 sub-goal
     +
     低层历史状态
     ```

  2. 输出：

     ```text
     线速度
     角速度
     stop probability
     ```

HCM 是早期“高层语言视觉推理 + 低层连续控制”的分层 VLN 框架；NaVILA 则是在 foundation model 和腿式机器人时代对这种思想的进一步升级。
证明了分层结构比 flat baseline 更适合长时程连续 VLN，在 validation unseen 上显著提升 SR、SPL 和 NDTW。

##### Uni-NaVid: A Video-based Vision-Language-Action Model for Unifying Embodied Navigation Tasks

通用具身导航模型：同时做 VLN、Object Navigation、EQA、Human Following 等任务。
Uni-NaVid 是一个 video-based VLA model，用来统一多种 embodied navigation tasks，并通过统一输入输出配置把多类任务整合到一个模型中。

- 它集成了 Vision-and-Language Navigation、Object Navigation、Embodied Question Answering 和 Human-following 四类任务。
- 输入输出的统一：

```text
Uni-NaVid 的输入统一成：
        单目 RGB 视频流
                +
        自然语言指令

输出统一成：
        低层动作 token
                或
        自然语言答案
```

- 提出 Online Visual Token Merging：借鉴 Atkinson-Shiffrin 记忆模型，将视觉 token 按时间远近分组，当前帧保留更多细节，短期历史中等压缩，长期历史更强压缩。

##### Find What You Want: Learning Demand-conditioned Object Attribute Space for Demand-driven Navigation
更接近日常需求的导航任务：Demand-driven Navigation，DDN 需求驱动导航。
DDN 不让机器人找“指定物体”，而是让机器人理解“人的需求”，再推理哪些物体能满足这个需求，并导航到其中一个合适物体。

- LLM 的作用是建立需求到物体属性的映射。
- CLIP 负责做视觉图像和需求指令的对齐。
- Attribute module：6-layers Transformer Encoder

```text
        它的输入是 demand-object feature，由两部分拼接而成：
        Demand BERT feature
                +
        CLIP textual / visual object feature

        在文本属性学习阶段：
        需求文本 → BERT → demand feature
        物体文本 → CLIP Text Encoder → textual object feature
        拼接 → demand-object feature

        然后通过 Attribute Module 输出：
        demand-conditioned attribute feature
```

架构图：

```text
需求指令：I am thirsty
        ↓
BERT 编码需求

RGB 图像
        ↓
DETR 提取候选物体
        ↓
CLIP Image Encoder 编码候选物体

需求 feature + 候选物体 visual feature
        ↓
Attribute Module
        ↓
需求条件视觉属性特征

属性特征 + 全局视觉特征
        ↓
Transformer Policy
        ↓
MoveAhead / Rotate / Look / Done

如果 Done：
        ↓
Visual Grounding Model 输出目标 bounding box
```

## 8. VLN 重点模块总览和学习

关键模块：

- 语言编码器：RNN、BERT、LLM encoder
- 视觉编码器：CNN、ViT、CLIP、object detector、VLM
- 历史模块：RNN memory、trajectory Transformer、topological graph、video context
- 决策模块：action classifier、policy network、search/reranking、LLM planner
- grounding 模块：region matching、object detection、bounding box prediction

### 8.1 重点模块学习笔记

#### 8.1.1 BERT：Bi-directional Encoder Representation from Transformer

1. 什么是 BERT？

   BERT 个人认为就是把自然语言处理成语义向量的编码器。

2. BERT 解决了什么？

   BERT 完成了从一段话到每个 token 的语义表达向量的编码，可以实现用向量表示自然语言，同时其中的自注意力机制更加解决了一词多义的理解难题。

```text
Input Sentence
↓
Embedding
↓
Transformer Encoder Layer × N
↓
Output Features
```

3. BERT 具体如何实现？

整体上看：就是语言的 embedding 经过连续的 Transformer Encoder，得到对应的语义表达。

```text
Word Embedding
        ↓
Encoder 1
        ↓
Encoder 2
        ↓
Encoder 3
        ↓
...
        ↓
Encoder 12
```

每个 Encoder 中有以下结构：

```text
Multi-Head Attention
↓
Feed Forward Network
↓
Residual
↓
LayerNorm
```
其中最重要的是 Self-Attention 和 Bi-directional。
Self-Attention 到底在学习什么：学习的是一个词语与它上下文之间的语义联系，通过这个可以准确地修正这个词语的语义表达。
Bi-directional 的意义是什么：双向，利用上文同时也要看到下文，带来双向的理解。

#### 8.1.2 CLIP-ViT: Contrastive Language-Image Pre-training - Vision Transformer

1. 什么是 CLIP？

CLIP 就是让图像信息和语言语义映射到统一空间，通过对比学习，掌握二者对应关系的一种预训练方法。

2. CLIP 解决了什么？

视觉模型——看得懂不会说
语言模型——会说话不会看
CLIP 解决的是图像和语言之间的对应关系，以及二者如何放到同一个表示空间里。

3. CLIP 具体实现？

CLIP 其实是训练两个编码器：

```text
Image
↓
Image Encoder
↓
Image Feature

Text
↓
Text Encoder
↓
Text Feature
```

其中 Image Encoder 一般使用 ResNet 或者 ViT。
Text Encoder 一般使用类似 BERT 的一个 Transformer。

CLIP 的核心在于训练方法：
假设有四张图片 P1、P2、P3、P4，四个文字 T1、T2、T3、T4。
分别经过 Image Encoder 和 Text Encoder 得到：

图片向量 V1 V2 V3 V4； 语义向量 L1 L2 L3 L4
然后需要计算图片向量和语义向量的相似性 Similarity，目标是使得矩阵对角线最大。

|       | Dog  | Cat  | Car  | Chair |
| ----- | ---- | ---- | ---- | ----- |
| Dog   | 0.95 | 0.10 | 0.03 | 0.01  |
| Cat   | 0.12 | 0.92 | 0.08 | 0.03  |
| Car   | 0.01 | 0.04 | 0.97 | 0.06  |
| Chair | 0.02 | 0.03 | 0.05 | 0.94  |
