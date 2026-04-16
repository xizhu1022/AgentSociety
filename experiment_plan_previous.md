# DeQA 实验方案（Section 3.2）工作文档

> 记录目的：为后续讨论 / 重构 Section 3.2 实验部分提供基线快照。
> 源文件：`sections/GameTheoryAndCaseStudy.tex` L361–L562；相关图环境 L269–L357、L385–L447。

---

## 0. 整体定位

- Case Study 名称：**DeQA**（Decentralized Question Answering），作为 Sec 3.1 理论结果的实证验证载体。
- 目标：验证 Sec 3.1 理论的三个实证含义
  1. 理性 agents 在 tokenomics–knowledge dual loop 下收敛到 truthful 策略（对应 IC / IR）
  2. 验证机制改善 welfare，且表现依赖于 committee size $n$（对应 Thm 1 的 comparative statics）
  3. 机制放大真实质量，但无法弥补根本不可靠的 agents（对应 Thm 2–3 的 MLRP threshold）

- 三个 RQ：
  - **RQ1**（Rationality & Evolution）：agent 策略如何影响生态演化？
  - **RQ2**（Validation Mechanism）：validator 数量 $n$ 如何影响净 welfare？
  - **RQ3**（Service Quality）：服务质量与整体 welfare 的关系？

---

## 1. Instantiation：DeQA 映射

| 抽象角色 | QA 实例 |
|---|---|
| Consumer | 发 domain-specific query |
| Producer | 用 $a_u:\mathcal Q\to\mathcal R$ 回答 |
| Validator | 独立作答 + 与 producer 输出比对，投票形成 PoI consensus |
| Service quality $q$ | 产生正确答案的概率 |

---

## 2. Simulation Setting（七个部分）

### (1) Agents & Roles
- $N=10{,}000$ agents；每轮随机担任 consumer / producer / validator 之一
- Consumer 查询在领域上均匀分布
- Agent quality：$q \sim 0.98 + 0.02\cdot\text{Beta}(4,4)$，保证 $q_v>1/2$（对应 Assum 1）
- Beta(4,4) 选择理由：均值集中、保留有意义方差，建模"整体胜任但异质"的 population
- 随机作答（shirk）时 $q=1/2$，对应 Assum 2
- 初始 token：2{,}000；耗尽即退出 → 产生有限资源压力驱动 evolutionary dynamics

### (2) Validation Mechanism
- $n \in \{1,5,9,17,33,65\}$（奇数，Assum 1）
- $m = 10$ sample queries
- $\tau_\text{val} = 1$，stake $\sigma = 1$
- 满足 $c_u < \tau_\text{val}$（Assum 3）
- 多数投票达成 consensus
- 策略 $t_u\in[0,1]$：truthful participation rate
  - $t_u=1$ → 总是 full effort（$e=1$）
  - $t_u=0$ → 总是 shirk（$e=0$）
- Agents 最大化 ROC；低 ROC agent 从高 ROC agent 学习策略
- 初始策略：$t_u \sim \text{Uniform}(0,1)$（"cold start"）

### (3) Promotion & Subscription
- Producer effort cost $c_u \in (0,1)$ per validation query
- 成功订阅收益 $\tau_s = 1{,}000$
- 通过 $\tau_s/\tau_\text{val}=1000$ 保证 $mn\tau_\text{val}\ll\tau_s$（Assum 3）
- **未显式设定**：$\tau_p$, $c_s$

### (4) System Assumptions & Protocol
- Validators 独立非合谋（Assum 1）
- 所有参数有界
- 100{,}000 **interaction episodes**（每个 episode = 一次完整 promotion–validation–subscription cycle）
- 5 个独立随机种子；图上阴影 $\pm 1$ std

### (5) Evaluation Metrics
- **Truthful participation rate** $\bar t := \frac{1}{|\mathcal U_\text{active}|}\sum_u t_u$
- **ROC**：$\text{ROC}_u := (B_u^\text{final}-B_u^\text{init})/B_u^\text{init}$；分 consumer / producer / total
- **Social welfare** $W := \sum_u \text{ROC}_u$，另报 $W_c, W_p$
- **Posterior quality** $\mathbb E[q_p\mid Z=1]$

### (6) Rationality Parameters
- $S_p\in[0,1]$：理性（学习）agent 占比
- $S_u\in[0,1]$：理性 agent 每轮更新策略的概率（imitate 更高 ROC 的 peer）
- $S_p=S_u=1$ → 完全自适应；$S_p=0$ → frozen baseline

### (7) Baselines（参数化边界）
- **No-PoI**（$n=0$）：consumer 仅依赖 producer 自报 → "market for lemons"
- **Single-Validator**（$n=1$）：单一 validator，$P_\text{match}=q_v$，几乎无竞争压力

---

## 3. 三个 RQ 的当前分析与对应图

### RQ1 — Agent Rationality & Community Evolution（L508–523）
- 变量：$S_p, S_u$
- 现象：
  - $S_u=1, S_p=0$（frozen）：producer surplus > consumer surplus，总 welfare 低
  - 理性 producer 提升 $t_u$ → 个体 surplus↓ 但 consumer & total welfare↑
  - 低 ROC agent 模仿高 ROC → population 收敛到 honest participation
- 对应图：
  - **Fig `fig:roc_comp`**（L269–313）：2 行 × 6 列，ROC comparison
    - 上排：固定 $S_u=1$，$S_p\in\{0,0.2,0.4,0.6,0.8,1.0\}$
    - 下排：固定 $S_p=1$，$S_u\in\{0,0.2,0.4,0.6,0.8,1.0\}$
  - **Fig `fig:truthful_participation`**（L315–357）：同样 2×6 layout，truthful participation rate
- 理论连接：empirically 验证 dual loop；IC + self-screening

### RQ2 — Validation Mechanism Analysis（L525–541）
- 变量：$n$ + effort cost $c_u$
- 现象：
  - $n=1$：monopoly problem，pervasive shirking、低 posterior quality
  - $n\uparrow$：mutual supervision 出现，truthful 成为 dominant
  - Cost trade-off：$mn\tau_\text{val}$ 线性增长 vs. 信息边际递减 → 存在最优 $n$
  - 最优区间：$n\in\{5,9,17\}$
- 对应图：**Fig `fig:cost_validators`**（L385–416）
  - 5 个子图：Truthful participation / Posterior quality / Consumption ROC / Production ROC / Overall ROC
  - 横轴：effort cost；曲线按 $n$ 分组

### RQ3 — Service Quality Impacts（L544–561）
- 变量：service quality $q$
- 现象：
  - $q\approx 0.5$：validation 偏离真值，consumer 亏损；producer 越发 idle；总 welfare 为负
  - $q\uparrow$：正反馈循环（$q\uparrow\Rightarrow V(q_p)\uparrow\Rightarrow$ revenue $\uparrow\Rightarrow$ incentive $\uparrow$）
- 对应图：**Fig `fig:quality_impact`**（L418–447）
  - 6 子图：Real ROC − Expected ROC / Truthful participation / Overall quality / Consumption ROC / Production ROC / Overall ROC
  - 横轴：$q$
- Core insight：**机制揭示并奖励已有能力，而非凭空创造能力**

---

## 4. 当前方案与理论的覆盖矩阵

### 4.1 Assumptions 覆盖

| Assumption | 覆盖 | 备注 |
|---|---|---|
| Assum 1 独立 + $q_v>1/2$ + $n$ 奇 | ⚠ 部分 | architectural heterogeneity 未建模 |
| Assum 2 binary signal | ✓ | |
| Assum 3 cost structure | ⚠ 部分 | $\tau_p, c_s$ 缺失；$\tau_p$ 双边区间无法验证 |
| Assum 4 MLRP | ✓ 自动 | |

### 4.2 Theorems 验证

| 结论 | 验证程度 | 备注 |
|---|---|---|
| Thm 1 V-IC + $P_\text{match}$ 单调 | ✓ 间接 | 仅 sweep $n, c_u$；$\sigma, \tau_\text{val}$ 未 sweep |
| Thm 2 P-IR threshold $\bar q_p$ | ✗ | $q\in[0.98,1]$ 太窄，self-screening 未显现 |
| Thm 3 C-IR threshold | ✗ | 无 $\tau_s, V_c$ sweep |
| Thm 4 BNE 收敛 | ✓ 间接 | RQ1 收敛动力学 |
| Thm 5 Sybil IC | ✗ | 无实验 |
| Sec 3.1.6(B) 残余 correlation $\rho$ | ✗ | 无实验 |
| Sec 3.1.6(C) heterogeneous quality | △ | Beta(4,4) 隐含，无隔离对照 |
| "stake $\sigma$ 是 universal lever" | ✗ | **$\sigma$ 全程固定为 1** |

---

## 5. 已识别的改进方向（待讨论清单）

1. 补 robustness 实验（Sybil、correlation、stake compensation）
2. 扩大 $q$ 分布以暴露 producer self-screening
3. 独立 sweep $\sigma, \tau_\text{val}$，验证 IC 不等式两个杠杆
4. 区分"能力不足"与"策略性偷懒"（RQ3 语义混淆）
5. Welfare 定义量纲问题：ROC 求和 vs. token 量
6. 直接展示 adverse selection spiral（population turnover, Gini）
7. Roles 随机重分配 vs. 理论 fixed type
8. 学习规则单一（仅 imitation），缺 robustness 测试
9. $m$ 固定为 10，未 sweep
10. Baselines 偏弱（缺中心化 orchestrator、无 stake、reputation-based）
11. 统计显著性 / 置信区间不足
12. Posterior quality 未出现在 RQ1

---

## 6. 相关文件索引

- 主文：`sections/GameTheoryAndCaseStudy.tex`
  - 理论：L1–L266
  - Case study：L361–L562
  - 图环境：L269–L357（RQ1 两张图）、L385–L447（RQ2/RQ3 两张图）
- 独立 `sections/Experiments.tex`（如存在）：待确认关系
- 图资源目录：`figures/fixed_sp/`, `figures/fixed_su/`, `figures/hard/`, `figures/quality/`
