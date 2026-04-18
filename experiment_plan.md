# DeQA Section 3.2 实验方案重设计

> **目的**：为论文 §3.2 实验部分的重写提供完整蓝图。
> **源文件**：`sections/GameTheoryAndCaseStudy.tex`（理论 L1–L266，旧 case study L361–L562）。
> **篇幅政策**：实验部分不设篇幅限制，主文保留完整 sweep 与多子图；Appendix 放扩展消融。

---

## 目录

1. [整体目标与定位](#1-整体目标与定位)
2. [原方案问题诊断与解决状态](#2-原方案问题诊断与解决状态)
3. [调整后的完整假设 / 参数清单](#3-调整后的完整假设--参数清单)
4. [重设计的四个 RQ](#4-重设计的四个-rq)
5. [Claim → RQ 覆盖矩阵](#5-claim--rq-覆盖矩阵)
6. [主文图表清单](#6-主文图表清单)
7. [执行顺序](#7-执行顺序)
8. [不纳入本轮的改动](#8-不纳入本轮的改动)
9. [相关文件索引](#9-相关文件索引)

---

## 1. 整体目标与定位

### 1.1 实验在论文中的角色

论文的 Section 3.1 用博弈论证明了生态系统在理想条件下的几个核心性质（validator 会诚实、producer 会自我筛选、整体达到均衡）。Section 3.2 的实验部分需要回答一个关键问题：**这些理论预测在多 agent 仿真中是否成立？**

我们使用 **DeQA（Decentralized Question Answering）** 作为具体场景：consumer agent 提出问题，producer agent 回答，validator agent 评估回答质量。虽然用的是问答场景，但机制设计和理论结论是通用的，适用于任何知识密集型的 agent 协作场景。

### 1.2 设计原则

- **每个实验精确对应理论 claim**：不做没有理论锚点的探索性实验，每个 RQ 都能回溯到 Section 3.1 的某条定理或命题。
- **自变量与预期现象单义**：每个实验只改变一个维度（或两个维度做 grid），确保因果关系清晰。
- **理论线 overlay 实证数据**：尽量在实验图中叠加理论预测的曲线或边界，直观展示理论与实验的吻合程度。

---

## 2. 原方案问题诊断与解决状态

### 2.1 旧方案存在的问题

旧实验方案有 13 个已识别的问题，按严重程度分为三级：

#### 🔴 高严重度（必须解决）

| # | 问题描述 | 为什么严重 |
|---|---|---|
| P1 | Agent 质量 $q$ 的分布过于集中（$[0.98, 1.00]$），几乎所有 agent 都是高质量的 | 理论预测 producer 会在某个质量门槛 $\bar{q}_p$ 处自我筛选（Theorem 2），但如果所有 agent 质量都接近 1.0，这个门槛根本观察不到 |
| P2 | Validator 的经济质押 $\sigma$ 全程固定为 1，没有做 sweep | 论文的核心 robustness 论点是"stake 是应对各种攻击的统一补偿杠杆"（Table 3），但实验完全没有验证这一点 |
| P3 | Validator 之间的相关性（correlation）没有建模 | 论文在 Section 3.1.6(B) 专门分析了残余相关性对共识的影响，"architectural heterogeneity" 是论文卖点之一，但实验中 validator 完全独立，没有测试相关性的影响 |
| P5 | Robustness 分析（Sybil 攻击、相关性、质量异质性）完全没有实验 | Section 3.1.6 花了大量篇幅分析三类扰动下的鲁棒性，但实验端完全空白，理论和实验脱节 |

#### 🟡 中严重度（应该解决）

| # | 问题描述 | 为什么重要 |
|---|---|---|
| P4 | 社会福利 $W$ 的定义有问题：用 ROC（回报率，一个比率）直接求和 | ROC 是无量纲比率，不同 agent 的 ROC 求和没有经济学意义。应该用 token 变化量（绝对值）作为福利度量 |
| P6 | "逆向选择螺旋被机制抑制"是论文 Section 1 的核心动机，但实验中没有直接度量 | 缺少展示高质量 agent 存活、低质量 agent 退出的演化过程指标（如 Gini 系数、active agent 质量分布） |
| P10 | 后验质量 $\mathbb{E}[q_p \mid Z=1]$（通过验证的 producer 的平均质量）没有作为 RQ1 的指标 | 这是衡量 PoI 共识质量的最直接指标：验证通过的 producer 到底有多好？ |
| P11 | Producer 的推广费 $\tau_p$ 和 consumer 的服务成本 $c_s$ 是隐式的，无法完整验证 P-IR 的双边区间 | 理论的 P-IR 条件有上下界，但实验只能验证一个方向 |
| P12 | Consumer IR（Theorem 3）的独立阈值没有测试 | $\tau_s$ 和 $V_c$ 没有做 sweep，无法独立验证 consumer 的参与条件 |

#### 🟢 低严重度（声明即可）

| # | 问题描述 | 处理方式 |
|---|---|---|
| P7 | Agent 策略 $t_u \in [0,1]$ 是混合策略，但理论 BNE 预测的是纯策略（要么完全诚实 $t=1$，要么完全偷懒 $t=0$） | 在写作中声明：$t_u$ 代表混合策略概率，BNE 预测 $t_u$ 会收敛到 0 或 1 的 corner |
| P8 | 仿真中 agent 每轮随机分配角色，但理论假设角色是固定的 | 在写作中声明：agent 的质量类型 $q_u$ 全程固定（与理论一致），仅角色在每轮重新分配 |
| P9 | 学习规则只有一种（模仿高回报 peer），没有声明这是 bounded-rationality 代理 | 在写作中声明 + Appendix 可选 best-response sanity check |
| P13 | 样本任务数 $m$ 固定；effort cost $c_u$ 没有现实 anchor；Baselines 偏弱 | $m$ 固定为 10；Baselines 维持 $n=0$ 和 $n=1$ 边界；留 Limitations |

### 2.2 解决状态汇总

| 问题 | 状态 | 解决方式 |
|---|---|---|
| P1 质量分布过窄 | ✅ 已解决 | 改用 $q \sim 0.5 + 0.5 \cdot \text{Beta}(2,5)$，支撑域 $[0.5, 1]$，让自筛选门槛可观测 |
| P2 Stake 固定 | ✅ 已解决 | 引入 $\sigma \in \{0, 0.5, 1, 2, 5\}$ sweep，作为 RQ2 和 RQ4 的核心自变量 |
| P3 无 correlation 建模 | ✅ 已解决 | 按 Section 3.1.6(B) 的 latent-factor mixture 模型采样；RQ4 专门测试 |
| P4 Welfare 量纲错 | ✅ 已解决 | 主指标改为 token 变化总量 $W = \sum_u(B_u^{\text{final}} - B_u^{\text{init}})$ |
| P5 Robustness 无实验 | ✅ 已解决 | RQ4 覆盖三类扰动（correlation、Sybil、质量异质性） |
| P6 逆向选择无度量 | ✅ 已解决 | RQ1 新增 token Gini 系数、population turnover、active-agent 质量分布热力图 |
| P7–P9 写作声明 | ✅ 已解决 | 在 writing 中加入声明 |
| P10 后验质量缺失 | ✅ 已解决 | RQ1 新增 posterior quality 12 子图 |
| P11 P-IR 双边区间 | ⏸ 推迟 | 留 Limitations；RQ3 用 $c_u$ sweep 间接推动门槛位移 |
| P12 C-IR 独立验证 | ⏸ 推迟 | 留 Limitations |
| P13 $m$ sweep 等 | ⏸ 推迟 | $m$ 固定为 10；Baselines 维持边界 |

> **结论**：13 条问题中，9 条已解决，1 条部分解决，3 条明确推迟到 Limitations。所有 🔴 高严重度问题全部解决。

---

## 3. 调整后的完整假设 / 参数清单

> 标记说明：🔧 = 相对旧方案修改；🆕 = 新增变量或维度；✍ = 仅 writing 声明；空 = 不变。

### 3.1 Agents 与 Quality 分布

这组参数定义了 agent 群体的基本属性。

| 参数 | 含义 | 新方案设定 | 变动 | 理论对应 |
|---|---|---|---|---|
| $N$ | agent 总数 | $10{,}000$ | — | — |
| $q$ 分布 | 每个 agent 的"能力值"，决定其回答/评估的准确率 | $q \sim 0.5 + 0.5 \cdot \text{Beta}(2,5)$，范围 $[0.5, 1]$，均值约 0.71 | 🔧 | Assumption 1 要求 $q_v > 1/2$；宽分布让 P-IR 门槛可测 |
| Shirk 信号 | agent 偷懒时的输出质量 | 准确率 = 50%（等于随机猜） | — | Assumption 2 |
| Quality 持久性 | agent 的能力是否随时间变化 | 每个 agent 的 $q_u$ 在整个 run 中固定不变；仅角色（consumer/producer/validator）每轮重新分配 | ✍ | 与理论中"private type"一致 |
| 初始 token | 每个 agent 的初始资金 | 2,000 tokens | — | — |
| 策略 $t_u$ | agent 选择诚实工作的概率 | $t_u \in [0, 1]$，其中 $t_u = 1$ 表示完全诚实，$t_u = 0$ 表示完全偷懒 | ✍ | 理论 BNE 预测收敛到 $t_u = 0$ 或 $t_u = 1$ |
| 初始策略 | agent 开始时的诚实概率 | $t_u \sim \text{Uniform}(0,1)$（冷启动，从随机状态开始） | — | — |
| 学习规则 | agent 如何更新策略 | 观察周围 agent，模仿回报率更高的 peer 的策略 | ✍ | 作为 bounded-rationality 近似，BNE 是吸引子 |

### 3.2 Validation 机制参数

这组参数控制 PoI 共识机制的运作方式。

| 参数 | 含义 | 新方案设定 | 变动 | 理论对应 |
|---|---|---|---|---|
| $n$ | 每次验证委员会的 validator 数量（必须是奇数，以便多数投票） | $\{1, 5, 9, 17, 33, 65\}$ | — | Assumption 1；$n$ 越大，共识越准确（Condorcet 定理） |
| $m$ | 每次验证中检测的样本任务数 | 10 | — | Assumption 2 |
| $\tau_{\text{val}}$ | validator 每次验证获得的基础报酬 | 1 token | — | Assumption 3 |
| **$\sigma$** | validator 必须质押的 token 数量（判断错误时没收） | $\{0, 0.5, 1, 2, 5\}$ | 🆕 | Theorem 1 的经济杠杆：$\sigma$ 越高，偷懒的代价越大 |
| **$\rho$** | validator 之间的残余相关性（0 = 完全独立，1 = 完全相关） | $\{0, 0.1, 0.3\}$，按 latent-factor mixture 模型采样 | 🆕 | Section 3.1.6(B)：即使有相关性，只要 $\rho < \rho^*$，IC 仍成立 |
| **$k/n$** | Sybil 攻击者控制的 validator 席位比例 | $\{0, 0.1, 0.3\}$（仅在 RQ4 的 $n=17$ 下测试） | 🆕 | Theorem 5：只要 $k < \lfloor n/2 \rfloor$，诚实多数仍保持共识质量 |
| **$\kappa$** | validator 质量的异质程度 | $\{\text{low}, \text{high}\}$（控制 $q_v$ 的方差） | 🆕 | Section 3.1.6(C)：异质质量下 IC 仍成立 |
| 共识规则 | 如何汇总 validator 投票 | 多数投票（majority vote） | — | Assumption 1 |

### 3.3 Promotion 与 Subscription 参数

这组参数控制 producer 推广和 consumer 订阅的经济条件。

| 参数 | 含义 | 新方案设定 | 变动 |
|---|---|---|---|
| $c_u$ | agent 做诚实工作的成本（effort cost） | 在 $(0, 1)$ 区间 sweep | — |
| $\tau_s$ | consumer 订阅一次服务需支付的费用 | 1,000 tokens（远大于验证成本 $mn\tau_{\text{val}}$，保证验证不是主要开销） | — |
| $\tau_p, c_s$ | producer 推广费 / consumer 端额外成本 | 隐式处理；完整验证留 Limitations | — |

### 3.4 仿真运行参数

| 参数 | 含义 | 新方案设定 |
|---|---|---|
| Episodes | 每次 run 的总轮数 | 100,000 |
| Seeds | 随机种子数（用于计算均值 ± 标准差） | 5 个种子，图中用阴影表示 $\pm 1$ std |
| $S_p$ | Producer 群体的"理性程度"（0 = 完全非理性/随机行动，1 = 完全理性） | $\{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ |
| $S_u$ | Consumer/Validator 群体的"理性程度" | $\{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ |

### 3.5 评估指标

| 指标 | 含义 | 计算方式 | 变动 |
|---|---|---|---|
| **社会福利 $W$**（主指标） | 整个生态系统创造的总价值 | $\sum_u (B_u^{\text{final}} - B_u^{\text{init}})$，即所有 agent 的 token 净变化总和 | 🔧 改为 token 绝对值 |
| Per-capita ROC | 单个 agent 的投资回报率，按角色分组 | $(B_u^{\text{final}} - B_u^{\text{init}}) / B_u^{\text{init}}$，分别统计 consumer 和 producer | 🔧 |
| Truthful participation $\bar{t}$ | agent 群体的平均诚实程度 | 所有活跃 agent 的 $t_u$ 均值 | — |
| **Posterior quality** | 通过 PoI 验证的 producer 的平均质量 | $\mathbb{E}[q_p \mid Z=1]$：只看验证通过的 producer，他们的平均能力有多高？ | 🆕 |
| **Promotion rate vs. $q_p$** | 不同质量的 producer 实际推广成功的比例 | 按 $q_p$ 分 bin，计算每个 bin 内 producer 发起推广的频率 | 🆕 |
| **Token Gini** | token 分配的不平等程度 | 活跃 agent 的 token 分布的 Gini 系数（0 = 完全平等，1 = 完全集中） | 🆕 |
| **Population turnover** | agent 退出生态系统的速率 | 每 1,000 轮中 token 归零退出的 agent 比例 | 🆕 |
| **Active-agent $q_p$ 分布** | 存活 agent 的质量分布随时间的变化 | 时间序列密度图 / 热力图，观察高质量 agent 是否越来越多 | 🆕 |
| **Empirical $\hat{q}_p$** | 实验中观测到的 producer 自筛选门槛 | Promotion rate 从 $\approx 0$ 跳到 $\approx 1$ 的 $q_p$ 值 | 🆕 |
| **Slash rate** | validator 被惩罚（没收 stake）的比例 | 被 slash 的 validator 数 / 总 validator 数（用于 RQ4） | 🆕 |

### 3.6 Baselines

我们不引入外部 baseline 系统，而是使用机制本身的极端参数设置作为对照：

- **No-PoI（$n=0$）**：完全没有验证机制，模拟"柠檬市场"场景。没有任何质量筛选，低质量 agent 可以自由参与。
- **Single-Validator（$n=1$）**：只有一个 validator 做判断，对应 Theorem 1 预测的"垄断失败"场景（单个 validator 没有足够的信息优势来保证诚实）。

---

## 4. 重设计的四个 RQ

> **全局默认参数**：除各 RQ 表中标明的"自变量"外，其余参数取默认值：$n=9$，$\sigma=1$，$\rho=0$，$S_p=S_u=1$。

---

### RQ1 — 双循环演化与 BNE 收敛

#### 核心问题

**当 agent 群体的理性程度从低到高变化时，生态系统的行为是否收敛到理论预测的均衡状态？**

#### 为什么重要

这是整个实验最基础的验证：Theorem 4 证明了一个对称 BNE 的存在，其中所有 agent 都应该诚实参与。RQ1 要验证这个均衡在仿真中是否真的是一个吸引子——即使 agent 从随机策略开始，是否会逐渐收敛到诚实参与？

同时，RQ1 还要验证论文 Section 1 的核心叙事：PoI 机制能否抑制逆向选择螺旋？即低质量 agent 是否会被逐渐淘汰，高质量 agent 是否会存活并繁荣？

#### 验证的理论 claim

- **Theorem 4（Symmetric BNE）**：所有 agent 的最优策略是诚实参与
- **Section 2 dual-loop 叙事**：tokenomics-knowledge 双循环驱动生态系统良性演化
- **Section 1 核心动机**："逆向选择螺旋被机制抑制"

#### 实验设计

| 项 | 内容 |
|---|---|
| 自变量 | Producer 理性程度 $S_p \in \{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ × Consumer/Validator 理性程度 $S_u \in \{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$，共 36 种组合 |
| 因变量 | 平均诚实度 $\bar{t}$、社会福利 $W$（总量 / consumer / producer）、per-capita ROC、posterior quality、token Gini、population turnover、active-agent $q_p$ 分布 |
| 控制变量 | $n=9$，$\sigma=1$，$\rho=0$（标准无扰动设置） |

#### 预期结果

1. **BNE 收敛**：$S_p$ 和 $S_u$ 越高，$\bar{t}$ 越接近 1（agent 越理性就越倾向于诚实，因为诚实是均衡策略）
2. **Welfare 分化**：producer 的个体回报先降后稳（因为自筛选淘汰了低质量 producer），而 consumer 和总福利持续上升
3. **Posterior quality 提升**：$\bar{t}$ 越高，通过验证的 producer 质量越高——PoI 共识确实在筛选质量
4. **逆向选择反转**：active agent 的 $q_p$ 分布随时间右偏（高质量 agent 存活，低质量 agent 退出）
5. **不出现赢家通吃**：token Gini 保持稳定，不出现极端集中

#### 对应图表

F1（ROC 12 子图）、F2（Truthful participation 12 子图）、F3（Posterior quality 12 子图）、F4（Token Gini / active agent count 时间序列）、F5（Active-agent $q_p$ 分布演化热力图）

---

### RQ2 — V-IC 的两个杠杆：信息 ($n$) 与经济 ($\sigma$)

#### 核心问题

**Validator 诚实报告的激励条件如何同时依赖于委员会大小（信息杠杆）和质押金额（经济杠杆）？**

#### 为什么重要

Theorem 1 给出了 V-IC 的核心不等式：

$$(\tau_{\text{val}} + 2\sigma) \cdot (P_{\text{match}}(n, q_v) - \frac{1}{2}) > c_u$$

这个不等式有两个"杠杆"：
- **信息杠杆**：$P_{\text{match}}(n, q_v) - \frac{1}{2}$ 随 $n$ 增大而增大（更多 validator → 更准确的共识 → 诚实 validator 更可能和多数一致）
- **经济杠杆**：$\tau_{\text{val}} + 2\sigma$ 随 $\sigma$ 增大而增大（质押越高 → 偷懒的代价越大）

RQ2 要验证：单独的信息杠杆不够（$\sigma=0$ 时即使 $n$ 很大，高 $c_u$ 下仍会偷懒），必须两个杠杆配合才能保证诚实。

#### 验证的理论 claim

- **Theorem 1（V-IC 不等式）**的因式结构：$n$ 和 $\sigma$ 的交互效应
- **Table 3**：stake 作为 universal compensation lever

#### 实验设计

| 项 | 内容 |
|---|---|
| 自变量 | 委员会大小 $n \in \{1, 5, 9, 17, 33, 65\}$ × 质押金额 $\sigma \in \{0, 0.5, 1, 2, 5\}$ × effort cost $c_u$ sweep |
| 因变量 | $\bar{t}$、posterior quality、social welfare $W$（总量 / consumer / producer） |
| 控制变量 | $\rho=0$（无相关性），$S_p=S_u=1$（完全理性），$q$ 用 §3.1 的宽分布 |

#### 预期结果

1. **$\sigma=0$ 下信息杠杆不足**：即使 $n$ 增大到 65，当 effort cost $c_u$ 足够高时，validator 仍然会偷懒——因为没有经济惩罚
2. **$\sigma$ 增大推移 IC 边界**：IC 可行区域（诚实报告成为最优策略的参数区域）随 $\sigma$ 增大而扩展到更高的 $c_u$
3. **Welfare 对 $n$ 先升后降**：$n$ 增大提高共识质量但增加验证成本，存在最优 $n^*$；**$n^*$ 的位置随 $\sigma$ 漂移**
4. **实证 IC 边界与理论曲线对齐**：实验中观测到的 $\bar{t}$ 相变点与理论预测的 $(\tau_{\text{val}} + 2\sigma)(P_{\text{match}} - \frac{1}{2}) = c_u$ 曲线吻合

#### 对应图表

F6（IC 可行区主图：$c_u \times \sigma$ heatmap，按 $n$ 分层，叠加理论边界）、F7（Welfare vs. $n$，按 $\sigma$ 分组）

---

### RQ3 — P-IR 门槛与自筛选效应

#### 核心问题

**Producer 是否真的会根据自身质量进行自我筛选——质量低于门槛的 producer 是否会主动退出市场？**

#### 为什么重要

Theorem 2 证明了一个阈值结构：存在一个质量门槛 $\bar{q}_p$，只有 $q_p \geq \bar{q}_p$ 的 producer 推广服务才是理性的（expected profit $\geq 0$）。这意味着 PoI 机制内建了一个质量筛选器——不需要中心化的质量审核，低质量 producer 会因为无法通过验证（从而亏损）而自动退出。

这是论文区别于传统中心化方案的核心论点之一。RQ3 直接验证这个自筛选是否在仿真中发生。

#### 验证的理论 claim

- **Theorem 2（Producer IR）**：存在唯一阈值 $\bar{q}_p$，低于此阈值的 producer 不参与
- **MLRP（Assumption 4）**：pass probability $V(q_p)$ 关于 $q_p$ 严格递增

#### 实验设计

| 项 | 内容 |
|---|---|
| 自变量 | 按 $q_p$ 分 bin 分析；辅助变量 $c_u \in \{0.2, 0.5, 0.8\}$（改变 effort cost 会移动理论门槛 $\bar{q}_p$ 的位置） |
| 因变量 | promotion rate vs. $q_p$（按质量分组的推广频率）、pass probability $V(q_p)$、producer ROC vs. $q_p$、active lifetime vs. $q_p$、prior vs. posterior quality 分布对比 |
| 控制变量 | $n=9$，$\sigma=1$，$\rho=0$，$S_p=S_u=1$ |

#### 预期结果

1. **阶跃函数**：promotion rate 对 $q_p$ 呈现明显的阶跃——低质量 producer 的推广率接近 0，高质量 producer 接近 1，转折点就是经验门槛 $\hat{q}_p$
2. **门槛吻合**：经验观测的 $\hat{q}_p$ 与理论计算的 $\bar{q}_p$ 吻合，并且随 $c_u$ 增大而右移（effort cost 越高，只有更高质量的 producer 才值得参与）
3. **门槛以下亏损**：$q_p < \hat{q}_p$ 的 producer 的中位 ROC $\leq 0$（他们在亏钱），活跃寿命短
4. **后验优于先验**：$\mathbb{E}[q_p \mid Z=1]$ 显著高于 $\mathbb{E}[q_p]$——验证通过的 producer 平均质量远高于群体平均

#### 对应图表

F8（Promotion rate vs. $q_p$ 阶跃图）、F9（Producer ROC 和 active lifetime vs. $q_p$）、F10（Prior vs. posterior quality 分布对比）、F11（经验 $\hat{q}_p$ vs. 理论 $\bar{q}_p$ 对齐散点图）

---

### RQ4 — Robustness：三类扰动与 stake 作为统一补偿杠杆

#### 核心问题

**当现实中的三类不理想因素（validator 相关性、Sybil 攻击、validator 质量差异）出现时，提高质押 $\sigma$ 是否能统一地补偿这些扰动，维持系统正常运作？**

#### 为什么重要

Section 3.1.6 分析了三类对理想化假设的放松：
- **(A) Sybil 攻击**：攻击者控制委员会中 $k$ 个席位，协同投票破坏共识
- **(B) 残余相关性**：即使强制 architectural heterogeneity，validator 之间仍可能因训练数据重叠而存在残余相关性
- **(C) 异质质量**：validator 的质量不是同质的，而是各有高低

论文的核心 robustness 论点是：**这三类扰动虽然机制不同，但都通过同一个渠道起作用——降低诚实 validator 的信息优势 $(P_{\text{match}} - \frac{1}{2})$，而提高 $\sigma$ 可以统一地补偿这个损失。** 这意味着协议设计者只需要调节一个旋钮（$\sigma$）就能应对各种攻击场景。

#### 验证的理论 claim

- **Theorem 5（Sybil IC）**：$k < \lfloor n/2 \rfloor$ 时，提高 $\sigma$ 可恢复 IC
- **Section 3.1.6(B)**：$\rho < \rho^*$ 时 IC 成立，$\rho^*$ 随 $\sigma$ 增大
- **Section 3.1.6(C)**：异质质量下 IC 仍成立
- **Table 3**：$\sigma$ 是统一的补偿杠杆

#### 实验设计

分三组子实验：

| 子实验 | 自变量 | 含义 |
|---|---|---|
| **R4a — Correlation** | $\rho \in \{0, 0.1, 0.3\}$ × $\sigma \in \{0.5, 1, 2, 5\}$ | 测试残余相关性的影响以及 $\sigma$ 能否补偿 |
| **R4b — Sybil** | $k/n \in \{0, 0.1, 0.3\}$ × $\sigma \in \{0.5, 1, 2, 5\}$（$n=17$） | 测试 Sybil 攻击的影响以及 $\sigma$ 能否补偿 |
| **R4c — 异质质量** | $\kappa \in \{\text{low}, \text{high}\}$（validator 质量方差） | 测试质量差异的影响 |

所有子实验共享：$n=17$（足以容纳 Sybil 攻击），$S_p=S_u=1$，$c_u$ 中等值。

因变量：$\bar{t}$、posterior quality、social welfare、honest-validator ROC、slash rate。

#### 预期结果

1. **扰动增强 → 性能下降**：$\sigma$ 固定时，增大 $\rho$ / $k/n$ / $\kappa$ 会导致 truthful rate 和 posterior quality 下降
2. **$\sigma$ 增大 → 性能恢复**：扰动固定时，提高 $\sigma$ 能把性能拉回接近无扰动的 clean baseline——**这是 "universal lever" 的直接证据**
3. **补偿曲线形状相似**：三类扰动的"扰动强度 vs. 所需 $\sigma$"补偿曲线具有相似的单调结构，支持 Table 3 的统一叙事
4. **理论界吻合**：实验中观测的临界 $\rho^*$（correlation 可容忍上界）和 $k^*$（Sybil 可容忍上界）随 $\sigma$ 增长的趋势与 Appendix 理论界一致

#### 对应图表

F12（$\rho \times \sigma$ heatmap，叠加理论 $\rho^*(\sigma)$ 曲线）、F13（$k/n \times \sigma$ heatmap）、F14（三类扰动的统一补偿曲线：归一化攻击强度 × posterior quality）

---

## 5. Claim → RQ 覆盖矩阵

此矩阵展示每条理论 claim 在新旧方案中的实验覆盖情况：

| 理论 Claim | 旧方案覆盖 | 新方案覆盖 |
|---|---|---|
| Thm 1 V-IC（$n$ 的 comparative statics） | RQ2 间接 | **RQ2 完整** |
| Thm 1 V-IC（经济杠杆 $\sigma$） | ✗ 未覆盖 | **RQ2 + RQ4 核心自变量** |
| Thm 2 P-IR 门槛 $\bar{q}_p$ | ✗ 未覆盖 | **RQ3 核心** |
| Thm 3 C-IR 独立阈值 | ✗ 未覆盖 | ⏸ 推迟到 Limitations |
| Thm 4 BNE 收敛 | RQ1 间接 | **RQ1 完整 + 新指标** |
| Sec 3.1.6(A) Sybil 攻击 | ✗ 未覆盖 | **RQ4-b** |
| Sec 3.1.6(B) 残余 correlation | ✗ 未覆盖 | **RQ4-a** |
| Sec 3.1.6(C) 异质 validator 质量 | △ 边缘 | **RQ4-c** |
| Table 3 "stake 是统一补偿杠杆" | ✗ 未覆盖 | **RQ4 统一补偿曲线（F14）** |
| Dual-loop 提升 posterior quality | 隐式 | **RQ1 显式（F3）** |
| 逆向选择螺旋被抑制 | ✗ 未覆盖 | **RQ1 显式（F4, F5）** |

---

## 6. 主文图表清单

| 编号 | 图名 | 所属 RQ | 展示内容 | 状态 |
|---|---|---|---|---|
| F1 | ROC 12 子图 | RQ1 | $6 \times 6$ 理性程度 grid 下的 per-capita ROC 演化 | 保留 + 量纲修改 |
| F2 | Truthful participation 12 子图 | RQ1 | $\bar{t}$ 随时间的演化，展示 BNE 收敛 | 保留 |
| F3 | Posterior quality 12 子图 | RQ1 | $\mathbb{E}[q_p \mid Z=1]$ 随时间演化 | 🆕 |
| F4 | Token Gini / active agent count | RQ1 | 两条时间序列：财富不平等 + 存活 agent 数 | 🆕 |
| F5 | Active-agent $q_p$ 分布热力图 | RQ1 | 时间 × $q_p$ 的密度图，展示质量分布右偏 | 🆕 |
| F6 | IC 可行区主图 | RQ2 | $c_u \times \sigma$ heatmap，按 $n$ 分层，叠加理论 IC 边界 | 🆕 核心新图 |
| F7 | Welfare vs. $n$ | RQ2 | 按 $\sigma$ 分组的福利曲线，展示最优 $n^*$ 的漂移 | 改造自旧图 |
| F8 | Promotion rate vs. $q_p$ | RQ3 | 阶跃函数图，展示自筛选门槛 | 🆕 |
| F9 | Producer ROC & lifetime vs. $q_p$ | RQ3 | 门槛以下亏损 + 短寿命 | 🆕 |
| F10 | Prior vs. posterior quality | RQ3 | 两个分布的对比，展示 PoI 的筛选效果 | 🆕 |
| F11 | $\hat{q}_p$ vs. $\bar{q}_p$ 散点 | RQ3 | 经验门槛 vs. 理论门槛的对齐 | 🆕 |
| F12 | $\rho \times \sigma$ heatmap | RQ4-a | Correlation robustness，叠加理论 $\rho^*(\sigma)$ | 🆕 |
| F13 | $k/n \times \sigma$ heatmap | RQ4-b | Sybil robustness | 🆕 |
| F14 | 三类扰动统一补偿曲线 | RQ4 | 归一化攻击强度 × posterior quality，三条曲线形状相似 | 🆕 核心新图 |

**合计 14 张主文图**。Appendix 放扩展内容：额外 sweep、异质质量分解、best-response sanity check、长时段演化等。

---

## 7. 执行顺序

### Phase 1：代码修改（可并行）

| 步骤 | 任务 | 类型 | 说明 |
|---|---|---|---|
| 1 | 修改 $q$ 分布 | 修改 | 从 $[0.98, 1.0]$ 改为 $0.5 + 0.5 \cdot \text{Beta}(2,5)$，范围 $[0.5, 1]$ |
| 2 | 修改 welfare 计算 | 修改 | ROC 求和 → token 净变化总量 |
| 3 | 添加 $\sigma$ sweep 维度 | 新增 | 质押金额作为可配置参数，支持多值 sweep |
| 4 | 添加 latent-factor mixture（$\rho$） | 新增 | 按 Section 3.1.6(B) 的模型实现 validator 相关性采样 |
| 5 | 添加 Sybil $k/n$ 采样 | 新增 | 委员会中注入 $k$ 个恶意 validator，需依赖步骤 4 的框架 |
| 6 | 添加异质 $q_v$（$\kappa$） | 新增 | Validator 质量从不同方差的分布中采样 |

### Phase 2：实验运行（按依赖顺序）

| 步骤 | 任务 | 依赖 |
|---|---|---|
| 7 | 跑 RQ1 全套（$S_p \times S_u$ grid，36 种组合 × 5 seeds） | 步骤 1, 2 |
| 8 | 跑 RQ2 full grid（$n \times \sigma \times c_u$） | 步骤 1, 2, 3 |
| 9 | 跑 RQ3（按 $q_p$ 分 bin 分析 + $c_u$ 三值） | 步骤 1, 2 |
| 10 | 跑 RQ4-a / 4-b / 4-c（三组 robustness 子实验） | 步骤 3, 4, 5, 6 |

### Phase 3：写作与出图

| 步骤 | 任务 | 依赖 |
|---|---|---|
| 11 | Writing 声明调整（T2-c, T2-d, T2-e） | 可与 Phase 2 并行 |
| 12 | 出图 14 张 + 主文 Section 3.2 改写 | 步骤 7–10 完成后 |

---

## 8. 不纳入本轮的改动

以下内容明确推迟到 Limitations / Future Work，不在本轮实验中处理：

| 内容 | 推迟原因 |
|---|---|
| **C-IR 独立验证**（Theorem 3）：$\tau_s$ 和 $V_c$ 的 sweep | Consumer 的参与条件验证需要额外的实验维度，且理论框架已充分支撑 |
| **P-IR 双边区间完整验证**：$\tau_p$ 和 $c_s$ 显式化 | 完整验证需要 producer 端的成本参数可调，当前框架中这些是隐式的 |
| **$m$ sweep**（样本任务数的影响） | 论文已在理论上证明 $m=1$ 分析可推广（Appendix A.1.2），实验中固定 $m=10$ |
| **$c_u$ 的现实 anchor** | 当前保持无量纲的 effort-fee ratio，不引入具体场景的成本标定 |
| **Baselines 扩展**：中心化 orchestrator / reputation-based 对照 | 本文核心是机制内部的激励分析，跨机制对比留给后续工作 |
| **完整 Sybil 理论**（voluntary collusion） | 本轮只做简化 $k/n$ grid；正式的 communication-proof equilibrium 分析留给理论扩展 |
| **统计显著性检验** | 仅报告 mean ± std（5 seeds），不做假设检验 |

---

## 9. 相关文件索引

| 文件 | 内容 |
|---|---|
| `sections/GameTheoryAndCaseStudy.tex` L1–L266 | Section 3.1 理论部分（Assumptions, Theorems 1–5, Robustness） |
| `sections/GameTheoryAndCaseStudy.tex` L361–L562 | Section 3.2 旧 case study（待重写） |
| `sections/GameTheoryAndCaseStudy.tex` L269–L357 | 旧图环境（RQ1 的两张图） |
| `sections/GameTheoryAndCaseStudy.tex` L385–L447 | 旧图环境（RQ2/RQ3 的两张图） |
| `figures/fixed_sp/`, `figures/fixed_su/`, `figures/hard/`, `figures/quality/` | 图资源目录 |
| `experiment_plan.md` | 本方案文档 |
