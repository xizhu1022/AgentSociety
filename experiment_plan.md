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

- **载体**：DeQA（Decentralized Question Answering），作为 §3.1 理论的实证验证。
- **目标**：在同一参数化 simulation 下，用四个 RQ 覆盖 Sec 3.1 的所有主要 claims：V-IC 的因式结构、P-IR 阈值自筛选、BNE 收敛、以及三类扰动下 stake 作为 universal compensation lever。
- **核心原则**：每个 RQ 精确对应 $\geq 1$ 条理论 claim；自变量与预期现象单义；尽量以"理论线 overlay 实证 colormap"的形式呈现对齐证据。

---

## 2. 原方案问题诊断与解决状态

### 2.1 旧方案主要问题

| # | 问题 | 严重度 |
|---|---|---|
| P1 | $q$ 分布过窄（$[0.98, 1.00]$），Thm 2 自筛选门槛无法显现 | 🔴 高 |
| P2 | Stake $\sigma$ 全程固定为 1，"stake 是 universal compensation lever"（Sec 3.1.6 Table 3）在实验端悬空 | 🔴 高 |
| P3 | Validator 独立性 / correlation 未建模，"architectural heterogeneity" 只是口号 | 🔴 高 |
| P4 | Welfare 定义 $W=\sum_u\text{ROC}_u$ 量纲错（比率求和） | 🟡 中 |
| P5 | Robustness（Sybil / correlation / heterogeneity）完全无实验 | 🔴 高 |
| P6 | Adverse selection spiral 是 Sec 1 核心动机，却无直接度量（Gini / turnover / quality 演化） | 🟡 中 |
| P7 | $t_u\in[0,1]$ 作为混合策略 vs. 理论 BNE 的纯策略 corner，未声明 | 🟢 低 |
| P8 | 角色每轮随机重分配 vs. 理论 fixed type，未 justify | 🟢 低 |
| P9 | 学习规则单一（imitation only），未声明 bounded-rationality 代理身份 | 🟢 低 |
| P10 | Posterior quality $\mathbb E[q_p\mid Z=1]$ 未作为 RQ1 指标 | 🟡 中 |
| P11 | $\tau_p, c_s$ 隐式；P-IR 双边区间无法完整验证 | 🟡 中 |
| P12 | C-IR 独立阈值未测；$\tau_s, V_c$ 无 sweep | 🟡 中 |
| P13 | $m$ 固定；$c_u$ 无现实 anchor；Baselines 偏弱 | 🟢 低 |

### 2.2 解决状态

| 问题 | 处置 | 如何解决 |
|---|---|---|
| P1 | ✅ 解决 | T1-a 拓宽 $q \sim 0.5+0.5\cdot\text{Beta}(2,5)$；RQ3 专门验证门槛 |
| P2 | ✅ 解决 | T1-b 引入 $\sigma$-sweep；RQ2 / RQ4 作为核心自变量 |
| P3 | ✅ 解决 | T2-a 按 Sec 3.1.6(B) latent-factor mixture 采样；RQ4 专测 |
| P4 | ✅ 解决 | T2-b 主指标改 token-denominated $W=\sum_u(B_u^\text{final}-B_u^\text{init})$ |
| P5 | ✅ 部分解决 | RQ4 覆盖 correlation + Sybil $k/n$ + heterogeneous quality；Sybil 用简化 grid |
| P6 | ✅ 解决 | RQ1 新增 posterior quality 12 子图、token Gini 时间序列、active-agent $q_p$ 演化热力图 |
| P7 | ✅ 解决 | T2-d writing 声明（$t_u$ 为 mixed strategy，BNE 在 corner） |
| P8 | ✅ 解决 | T2-c writing 声明（quality type fixed，仅 role 每轮 draw） |
| P9 | ✅ 解决 | T2-e writing 声明 + Appendix 可选 best-response sanity check |
| P10 | ✅ 解决 | RQ1 新增 F3 posterior quality 12 子图 |
| P11 | ⏸ 留 Limitations | 本轮不展开；RQ3 用 $c_u$ sweep 间接推动 $\bar q_p$ 位移 |
| P12 | ⏸ 留 Limitations | 本轮不展开 |
| P13 | ⏸ 留 Limitations | $m$ 固定为 10；Baselines 维持 $n=0, n=1$ 边界 |

> **结论**：13 条问题中，9 条在新方案里解决，1 条部分解决，3 条明确推迟到 Limitations。三条 🔴 高严重度问题（P1, P2, P3, P5）全部解决。

---

## 3. 调整后的完整假设 / 参数清单

> 🔧 = 相对旧方案修改；🆕 = 新增变量或维度；✍ = 仅 writing 声明；空 = 不变。

### 3.1 Agents & Quality

| 参数 | 新方案设定 | 变动 | 理论对应 |
|---|---|---|---|
| $N$ agent 数 | $10{,}000$ | — | — |
| Agent quality $q$ 分布 | $q \sim 0.5 + 0.5\cdot\text{Beta}(2,5)$，支撑 $[0.5, 1]$ | 🔧 | Assum 1 $q_v>1/2$；让 P-IR 门槛可测 |
| Shirk 时信号 | $\Pr(r=1)=1/2$ | — | Assum 2 |
| Quality type 持久性 | Agent 的 $q_u$ 在整个 run 中 fixed；仅 role 每轮重分配 | ✍ | 与理论 private type 一致 |
| 初始 token | $2{,}000$ | — | — |
| 策略 $t_u \in [0,1]$ | Mixed strategy over $e\in\{0,1\}$；BNE 在 corner | ✍ | Assum 2 / Thm 1 |
| 初始策略 | $t_u \sim \text{Uniform}(0,1)$（cold start） | — | — |
| 学习规则 | 低 ROC imitate 高 ROC peer；作为 bounded-rationality 代理 | ✍ | Thm 4 BNE 收敛 |

### 3.2 Validation Mechanism

| 参数 | 新方案设定 | 变动 | 理论对应 |
|---|---|---|---|
| Committee 大小 $n$ | $\{1, 5, 9, 17, 33, 65\}$（奇数） | — | Assum 1 |
| Sample queries $m$ | $10$ | — | Assum 2 |
| Validation fee $\tau_\text{val}$ | $1$ | — | Assum 3 |
| **Stake $\sigma$** | $\{0, 0.5, 1, 2, 5\}$ | 🆕 | Thm 1 经济杠杆；Table 3 universal lever |
| **残余 correlation $\rho$** | $\{0, 0.1, 0.3\}$，按 latent-factor mixture 采样 | 🆕 | Sec 3.1.6(B) |
| **Sybil 比例 $k/n$** | $\{0, 0.1, 0.3\}$（仅在 RQ4 的 $n=17$ 触点） | 🆕 | Thm 5（简化验证） |
| **Validator 质量异质度 $\kappa$** | $\{\text{low}, \text{high}\}$（$q_v$ 方差尺度） | 🆕 | Sec 3.1.6(C) |
| Consensus | Majority vote | — | Assum 1 |

### 3.3 Promotion & Subscription

| 参数 | 新方案设定 | 变动 |
|---|---|---|
| Effort cost $c_u$ | $(0, 1)$ sweep | — |
| Subscription fee $\tau_s$ | $1{,}000$（保证 $mn\tau_\text{val}\ll\tau_s$） | — |
| $\tau_p, c_s$ | 隐式；留 Limitations | — |

### 3.4 Evolution & Protocol

| 参数 | 新方案设定 | 变动 |
|---|---|---|
| Episodes / run | $100{,}000$ | — |
| Seeds | 5，阴影 $\pm 1$ std | — |
| Rationality $S_p$ | $\{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ | — |
| Rationality $S_u$ | $\{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ | — |

### 3.5 Evaluation Metrics

| 指标 | 定义 | 变动 |
|---|---|---|
| **Social welfare $W$（主）** | $\sum_u (B_u^\text{final}-B_u^\text{init})$ token-denominated | 🔧 |
| Per-capita ROC（辅） | $(B_u^\text{final}-B_u^\text{init})/B_u^\text{init}$，分 consumer / producer | 🔧 |
| Truthful participation $\bar t$ | $\frac{1}{|\mathcal U_\text{active}|}\sum_u t_u$ | — |
| **Posterior quality** | $\mathbb E[q_p\mid Z=1]$ | — |
| **Promotion rate vs. $q_p$** | 按 $q_p$ 分 bin 的 promotion 频率 | 🆕 |
| **Token Gini** | Active agents 的 token 分布 Gini 系数 | 🆕 |
| **Population turnover** | 每 1{,}000 episodes 的 exit rate | 🆕 |
| **Active-agent $q_p$ 分布** | 时间序列 density / 热力图 | 🆕 |
| **Empirical $\hat q_p$** | Promotion 阶跃点的经验阈值 | 🆕 |
| **Slash rate** | 被 slash 的 validator 比例（RQ4） | 🆕 |

### 3.6 Baselines（参数化边界，不变）

- **No-PoI**：$n=0$ → market-for-lemons
- **Single-Validator**：$n=1$ → 对应 Thm 1 预测的 monopoly failure

---

## 4. 重设计的四个 RQ

> **全局默认**：除下表"自变量"外，其余参数取 §3 默认值（$n=9$，$\sigma=1$，$\rho=0$，$S_p=S_u=1$ 作为 moderate baseline）。

### 🔸 RQ1 — Dual-loop 演化与 BNE 收敛

**验证 claim**：Thm 4（Symmetric BNE）+ Sec 2 dual-loop 叙事 + "adverse selection spiral 被机制抑制"

| 项 | 内容 |
|---|---|
| 自变量 | $S_p \in \{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ × $S_u \in \{0, 0.2, 0.4, 0.6, 0.8, 1.0\}$ |
| 因变量 | $\bar t$；token-denominated $W, W_c, W_p$；per-capita ROC（分角色）；**posterior quality $\mathbb E[q_p\mid Z=1]$**；**token Gini**；**population turnover**；**active-agent $q_p$ 分布** |
| 控制变量 | $n=9$；$\sigma=1$；$\rho=0$ |
| 预期现象 | (a) $S_p, S_u \uparrow$ 时 $\bar t \to 1$（BNE corner 收敛）；(b) producer 个体 surplus 先降后稳，consumer / total welfare 持续上升；(c) posterior quality 随 $\bar t$ 单调提升；(d) active-agent $q_p$ 分布随时间右偏（adverse selection 反向过程）；(e) token Gini 稳定，不出现赢家通吃崩盘 |
| 主图 | F1, F2, F3, F4, F5（见 §6） |

### 🔸 RQ2 — V-IC 两个杠杆：信息 ($n$) × 经济 ($\sigma$)

**验证 claim**：Thm 1 V-IC 不等式 $(\tau_\text{val}+2\sigma)(P_\text{match}(n,q_v)-\tfrac12) > c_u$ 的因式结构

| 项 | 内容 |
|---|---|
| 自变量 | $n \in \{1, 5, 9, 17, 33, 65\}$ × $\sigma \in \{0, 0.5, 1, 2, 5\}$ × $c_u$ sweep |
| 因变量 | $\bar t$；posterior quality；token-denominated $W, W_c, W_p$ |
| 控制变量 | $\rho=0$；$S_p=S_u=1$；$q$ 用 §3.1 分布 |
| 预期现象 | (a) $\sigma=0$：即使 $n\uparrow$，高 $c_u$ 下仍 shirk —— 证明 information lever 不足；(b) $\sigma\uparrow$：IC 边界向高 $c_u$ 推移；(c) welfare 对 $n$ 仍先升后降，但**峰值位置随 $\sigma$ 漂移**；(d) 实证 IC 边界与理论曲线 $(\tau_\text{val}+2\sigma)(P_\text{match}-\tfrac12)=c_u$ 对齐 |
| 主图 | F6, F7（见 §6） |

### 🔸 RQ3 — P-IR 门槛与 self-screening

**验证 claim**：Thm 2（Producer IR）+ MLRP 导出的唯一阈值 $\bar q_p$

| 项 | 内容 |
|---|---|
| 自变量 | 按 $q_p$ 分 bin 的分析；辅助 $c_u \in \{0.2, 0.5, 0.8\}$（改变理论 $\bar q_p$ 位置） |
| 因变量 | **promotion rate vs. $q_p$**；pass probability $V(q_p)$；producer ROC vs. $q_p$；active lifetime vs. $q_p$；prior vs. posterior quality 分布 |
| 控制变量 | $n=9$；$\sigma=1$；$\rho=0$；$S_p=S_u=1$ |
| 预期现象 | (a) promotion rate 对 $q_p$ 呈阶跃（$\approx 0 \to \approx 1$ 在经验 $\hat q_p$）；(b) $\hat q_p$ 与理论 $\bar q_p$ 吻合，且随 $c_u$ 移动；(c) $q_p<\hat q_p$ 的 producer ROC 中位数 $\leq 0$，lifetime 短；(d) posterior $\mathbb E[q_p\mid Z=1]$ 显著高于 prior $\mathbb E[q_p]$ |
| 主图 | F8, F9, F10, F11（见 §6） |

### 🔸 RQ4 — Robustness：三类扰动与 stake universal lever

**验证 claim**：Sec 3.1.6 三条 robustness（Sybil / correlation / heterogeneous quality）+ Table 3 stake 是统一补偿杠杆

| 项 | 内容 |
|---|---|
| 自变量（分三组） | **R4a** $\rho \in \{0, 0.1, 0.3\}$ × $\sigma \in \{0.5, 1, 2, 5\}$；**R4b** $k/n \in \{0, 0.1, 0.3\}$ × $\sigma \in \{0.5, 1, 2, 5\}$（$n=17$）；**R4c** $\kappa \in \{\text{low}, \text{high}\}$（validator 质量方差） |
| 因变量 | $\bar t$；posterior quality；token-denominated welfare；honest-validator ROC；**slash rate** |
| 控制变量 | $n=17$（足以容纳 Sybil）；$S_p=S_u=1$；$c_u$ 中等值 |
| 预期现象 | (a) $\sigma$ 固定、扰动增强 → truthful rate & posterior quality 下降；(b) **扰动固定、$\sigma\uparrow$ 能把性能拉回接近 clean baseline**（universal lever）；(c) 三类扰动的"补偿曲线"形状相似 → Table 3 得到实证支持；(d) 实证 $\rho^*, k^*$ 随 $\sigma$ 增长，与 Appendix 理论界吻合 |
| 主图 | F12, F13, F14（见 §6） |

---

## 5. Claim → RQ 覆盖矩阵

| 理论 claim | 旧方案 | 新方案 |
|---|---|---|
| Thm 1 V-IC（对 $n$ comparative statics） | RQ2 间接 | **RQ2 完整** |
| Thm 1 V-IC（经济杠杆 $\sigma, \tau_\text{val}$） | ✗ | **RQ2 + RQ4 核心自变量** |
| Thm 2 P-IR 门槛 $\bar q_p$ | ✗ | **RQ3 核心** |
| Thm 3 C-IR 独立阈值 | ✗ | ⏸ Limitations |
| Thm 4 BNE 收敛 | RQ1 间接 | **RQ1 完整 + 新指标** |
| Sec 3.1.6(B) residual correlation | ✗ | **RQ4-a** |
| Sec 3.1.6(C) heterogeneous quality | △ | **RQ4-c** |
| Thm 5 Sybil IC（简化验证） | ✗ | **RQ4-b** |
| Table 3 "stake universal lever" | ✗ | **RQ4 unified 补偿曲线（F14）** |
| "Dual-loop 提升 posterior quality" | 隐式 | **RQ1 显式（F3）** |
| "Adverse selection spiral 被抑制" | ✗ | **RQ1 显式（F4, F5）** |

---

## 6. 主文图表清单

| 编号 | 图名 | 所属 RQ | 状态 |
|---|---|---|---|
| F1 | ROC 12 子图（$S_p \times S_u$ grid） | RQ1 | 保留 + welfare 量纲改 |
| F2 | Truthful participation 12 子图 | RQ1 | 保留 |
| F3 | Posterior quality 12 子图 | RQ1 | 🆕 |
| F4 | Token Gini / active agent count 时间序列 | RQ1 | 🆕 |
| F5 | Active-agent $q_p$ 分布演化热力图 | RQ1 | 🆕 |
| F6 | IC 可行区主图（$c_u \times \sigma$，按 $n$ 分层，理论边界 overlay） | RQ2 | 🆕 核心新图 |
| F7 | Welfare vs. $n$（按 $\sigma$ 分组） | RQ2 | 改造自 `fig:cost_validators` |
| F8 | Promotion rate vs. $q_p$ 阶跃图 | RQ3 | 🆕 |
| F9 | Producer ROC & active lifetime vs. $q_p$ | RQ3 | 🆕 |
| F10 | Prior vs. posterior quality 分布对比 | RQ3 | 🆕 |
| F11 | $\hat q_p$（经验） vs. $\bar q_p$（理论） 对齐散点 | RQ3 | 🆕 |
| F12 | $\rho \times \sigma$ heatmap（叠加理论 $\rho^*(\sigma)$） | RQ4-a | 🆕 |
| F13 | $k/n \times \sigma$ heatmap（Sybil） | RQ4-b | 🆕 |
| F14 | 三类扰动的 unified 补偿曲线（归一化 attack severity × posterior quality） | RQ4 | 🆕 核心新图 |

**合计 14 张主图**；Appendix 放：扩展 sweep、heterogeneous quality 分解、best-response sanity check、长时段演化等。

---

## 7. 执行顺序

| 顺序 | 任务 | 类型 | 依赖 |
|---|---|---|---|
| 1 | 仿真代码：$q$ 分布改 T1-a | 修改 | — |
| 2 | 仿真代码：welfare 量纲改 T2-b | 修改 | — |
| 3 | 仿真代码：加 $\sigma$ 维度 | 新增 | — |
| 4 | 仿真代码：加 latent-factor mixture（$\rho$） | 新增 | — |
| 5 | 仿真代码：加 Sybil $k/n$ 采样 | 新增 | 4 后 |
| 6 | 仿真代码：加 heterogeneous $q_v$（$\kappa$） | 新增 | — |
| 7 | 跑 RQ1 全套 | 实验 | 1, 2 |
| 8 | 跑 RQ2 full grid | 实验 | 1, 2, 3 |
| 9 | 跑 RQ3 | 实验 | 1, 2 |
| 10 | 跑 RQ4-a / 4-b / 4-c | 实验 | 3, 4, 5, 6 |
| 11 | Writing 调整：T2-c, T2-d, T2-e | 写作 | 并行 |
| 12 | 出图 + 主文改写 | 写作 | 7–10 后 |

---

## 8. 不纳入本轮的改动

明确推迟到 Limitations / Future Work：
- **C-IR 独立验证**（Thm 3）：$\tau_s, V_c$ sweep
- **P-IR 双边区间完整验证**：$\tau_p, c_s$ 显式化
- **$m$ sweep**
- **$c_u$ 现实 anchor**（维持无量纲 effort–fee ratio）
- **Baselines 扩展**：centralized orchestrator / reputation-based 对照
- **完整 Sybl 理论**（voluntary collusion）：本轮只做简化 $k/n$ grid
- **统计显著性检验**：仅报 mean ± std

---

## 9. 相关文件索引

- 主文：`sections/GameTheoryAndCaseStudy.tex`
  - 理论（Sec 3.1）：L1–L266
  - 旧 case study（Sec 3.2）：L361–L562
  - 旧图环境：L269–L357（RQ1 两张）、L385–L447（RQ2/RQ3 两张）
- 独立 `sections/Experiments.tex`：待确认是否使用
- 图资源目录：`figures/fixed_sp/`, `figures/fixed_su/`, `figures/hard/`, `figures/quality/`
- 本方案文档：`experiment_plan.md`（当前文件）
