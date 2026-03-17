---
title: "FL-Learning-2025 Paper Summary"
description: "FL paper summary I have read in 2025: Advanced Paradigms in Decentralized Intelligence."
createdAt: 2026-03-17
category: "technology"
tags:
  - Research
---

# FL-Learning-2025
FL paper summary I have read in 2025

[Advanced-Paradigms-in-Decentralized-Intelligence-An-Exhaustive-Analysis-of-Feder.md](https://github.com/user-attachments/files/25758742/Advanced-Paradigms-in-Decentralized-Intelligence-An-Exhaustive-Analysis-of-Feder.md)
# Advanced Paradigms in Decentralized Intelligence: An Exhaustive Analysis of Federated Learning Optimization, Heterogeneity Management, and Systemic Robustness

## 1. Executive Summary
从集中式数据处理向去中心化、隐私保护的机器学习演进，催生了联邦学习（Federated Learning, FL）研究的爆发。作为一种基于本地数据孤岛进行全局模型更新且无需暴露原始信息的机制，FL 正在快速演进以应对深层的系统性瓶颈。当前分布式智能的格局受到三个关键维度的交织影响：网络带宽与边缘计算硬件的物理局限；非独立同分布（non-IID）数据架构的统计现实；以及在对抗性攻击与硬件故障下维持系统鲁棒性的迫切需求。
本报告对该领域的近期文献进行了详尽分析，细致解构了旨在解决这些多层面瓶颈的理论框架、数学模型以及架构创新。通过系统性提取大量近期研究背后的动机、面临的技术挑战、应用方法与核心创新点，本文构建了当前前沿技术的精细分类体系。

## 2. 领域文献分类体系
为建立结构化的分析框架，根据文献的核心优化目标，我们将近期研究划分为三个细化的子类：

1. **通信效能与架构优化 (Communication Efficacy and Architectural Optimization)**：涵盖旨在最小化传输梯度负载、减少边缘计算开销以及重构模型同步流程（如量化、稀疏化、剪枝和拆分学习拓扑）的研究。
2. **统计异质性与自适应个性化 (Statistical Heterogeneity and Adaptive Personalization)**：重点剖析旨在调和 non-IID 数据分布导致的发散问题的机制，包括类别不平衡缓解、动态加权客户端选择以及新兴的联邦基础模型（LLMs）微调。
3. **系统鲁棒性、隐私保护与控制 (Systemic Robustness, Privacy, and Control)**：详细阐述用于保护联邦网络免受拜占庭攻击、意外数据泄露、不可靠设备掉线影响的协议，以及联邦协议在实时反馈控制系统中的数学整合。

---

## 3. Category I: 通信效能与架构优化
去中心化训练的根本瓶颈在于带宽受限的网络中传输大规模参数模型的高昂成本。随着模型从百万参数级扩展至百亿参数级，简单的全量同步在数学与物理上均不可行。文献表明，当前的重点已大规模转向高级有损压缩技术（特别是量化与剪枝），并辅以误差补偿协议，以防止收敛界限的劣化。

### 3.1 梯度压缩与量化优化
量化技术降低了权重更新浮点表示的精度。然而，多篇文献指出的核心挑战在于量化误差的累积会严重扭曲全局目标函数。TLAQC (Two-layer accumulated quantized compression) 理论框架对此进行了明确应对 。TLAQC 通过累积量化误差和保留的权重增量（Weight Deltas）来缓解精度损失，这本质上充当了一个内存缓冲区，将丢弃的梯度信息注入到随后的训练轮次中。经验评估表明，这能显著降低平均量化误差，同时保持高水平的通信压缩 。
同样，网络控制系统的物理限制要求实现高精度的量化反馈控制。分析表明，虽然理论上无法在实践中实现无约束的最优量化，但在高分辨率量化环境中，允许使用独立处理输入样本的无记忆量化器 (Memoryless Quantizers) 。将这一理念扩展至聚合优化 (Aggregative Optimization)，利用 Nesterov 动量系数的压缩梯度追踪算法被证明，即使在深度压缩的通信下也能产生有利的线性收敛率 。
在动态量化领域，静态量化范围无法捕获梯度不断演变的方差。诸如 **DAdaQuant** 这样的双重自适应量化技术，针对早期轮次中因高梯度方差导致的溢出或严重截断问题，通过每轮动态调整量化位宽和截断阈值来实现通信效率的最大化。更进一步，**FracBits** 指出硬件仅原生支持离散整数精度（如 INT4, INT8），因此通过在相邻整数精度间进行随机切换，模拟了更细粒度的小数位宽量化，以完美匹配梯度方差。
对于不同的训练阶段，**FedDQ (Descending Quantization)** 观察到梯度范数随着训练自然下降，后期高精度训练收益递减，因此提出与全局梯度范数成比例单调递减量化级别的策略。针对网络层间的统计差异，**FedLAQ** 提出了逐层分桶 (Layer-wise Bucketing) 策略，将参数分组并基于桶内方差进行量化；而 **FedFQ** 进一步细化，通过将层划分为微块并为每个子矩阵分配独立的缩放因子，解决了层内异常权重导致的失真问题。在混合精度和硬件感知层面，**FedHQ** 则将算法量化与本地 NPU 的异构硬件精度约束对齐，在运行时动态编译混合精度格式。

### 3.2 块级剪枝、稀疏化与网络架构解耦
除了降低位宽，通过剪枝完全移除冗余参数提供了平行的优化路径。然而，主要挑战在于传统剪枝往往忽略了输入数据的特征敏感性，从而降低了模型效用。高级框架采用了**敏感度感知和块级剪枝 (Sensitivity-aware and Block-wise Pruning)**。该方法通过计算相对于输入的梯度敏感度并结合特征重组，减少了通道冗余，同时保留了差分隐私所需的数学结构 。
针对深度神经网络功能冗余的问题，**FEDLWS (Federated Learning with Adaptive Layer-wise Weight Shrinking)** 避免了硬剪枝过早破坏特征路径，而是基于层敏感度动态应用 L1/L2 正则化迫使权重归零。同时，在跨客户端通信中，**FetchSGD** 采用 Count-Sketch 矩阵投影技术，将庞大的梯度向量投影到低维草图空间，由服务器提取重要信息，解决了低带宽链路中维护误差累积缓冲区的高内存开销问题。
架构的去中心化甚至已扩展到物理网络基础设施本身。**基于投票的共识模型压缩 (Voting-Based Consensus Model Compression)** 实现了网络内联邦学习 (In-Network FL)，其中聚合直接发生在可编程网络交换机上，而不是中心化的参数服务器上 。由于交换机硬件有严重的内存限制（通常只能处理整数聚合），该系统采用两阶段协议：客户端首先上传 0-1 数组以投票选出最重要的模型更新，随后仅上传达成共识的稀疏更新，从而绕过内存瓶颈实现快速网络内聚合 。
在极端资源受限的异构边缘网络中，**Heroes** 框架指出，压缩或剪枝参数如果不充分优化会降低训练性能 。它引入了神经组合 (Neural Composition) 技术，利用低秩张量构建尺寸可调的模型，并结合自适应局部更新频率，以适应客户端计算能力和资源预算的差异 。类似地，**BOSE** 通过块级模型拆分和传输，解决 DNN 超出单个 IoT 设备内存容量的问题；而 **C2-SFL** 则针对移动边缘计算 (MEC) 制定了考虑成本和类别不平衡的拆分点选择联合优化策略。

### 3.3 文献提取：通信效能与架构优化

| Paper Title | 中文论文名 | Keywords | Motivation | Challenge | Method | Innovation |
| --- | --- | --- | --- | --- | --- | --- |
| A Sensitivity-aware and Block-wise Pruning Method... | 隐私保护联邦学习的敏感度感知与分块剪枝方法 | Pruning, Privacy, Sparsity, Feature Selection | 本地模型通信开销大且存在冗余参数。 | 传统剪枝忽略了特征选择对模型效用的影响。 | 敏感度感知块级剪枝 (Sensitivity-aware block-wise pruning)。 | 定义了相对于输入的梯度敏感度，动态优化子网络选择。 |
| A Tutorial on Quantized Feedback Control | 量化反馈控制教程 | Quantization, Control, Networked Systems | 数字网络日益普及需要高效的数据传输。 | 带宽限制阻碍了连续的高精度反馈信号传输。 | 静态/动态量化；无记忆量化 (Memoryless quantization)。 | 系统性地映射了对数级量化器在 LTI 系统中的稳定性边界。 |
| Accelerated Distributed Aggregative Optimization | 加速分布式聚合优化 | Aggregative Optimization, Gradient Tracking | 节点协作需要高效地最小化整体网络成本。 | 大规模多智能体协调中的高通信负载。 | 结合 Nesterov 动量的压缩梯度追踪。 | 在强凸目标函数和压缩通信下实现线性收敛率。 |
| BiPruneFL | 具备二值量化与剪枝的计算与通信高效联邦学习 | Binary Quantization, Pruning | 计算和带宽需要同时大幅减少。 | 极端量化与剪枝结合导致的级联精度损失。 | 联合二值量化与稀疏剪枝。 | 在聚合前本地联合优化剪枝掩码与二值缩放因子。 |
| Blade | 突破异步联邦学习的性能极限 | Asynchronous FL, Stragglers | 异步训练常导致过时梯度和模型发散。 | 平衡硬件利用率与统计效率。 | 感知陈旧度的异步聚合。 | 基于持续评估的陈旧度指数，动态调整全局更新步调。 |
| BOSE | 异构边缘计算中的分块联邦学习 | Block-Wise FL, Heterogeneous Edge | DNN 超过单个 IoT 设备的内存容量。 | 拆分模型引入显著延迟和特征退化。 | 块级模型拆分与传输。 | 独立训练神经网络块并实现局部误差传播。 |
| C2-SFL | 面向移动边缘计算的类别均衡且成本感知的拆分联邦学习 | Split FL, MEC, Cost-Aware | 移动边缘计算面临严格能耗与延迟约束。 | 在异构节点拆分模型时平衡数据分布。 | 类别均衡且考虑成本的拆分学习。 | 为拆分点选择与类别不平衡建立联合优化问题。 |
| DAdaQuant | 面向通信高效联邦学习的双重自适应量化 | Doubly-adaptive Quantization | 静态量化范围无法捕捉梯度的演变方差。 | 早期轮次的高梯度方差导致溢出或严重截断。 | 双重自适应量化。 | 每轮动态调整量化位宽与截断阈值。 |
| EF21 | 一种全新、更简、理论更优且实际更快的误差反馈 | Error Feedback, Gradient Compression | 标准误差反馈证明复杂且实际边界次优。 | 现有误差反馈未能严格修复 SignSGD 的发散。 | 简化的误差反馈 (EF21)。 | 提供更紧凑收敛率的马尔可夫误差反馈。 |
| Expediting In-Network FL... | 通过基于投票的共识模型压缩加速网内联邦学习 | In-Network Computing, Consensus | 在大规模 FL 中，参数服务器成为瓶颈。 | 网络交换机内存受限，仅支持简单整数聚合。 | 基于投票的共识 (FediAC)。 | 两阶段协议：先使用 0-1 数组进行索引投票，再传输稀疏更新。 |
| Heroes | 异构边缘网络中结合神经组合与自适应局部更新的轻量级联邦学习 | Neural Composition, Adaptive Update | 资源限制导致压缩参数优化不足。 | 同一模型结构无法适应高度异构的边缘设备。 | 增强的神经组合与自适应局部更新。 | 利用低秩张量构建尺寸可调模型，并动态调整局部更新频率。 |
| Two-layer accumulated quantized compression (TLAQC) | 用于通信高效联邦学习的双层累积量化压缩 (TLAQC) | Quantization Error, Two-layer Accumulation | 带宽限制阻碍 DNN 在 FL 中的扩展。 | 极端量化导致高精度损失及无效的零值映射。 | TLAQC；最小因子变换。 | 累积量化误差和保留的权重增量，极大降低精度损失。 |
| Communication Efficient FL with Adaptive Quantization | 基于自适应量化的通信高效联邦学习 | Adaptive Quantization, Bandwidth | 静态量化在稳定收敛阶段浪费位宽。 | 精确预测每轮所需的最小精度。 | 依赖 Epoch 的自适应量化。 | 随模型接近局部最优，动态缩小量化位宽。 |
| DoubleSqueeze | 具有双向误差补偿压缩的并行随机梯度下降 | Parallel SGD, Error-Compensated | 上传和下载的双向通信开销巨大。 | 压缩服务器到客户端的广播破坏了全局同步。 | 双向误差补偿压缩。 | 在客户端上传和服务器下载两侧均应用误差反馈。 |
| FedComLoc | 稀疏量化模型的通信高效分布式训练 | Distributed Training, Sparse Models | 对特定本地任务，密集全局模型是不必要的。 | 在稀疏化本地通信的同时维持全局泛化。 | 稀疏和量化的分布式训练。 | 将空间定位参数直接整合到稀疏化掩码中。 |
| FedDQ | 基于递减量化的通信高效联邦学习 | Descending Quantization | 梯度范数随训练进程自然下降。 | 在训练后期保持高精度收益递减。 | 递减量化。 | 随全局梯度范数下降，单调减少量化级别。 |
| FedFQ | 具备细粒度量化的联邦学习 | Fine-Grained Quantization | 逐层量化受制于层内梯度方差差异。 | 每层单一的缩放因子导致异常权重失真严重。 | 细粒度块量化。 | 将层划分为微块，为每个子矩阵分配独立缩放因子。 |
| FedHQ | 面向联邦学习的混合运行时量化 | Hybrid Runtime Quantization | 硬件加速器具有高度特定的量化要求。 | 将算法量化与异构硬件精度约束对齐。 | 混合运行时量化。 | 将本地模型动态编译为本地 NPU 支持的混合精度格式。 |
| FedLAQ | 面向通信高效联邦学习的逐层分桶自适应量化 | Layer-wise Bucketing | 浅层与深层之间梯度分布差异巨大。 | 跨不同统计特征的层应用统一压缩掩码。 | 逐层分桶 (Layer-wise bucketing)。 | 按层将参数分组，基于桶内方差进行量化。 |
| FedLPS | 局部参数共享的多任务异质联合学习 | Local Parameter Sharing, Multi-Task | 独立任务训练浪费共享的特征提取能力。 | 在相关但不同的任务间无延迟协调参数共享。 | 局部参数共享 (Local Parameter Sharing)。 | 将模型分为共享特征提取器和任务特定保留的头部。 |
| FEDLWS | 采用自适应逐层权重收缩的联邦学习 | Layer-wise Weight Shrinking | 密集神经网络含有高度功能冗余。 | 硬剪枝过早地彻底破坏特定特征路径。 | 自适应逐层权重收缩。 | 基于层敏感度动态应用 L1/L2 正则化迫使权重归零。 |
| FedMPQ | 基于多共享码本乘积量化的安全且通信高效联邦学习 | Multi-codebook Quantization, Security | 传统量化暴露了梯度分布，易受推理攻击。 | 在不泄漏本地数据集统计特性的前提下压缩。 | 多码本乘积量化 (Multi-codebook product quantization)。 | 使用随机正交码本编码梯度，掩盖统计特征。 |
| FedZipper | 针对统计异质性联邦学习的逐层量化压缩框架 | Statistical Heterogeneity, Layer-wise | non-IID 数据导致不同层以不同速率发散。 | 统一层压缩破坏了客户端特定的发散路径。 | 逐层量化压缩框架。 | 基于统计异质性指标压缩参数，保留个性化路径。 |
| FetchSGD | 利用草图化的通信高效联邦学习 | Sketching, Count-Sketch | 在低带宽链路上追踪海量梯度向量效率低。 | 维护误差累积缓冲区需要高内存占用。 | Count-Sketch 矩阵投影。 | 将梯度投影到低维草图空间，由服务器提取高频特征。 |
| FracBits | 通过分数位宽实现的混合精度量化 | Mixed Precision, Fractional Bit-Widths | 整数位宽缺乏完美匹配梯度方差的粒度。 | 硬件原生仅支持离散整数精度。 | 分数位宽量化 (Fractional bit-width quantization)。 | 通过在相邻整数精度间随机切换模拟分数位宽。 |

---

## 4. Category II: 统计异质性与自适应个性化
独立同分布 (IID) 的假设在真实的边缘部署中是不存在的。设备收集的数据受用户行为、地理位置和时间变量的严重偏差影响。这种 non-IID 特性导致客户端模型向不同局部最优发散，产生的权重背离严重降低了聚合全局模型的性能。

### 4.1 缓解类别不平衡与客户端漂移 (Client Drift)
权重背离的一个主要驱动因素是类别不平衡。当全局数据集本质上不平衡时，随机选择客户端通常会构建高度倾斜的小批量进行聚合，从而加剧该问题。**Fed-CBS** 框架通过设计异构感知的采样机制来应对 。它制定了一个类别不平衡度量标准，利用同态加密以保护隐私的方式安全推导该度量。通过计算高效的采样，Fed-CBS 刻意选择能共同生成完全类别均衡数据集的客户端子集，其准确性优于完全随机采样 。
或者，可以通过局部干预在传输前防止漂移。**FLY-SMOTE** 框架通过在边缘直接注入合成数据来处理 non-IID 特征 。客户端在本地计算少数类与多数类的比例，如果超出不平衡阈值，本地节点将应用修改后的合成少数类过采样技术 (SMOTE)，基于最近邻生成合成样本来在本地重新平衡训练分布 。这确保了发送给服务器的更新梯度代表了一个经过统计归一化的局部流形。此外，**Addressing Class Imbalance in Federated Learning** 通过监控梯度方向来推断全局类别分布，动态缩放损失函数进行聚合层面的不平衡感知调整；而 **Distribution-Regularized FL** 则利用 Wasserstein 距离作为全局与局部 Logit 间的正则化项，在不减缓收敛的前提下将局部模型拉回全局流形。

### 4.2 客户端选择机制与异构聚类
静态或随机客户端选择会导致选择了没有有价值梯度更新或数据冗余的设备，浪费通信轮次。动态贡献评估模型 **AdaFL** 通过从小型选择池开始并逐渐扩大规模优化了这一点 。它使用加权线性函数计算动态贡献分数，融合当前性能指标与历史轮次数据，并通过权重衰减参数调整历史数据的影响 。这在数学上保证了通信带宽被严格分配给提供最大梯度新颖性的设备。
此外，由于强制将单一全局模型应用于互斥的数据任务是破坏性的，聚类联邦学习成为必然。**FedGroup** 无需访问本地数据，仅通过计算上传参数更新的余弦相似度即可将客户端聚类。而 **FedAWA** 则将评估客户端相似度的高昂矩阵比较开销，转化为将客户端模型差异投影到低维向量，从而快速计算自适应聚合权重。在移动边缘计算 (MEC) 场景下，**Client Selection for Wireless FL With Data and Latency Heterogeneity** 制定了一个混合整数非线性规划问题，联合优化信道状态与数据方差，解决网络物理层延迟与数据质量的权衡。

### 4.3 联邦基础模型 (Foundation Models) 与推测解码
将大型语言模型 (LLMs) 集成到联邦架构中面临着由于其庞大内存占用（如 KV 缓存限制）和高词元生成延迟带来的前所未有的挑战。**Hat-Shaped Device-Cloud Collaborative Inference (HAT)** 框架重构了推理流水线以保护提示词隐私并消除延迟 。传统的 U 型推理强制要求隐藏状态的双向传输。HAT 通过“提示词分块 (Prompt Chunking)”打破了这一限制，允许设备传输分块状态，同时云端计算前一个相邻状态 。此外，它利用并行推测解码 (Speculative Decoding)，边缘设备在等待服务器验证的同时在本地草拟可能的词元序列，从而将首次词元延迟 (TTFT) 缩减了 50% 以上 。
在微调领域，**FedICU** (Splitting with Importance-aware Updating) 重新定义了 non-IID 约束下的参数高效微调 (PEFT) 。在不同任务间不加区分地聚合低秩适配器 (LoRA) 参数会导致基础模型核心能力的灾难性遗忘 。FedICU 明确地将客户端更新分解为共识 (Consensus) 和发散 (Divergence) 组件，并根据它们对全局目标的贡献动态平衡。它实施了重要性感知参数更新掩码，确保仅上传和集成具有统计显著性、特定领域的权重偏差 。类似地，双层个性化方法 (**FedBip**) 利用任务向量聚合 (Task-vector Aggregation)，基于任务向量相似性逐层聚合客户端级微调，在数学上隔离了利益冲突客户端造成的干扰 。针对边缘设备的计算限制，**FedTuneFM** 基于本地输入的注意力映射方差动态改变适配器矩阵的秩，实现自适应压缩；而 **Collaborative Learning of On-Device Small Model and Cloud-Based Large Model** 采用双向知识蒸馏，在云端大模型与边缘极小模型之间实现特征与软标签的知识同步。

### 4.4 文献提取：统计异质性与自适应个性化

| Paper Title | 中文论文名 | Keywords | Motivation | Challenge | Method | Innovation |
| --- | --- | --- | --- | --- | --- | --- |
| A Novel Hat-Shaped Device-Cloud Collaborative Inference Framework... | 一种新型的大语言模型帽型端云协同推理框架 | LLMs, Device-Cloud, Speculative Decoding | U 型推理虽保护隐私但通信开销大。 | 长 Prompt 产生海量隐藏状态，导致极端延迟。 | HAT 框架；提示词分块；并行草拟。 | 最大化设备传输与云端计算的重叠；边缘进行推测性词元草拟。 |
| AdaFL: Adaptive Client Selection... | 面向高效联邦学习的自适应客户端选择与动态贡献评估 | Client Selection, Contribution Evaluation | 随机选择浪费带宽与计算资源。 | 仅基于不稳定的过去表现评估客户端未来效用。 | 基于奖励的收益策略；加权线性函数。 | 融合历史和当前性能指标，使用权重衰减动态调整选择池。 |
| Bi-level Personalization for Federated Foundation Models... | 联邦基础模型的双层个性化：一种任务向量聚合方法 | Foundation Models, Task-vector Aggregation | 标准微调因局部偏差破坏基础预训练知识。 | 在深度任务个性化与维持基础能力间取得平衡。 | 双层个性化 (FedBip)；任务向量。 | 计算逐层任务向量，在服务器端聚合时过滤掉冲突知识。 |
| Fed-CBS | 一种通过减少类别不平衡实现联邦学习的异构感知客户端采样机制 | Class-Imbalance, Client Sampling | 随机选择构建高度倾斜的全局聚合批次。 | 在不暴露本地标签分布的情况下衡量全局类别不平衡。 | 联邦类别均衡采样；同态加密。 | 安全计算标签偏斜并刻意采样特定客户端组合以产生均衡批次。 |
| FLY-SMOTE | 在联邦学习系统中重新平衡非独立同分布的物联网边缘设备数据 | Non-IID Data, SMOTE, Edge Computing | 边缘数据集不平衡导致局部最优发散。 | 严格不共享原始数据的前提下纠正数据分布。 | FLY-SMOTE；本地合成生成。 | 本地计算少数/多数类比例，在边缘动态生成合成样本。 |
| Splitting with Importance-aware Updating... (FedICU) | 大语言模型异构联邦学习的重要性感知更新拆分 | LLMs, LoRA, Catastrophic Forgetting | 不加区分地聚合 LoRA 导致对特定分布过拟合。 | 上传全量 LoRA 矩阵覆盖了通用语言推理能力。 | FedICU；共识/发散拆分。 | 应用重要性感知掩码的稀疏更新，过滤掉不显著的 LoRA 参数。 |
| Adaptive Personalized FL for Non-IID Data... | 针对具有持续分布偏移的非独立同分布数据的自适应个性化联邦学习 | Continual Distribution Shift | 客户端数据分布随时间演变而非静态。 | 随着本地数据流形漂移，个性化模型变得过时。 | 持续个性化 FL。 | 整合时间正则化项，衰减旧特征同时吸收新局部数据偏移。 |
| Addressing Class Imbalance in Federated Learning | 解决联邦学习中的类别不平衡问题 | Class Imbalance, Loss Reweighting | 全局模型偏向聚合数据集中的多数类。 | 服务器无法查看全局分布来应用标准重加权。 | 不平衡感知聚合。 | 通过监控梯度方向推断全局构成，并动态缩放损失函数。 |
| Capture Global Feature Statistics for One-Shot FL | 为单次通信联邦学习捕获全局特征统计信息 | One-Shot FL, Feature Statistics | 高度断开连接的边缘环境无法进行多轮通信。 | 在单一通信轮次内实现收敛。 | 全局特征统计聚合。 | 客户端上传其特征空间方差的参数化表示，实现服务器端合成。 |
| Client Selection for Wireless FL... | 考虑数据和延迟异质性的无线联邦学习客户端选择 | Wireless Networks, Latency | 掉队者决定了同步 FL 的整体训练时间。 | 联合优化最具信息量的数据与最快的物理连接。 | 延迟-数据联合优化。 | 求解混合整数非线性规划，基于信道状态和数据方差选择客户端。 |
| Collaborative Learning of On-Device Small Model... | 端侧小模型与云端大模型的协同学习：进展与未来方向 | Knowledge Distillation, Asymmetric Models | 边缘设备内存无法容纳大型全局模型。 | 在架构完全不同的神经网络间同步知识。 | 双向知识蒸馏。 | 云模型向下传输软标签，边缘模型向上传输硬特征表示。 |
| Distribution-Regularized FL on Non-IID Data | 非独立同分布数据上分布正则化的联邦学习 | Distribution Regularization | 局部交叉熵损失导致倾斜数据上的极端发散。 | 在不减缓收敛的情况下将局部模型拉回全局模型。 | 分布正则化。 | 应用 Wasserstein 距离作为全局和局部 Logit 间的正则化器。 |
| Fast-Convergent FL with Adaptive Weighting | 采用自适应加权的快速收敛联邦学习 | Adaptive Aggregation, Convergence | 标准 FedAvg 仅凭本地样本量加权。 | 大数据量但低质量数据的客户端主导了模型。 | 基于损失地形的自适应加权。 | 基于局部模型损失地形平滑度的倒数动态重加权。 |
| FedAWA | 基于客户端向量的联邦学习聚合权重自适应优化 | Client Vectors, Aggregation Weights | 评估客户端相似度需要比较庞大的参数矩阵。 | 服务器端分析数百万权重的计算成本过高。 | 使用客户端向量的自适应加权。 | 将客户端模型差异投影到低维向量以快速计算聚合系数。 |
| Federated Learning with Domain Shift Eraser | 带有域偏移消除器的联邦学习 | Domain Generalization | 数据源自不同的物理硬件（如不同 MRI 扫描仪）。 | 特征在物理上不同，尽管代表相同的底层对象。 | 域偏移消除器 (Domain Shift Eraser)。 | 在本地网络嵌入域对抗特征提取器，剥离硬件特定特征。 |
| FedGCS | 通过基于梯度的优化实现联邦学习高效客户端选择的生成式框架 | Generative Client Selection | 客户端选择是高约束的组合优化问题。 | 随着网络规模扩大，搜索最优子集是 NP-Hard。 | 生成式客户端选择。 | 训练轻量级生成模型，输出客户端空间上的概率分布。 |
| FedGroup | 基于分解数据驱动度量的高效聚类联邦学习 | Clustered FL, Data-Driven | 强加单一模型于互斥数据任务具破坏性。 | 在不访问本地数据的情况下识别客户端集群。 | 分解数据驱动度量。 | 基于上传参数更新的余弦相似度在聚合前对客户端聚类。 |
| FedTuneFM | 面向移动边缘计算的基础模型联邦微调：通过自适应压缩和注意力... | Foundation Models, Adaptive Compression | 边缘微调基础模型耗尽电池和内存。 | 对具有不同注意力敏感度的层应用统一 LoRA 秩。 | 自适应压缩与注意力。 | 基于本地输入的注意力映射方差动态改变适配器矩阵的秩。 |
| FedsId | 用于医学图像分类的共享标签分布联邦学习 | Medical Image Classification | 医疗图像标签主观性强，不同机构差异大。 | 硬标签迫使网络对特定标注者偏差过拟合。 | 共享标签分布。 | 通过共享全局标签分布先验软化真值标签，平滑标注差异。 |
| GPAFed | 非独立同分布数据下联邦学习的梯度投影引导自适应聚合策略 | Gradient Projection-Guided Aggregation | 来自 non-IID 客户端的冲突梯度在聚合时相互抵消。 | 平均正交梯度时全局模型丢失信息。 | 梯度投影引导策略。 | 将冲突的局部梯度投影到共享正交平面以保留方向幅度。 |

---

## 5. Category III: 系统鲁棒性、隐私保护与控制
联邦环境本质上是不可信的。数据与计算的解耦引入了数据投毒、恶意干扰和意外记忆的向量。此外，物理网络的现实意味着设备经常掉线，这需要鲁棒的框架在深度的系统波动下维持分析完整性。

### 5.1 设备可靠性评估与陈旧梯度整合
“掉队者 (Stragglers)”——断开连接或遭遇严重计算延迟的设备——会瘫痪同步 FL 协议。丢弃这些延迟梯度的代价是浪费大量算力，但盲目整合又会注入“陈旧”信息，破坏当前全局状态。**FLUDE (Federated Learning Framework for Undependable Devices at Scale)** 框架对此进行了全面建模 。FLUDE 严格根据历史行为评估设备可靠性，而不引入额外的探测开销，将可靠性定义为成功完成训练的连续概率分布 。通过结合感知陈旧度 (Staleness-aware) 的策略，服务器明智地仅将全局模型分发给优化的子集，并根据实时网络波动动态调整纳入的频率阈值 。针对纯异步环境，**FedASMU (Efficient Asynchronous FL with Dynamic Staleness-Aware Model Update)** 则彻底抛弃同步屏障，在任何设备更新到达时立即调整学习率衰减以抵消其陈旧度。

### 5.2 隐私保护、机器遗忘与拜占庭共识
数据隐私的内涵远不止于本地留存。高级梯度反演攻击可以仅从拦截的参数更新中重建原始图像和文本。诸如 **SecEmb** 等框架展示了稀疏感知安全聚合 (Sparsity-aware secure aggregation)，在这种无损的安全推荐系统中，用户仅下载相关的嵌入向量，确保服务器对项目索引的信息一无所知 。同样，**Efficient Sparse Secure Aggregation** 利用专门为稀疏矩阵构建的同态加密，跳过了零值的加密计算，大幅降低了传统 SecAgg 的密码学开销。在物理层面上，**Private FL with Dynamic Power Control** 甚至利用物理传输功率控制，将无线信道本身的噪声转化为天然的差分隐私掩码。
当对抗节点故意注入错误数据（FDI 攻击）以扭曲全局模型时，标准平均法会遭遇灾难性失败。在物理传感器网络（如雷达系统）中，**基于共识的分布式状态估计 (DSE)** 利用平均共识协议过滤异常 。通过制定混合共识方法，节点在状态更新步骤之前以数学方式补偿 FDI 攻击引起的估计误差，根据最大节点度阈值隔离异常向量 。此外，图理论也被广泛应用，**Consensus and Cooperation in Networked Multi-Agent Systems** 利用拉普拉斯矩阵保证了去中心化智能体在无领导状态下的渐进状态共识。
此外，“被遗忘权”的监管要求促生了“联邦机器遗忘 (Federated Unlearning)”。由于无法集中隔离和删除原始数据，网络必须以数学方式提取特定客户端在历史上对全局模型施加的精确梯度影响。这通常被建模为逆海森矩阵近似 (Inverse Hessian approximation)。然而，计算海森矩阵开销巨大，**Unlearning during Learning** 提出通过在全局训练前向传递期间持续更新影响矩阵，实现近似瞬时的影响抵消，从而使模型在持续运行中“边学边忘”。针对生成模型（如 GAN 和 Diffusion Models）容易精确记忆和复述训练样本的问题，**PRISM** 采用了改进的随机掩码，应用根据生成器损失函数敏感度动态缩放的校准高斯噪声，在保护差分隐私边界的同时维持生成图像的保真度。

### 5.3 文献提取：系统鲁棒性、隐私保护与控制

| Paper Title | 中文论文名 | Keywords | Motivation | Challenge | Method | Innovation |
| --- | --- | --- | --- | --- | --- | --- |
| A Robust Federated Learning Framework for Undependable Devices... (FLUDE) | 针对大规模不可靠设备的鲁棒联邦学习框架 | Undependable Devices, Staleness, Scale | 边缘设备在长训练轮次中经常断网。 | 资源浪费在断线节点并整合高度陈旧的破坏性梯度。 | FLUDE；历史行为评估。 | 基于历史行为定义数学可靠性分数，选择性分发模型。 |
| Consensus-Based Algorithms for Distributed Network-State... | 用于分布式网络状态估计与定位的共识算法 | FDI Attacks, Target Tracking, DSE | 对抗节点可注入错误数据扭曲雷达跟踪网络。 | 在完全去中心化、时变网络中识别恶意状态。 | 混合共识方法；DSE。 | 利用由节点度限制的平均共识协议补偿 FDI 误差。 |
| SecEmb: Sparsity-Aware Secure Federated Learning... | 具大规模嵌入端侧推荐系统的稀疏度感知安全联邦学习 | Recommender Systems, Secure Aggregation | 推荐系统即使稀疏化也会暴露项目索引。 | 在保持稀疏性的同时屏蔽用户嵌入的精确索引。 | 无损安全嵌入检索。 | 客户端在中心服务器完全不透明的情况下下载/上传嵌入。 |
| A Survey on Federated Unlearning... | 联邦机器遗忘综述：挑战、方法与未来方向 | Machine Unlearning, Right to be Forgotten | 法律框架（GDPR）要求根据请求删除用户数据。 | 在不重新训练的情况下删除深度神经网络中的影响。 | 梯度上升；影响函数。 | 以数学方式分离并从全局状态减去特定客户端的历史梯度轨迹。 |
| PRISM: Privacy-Preserving Improved Stochastic Masking... | 联邦生成模型的隐私保护改进随机掩码 | Generative Models, Stochastic Masking | GAN 和 Diffusion 模型精确记忆并复述训练样本。 | 在保证差分隐私的同时保留生成数据的保真度。 | 改进的随机掩码。 | 应用根据生成器损失敏感度动态缩放的校准高斯噪声。 |
| Private FL with Dynamic Power Control... | 通过非相干空中计算实现动态功率控制的隐私联邦学习 | Over-the-Air Computation, Power Control | 无线传输通过信道窃听泄漏梯度。 | 无需复杂的密码学开销保护物理层传输。 | 非相干空中计算 (Over-the-Air Computation)。 | 动态控制物理传输功率，利用信道噪声作为天然隐私掩码。 |
| Efficient Sparse Secure Aggregation... | 面向联邦学习的高效稀疏安全聚合 | Secure Aggregation, Cryptography | 密码学安全聚合 (SecAgg) 需要巨大的计算开销。 | 当模型高度稀疏且动态时 SecAgg 协议失效。 | 稀疏安全聚合。 | 利用专为稀疏矩阵构建的同态加密，跳过零值加密。 |
| Fed-CAD | 结合相关性感知自适应局部差分隐私的联邦学习 | Local Differential Privacy, Correlation | 标准局部差分隐私 (LDP) 注入过多噪声破坏精度。 | 平衡强大的数学隐私保证与模型收敛效用。 | 相关性感知的自适应 LDP。 | 识别特征相关性，战略性地向高度相关的权重注入低方差噪声。 |
| Unlearning during Learning... | 边学边忘：一种高效的联邦机器遗忘方法 | Efficient Unlearning, Real-time | 事后遗忘需要计算海森矩阵，计算成本高昂。 | 在持续全局训练的同时执行并发遗忘请求。 | 边学边忘 (UDL)。 | 在前向传播中持续更新影响矩阵，实现瞬间抵消。 |
| A Generalized Hierarchical FL Framework... | 具有理论保证的广义分层联邦学习框架 | Hierarchical FL, Theoretical Guarantees | 扁平拓扑在中央服务器引起严重的通信瓶颈。 | 证明在多个中间边缘层聚合时的收敛边界。 | 分层聚合拓扑。 | 在强凸假设下，建立严格的多层聚合理论收敛界限。 |
| Consensus and Cooperation in Networked Multi-Agent Systems | 网络化多智能体系统中的共识与合作 | Multi-Agent Systems, Consensus | 去中心化自治智能体必须在无领导者的情况下对状态达成一致。 | 在边缘变化的动态网络拓扑上实现快速收敛。 | 代数图论；共识。 | 利用网络图的拉普拉斯矩阵保证渐进的状态一致性。 |
| Enhanced FL with Adaptive Block-wise Regularization... | 采用自适应块级正则化和知识蒸馏的增强联邦学习 | Knowledge Distillation, Regularization | 恶意客户端可能通过微妙的投毒缓慢降低模型性能。 | 检测隐藏在复杂神经网络参数更新中的后门攻击。 | 块级正则化 (Block-wise Regularization)。 | 利用干净代理数据集应用知识蒸馏，从数学上验证块级完整性。 |

---

## 6. 多维度深层洞察与演进趋势 (Multi-Dimensional Analytical Deductions)
对这八十余种方法的收敛性进行评估，揭示了去中心化智能架构演进中深刻的潜在轨迹。

### 6.1 压缩与参数高效微调 (PEFT) 的共生
从历史上看，通信压缩（量化、草图化）与架构适配（LoRA、Adapters）被视为不同的学科分支。目前的文献表明了它们在系统层面上的融合。像 **FedICU** 这样的框架利用重要性感知掩码仅更新关键的 LoRA 参数。这同时起到了异质性管理工具（防止基础模型遗忘一般知识）和海量通信压缩器（仅上传极其稀疏的掩码矩阵）的作用。第二层面的暗示是，未来的联邦学习架构将不再致力于压缩密集的全局梯度，而是原生性地仅在统计上显著的发散向量（Sparse Divergence Vectors）层面进行网络流量交互。

### 6.2 从被动等待到预测性网络拓扑的演变
早期的联邦学习协议具有高度的被动性——服务器只是被动地等待梯度，丢弃掉队者，并承受由此产生的统计偏差。现代范式已转向积极的预测性控制。**FLUDE** 等系统主动对节点的历史可靠性进行画像，建立未来可用性的概率模型 。同时，客户端选择机制（如 **AdaFL**）使用加权衰减函数来预测哪些客户端将提供最高的梯度效用 。这将联邦学习从一个静态的分布式计算问题，转变为一个运筹学调度优化问题，在这个问题中，网络流量、设备电池寿命和数据熵被联合实时优化。

### 6.3 边缘侧的硬件-算法协同设计与计算液化
假设边缘硬件具有统一精度的时代已经彻底终结。诸如 **FedHQ** 的框架根据接收设备的特定神经处理单元 (NPU) 在运行时动态编译混合精度格式。更重要的是，**HAT** 框架引入的设备-云协同推理架构  表明，计算不再严格绑定在“边缘”或“云端”。通过提示词分块 (Prompt Chunking) 和重叠推测解码 (Speculative Decoding)，该架构消除了“客户端”和“服务器”之间的二元界限，创造了一个流动的计算连续体 (Liquid Computational Continuum)，在这里，张量 (Tensors) 会根据实时的带宽可用性和硅片约束，在推理过程中物理性地来回移动。

## 7. 结论性综合 (Concluding Synthesis)
对上述研究语料的详尽评估表明，联邦学习已经超越了其作为简单隐私保护平均机制的起源。它现在是一个高度复杂的跨学科框架，需要在统计学、物理层网络和密码学等多个维度进行持续优化。
双层累积量化压缩  和基于投票的网络内共识聚合  的结合，在很大程度上中和了根本的通信瓶颈。同时，同态加密类别均衡采样  和通过 **FLY-SMOTE** 在边缘动态生成合成数据等技术的部署，为应对统计异质性提供了稳健的数学解决方案。随着基础模型不断挑战内存和计算能力的极限，架构将继续向双层个性化 (Bi-level Personalization) 和持续的、推测性的设备-云协作演进。最终，去中心化智能的未来，将取决于能否通过算法化手段，将海量不可靠的异构边缘设备无缝编排成一个单一的、极具韧性的认知网络。

---

