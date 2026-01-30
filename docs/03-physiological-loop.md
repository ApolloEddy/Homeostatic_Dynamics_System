# L1 数字生理循环

> **Homeostatic Dynamics System** 的第一层——数字生理循环

---

## 章节导航

**[返回目录](./INDEX.md)** | **[上一篇：系统架构总览](./02-system-architecture.md)** | **[下一篇：L2 神经接口与编译器](./04-neural-interface.md)**

---

## 1. 概述

### 1.1 定位

L1 数字生理循环是 April 的"潜意识与生命体征"，由独立于对话回合的 time-step 驱动。它负责管理内驱力变量（Homeostasis）和神经递质系统（Neuromodulators）的状态更新。

### 1.2 核心职责

L1 数字生理循环的核心职责包括：

- 管理内驱力变量（$H_{social}$、$H_{energy}$）的衰减与恢复
- 执行神经递质系统（DA、NE、5HT）的 OU 过程积分
- 处理外界刺激输入的归一化与限幅
- 执行时间步进（tick）

### 1.3 驱动方式

L1 数字生理循环的驱动方式：

- 独立于对话回合的 time-step 驱动
- 可 $w_{tick}$ 秒 / tick，或按回合 tick
- 支持空闲时补 tick

---

## 2. 内驱力系统（Homeostasis）

### 2.1 设计原理

内驱力是生物体维持生存的基本需求驱动。在 HDS 中，内驱力变量归一化到区间 $[0,1]$，采用软饱和而非硬截断，避免贴边后策略失真。

### 2.2 核心内驱力变量

#### 2.2.1 $H_{social}$（社交饱腹感）

**生物学基础**：
社交需求是人类的基本需求之一，缺乏社交会导致孤独感、抑郁等问题。社交饱腹感反映个体对社交互动的满足程度。

**动态特性**：
- 时间衰减：孤独积累
- 高质量互动回升

**行为影响**：
- 主动性
- 亲近表达上限

**数学模型**：

$$
H_{social}(t+\Delta t) = \sigma_{soft}(H_{social}(t) - d_s \Delta t + w_{quality} \cdot quality)
$$

其中：
- $d_s$：社交需求衰减率（每小时）
- $w_{quality}$：互动质量权重
- $quality$：互动质量 $[0,1]$
- $\sigma_{soft}$：软饱和函数

**参数解释**：
- $d_s$：社交需求自然衰减率，默认值 $w_{d_s}$
- $w_{quality}$：互动质量对社交饱腹感的提升权重，默认值 $w_{w_q}$

#### 2.2.2 $H_{energy}$（认知能量）

**生物学基础**：
认知能量反映个体的认知资源可用性。认知资源是有限的，复杂的推理、工具调用、长文本生成会消耗认知能量，而空闲和睡眠可以恢复认知能量。

**动态特性**：
- 随推理/工具调用/长文本生成消耗
- 随空闲/睡眠恢复

**行为影响**：
- 配合度
- 解释深度
- 拒绝阈值

**数学模型**：

$$
H_{energy}(t+\Delta t) = \sigma_{soft}(H_{energy}(t) + r_e \Delta t - c_e \cdot complexity)
$$

其中：
- $r_e$：认知能量恢复率（每小时）
- $c_e$：基础任务消耗
- $complexity$：任务复杂度倍率

**参数解释**：
- $r_e$：认知能量自然恢复率，默认值 $w_{r_e}$
- $c_e$：基础任务消耗，默认值 $w_{c_e}$

### 2.3 软饱和函数

为了避免硬边界导致策略失真，HDS 使用 sigmoid 函数实现软饱和：

$$
\sigma_{soft}(x) = \frac{1}{1 + e^{-k(x - w_{midpoint})}}
$$

其中：
- $k$：压缩系数
- $x$：输入值
- $w_{midpoint}$：中点值，默认值 $w_{mp}$

**参数解释**：
- $k$：压缩系数，默认值 $w_k$
- $w_{midpoint}$：中点值，控制 sigmoid 函数的中心位置

**特性**：
- 将输入值压缩到 $(0,1)$ 区间
- 在 $x = w_{midpoint}$ 处斜率最大
- 避免硬截断导致的策略失真

### 2.4 可选扩展内驱力

除了核心内驱力变量，HDS 还支持可选扩展：

#### 2.4.1 $H_{safety}$（安全感）

**生物学基础**：
安全感是个体对自身安全和稳定的主观感受。受攻击/越界触发下降，影响防御性与边界力度。

**动态特性**：
- 受攻击/越界触发下降
- 随时间缓慢恢复

**行为影响**：
- 防御性
- 边界力度

#### 2.4.2 $H_{novelty}$（新鲜感）

**生物学基础**：
新鲜感反映个体对新奇事物的兴趣程度。长期重复主题下降，影响探索欲、提问欲。

**动态特性**：
- 长期重复主题下降
- 新奇刺激回升

**行为影响**：
- 探索欲
- 提问欲

---

## 3. 神经递质系统（Neuromodulators）

### 3.1 设计原理

神经递质遵循 Ornstein–Uhlenbeck（OU）随机过程，具备惯性和回归特性。OU 过程是具有均值回归特性的随机过程，适合模拟情绪的惯性和波动。

### 3.2 OU 随机过程

#### 3.2.1 数学模型

OU 过程的随机微分方程：

$$
dX_t = \theta(\mu - X_t)dt + \sigma dW_t + I_{stimulus}
$$

其中：
- $\mu$：人格/基线（慢变量）
- $\theta$：回归速率（弹性）
- $\sigma$：噪声强度（个体差异/波动）
- $W_t$：维纳过程（布朗运动）
- $I_{stimulus}$：外界刺激输入（归一化与限幅）

**参数解释**：
- $\mu$：神经递质的基线水平，反映人格特质
- $\theta$：回归速率，控制状态回归基线的速度
- $\sigma$：噪声强度，控制状态的波动程度
- $I_{stimulus}$：外界刺激输入，归一化到 $[-I_{max}, I_{max}]$

#### 3.2.2 离散化实现

OU 过程的离散化形式（欧拉-丸山方法）：

$$
X_{t+\Delta t} = X_t + \theta(\mu - X_t)\Delta t + \sigma\sqrt{\Delta t}\,\epsilon + I_{stimulus}
$$

其中：
- $\Delta t$：时间步长
- $\epsilon \sim \mathcal{N}(0,1)$：标准正态随机变量

#### 3.2.3 稳定性分析

OU 过程具有以下稳定性特性：

**均值回归**：

$$
\lim_{t \to \infty} \mathbb{E}[X_t] = \mu
$$

**方差收敛**：

$$
\lim_{t \to \infty} \text{Var}(X_t) = \frac{\sigma^2}{2\theta}
$$

**平稳分布**：

$$
X_t \sim \mathcal{N}\left(\mu, \frac{\sigma^2}{2\theta}\right)
$$

### 3.3 核心神经递质

#### 3.3.1 $P_{DA}$（Dopamine）

**生物学功能**：
- 奖赏预测误差（RPE）驱动的动机/学习率调制
- 探索行为驱动
- 工作记忆和注意力调节

**HDS 映射**：
- $P_{DA}$ 高：动机增强、探索欲增加、学习率提高
- $P_{DA}$ 低：动机减弱、保守倾向、重复行为

**数学模型**：

$$
P_{DA}(t+\Delta t) = P_{DA}(t) + \theta_{DA}(\mu_{DA} - P_{DA}(t))\Delta t + \sigma_{DA}\sqrt{\Delta t}\,\epsilon_{DA} + I_{DA}
$$

**参数解释**：
- $\mu_{DA}$：多巴胺基线水平，默认值 $w_{\mu_{DA}}$
- $\theta_{DA}$：多巴胺回归速率，默认值 $w_{\theta_{DA}}$
- $\sigma_{DA}$：多巴胺噪声强度，默认值 $w_{\sigma_{DA}}$
- $I_{DA}$：多巴胺刺激输入，归一化到 $[-I_{max}, I_{max}]$

#### 3.3.2 $P_{NE}$（Norepinephrine）

**生物学功能**：
- 唤醒和注意力调节
- 压力反应和警觉性
- 认知灵活性
- 情绪强度调制

**HDS 映射**：
- $P_{NE}$ 高：注意力集中、防御性增强、直率表达
- $P_{NE}$ 低：注意力分散、放松状态、委婉表达

**数学模型**：

$$
P_{NE}(t+\Delta t) = P_{NE}(t) + \theta_{NE}(\mu_{NE} - P_{NE}(t))\Delta t + \sigma_{NE}\sqrt{\Delta t}\,\epsilon_{NE} + I_{NE}
$$

**参数解释**：
- $\mu_{NE}$：去甲肾上腺素基线水平，默认值 $w_{\mu_{NE}}$
- $\theta_{NE}$：去甲肾上腺素回归速率，默认值 $w_{\theta_{NE}}$
- $\sigma_{NE}$：去甲肾上腺素噪声强度，默认值 $w_{\sigma_{NE}}$
- $I_{NE}$：去甲肾上腺素刺激输入，归一化到 $[-I_{max}, I_{max}]$

#### 3.3.3 $P_{5HT}$（Serotonin）

**生物学功能**：
- 情绪稳定性和抑制控制
- 长期规划和决策
- 社会行为调节
- 冲动控制

**HDS 映射**：
- $P_{5HT}$ 高：情绪稳定、控制力强、边界清晰
- $P_{5HT}$ 低：情绪波动、控制力弱、边界模糊

**数学模型**：

$$
P_{5HT}(t+\Delta t) = P_{5HT}(t) + \theta_{5HT}(\mu_{5HT} - P_{5HT}(t))\Delta t + \sigma_{5HT}\sqrt{\Delta t}\,\epsilon_{5HT} + I_{5HT}
$$

**参数解释**：
- $\mu_{5HT}$：血清素基线水平，默认值 $w_{\mu_{5HT}}$
- $\theta_{5HT}$：血清素回归速率，默认值 $w_{\theta_{5HT}}$
- $\sigma_{5HT}$：血清素噪声强度，默认值 $w_{\sigma_{5HT}}$
- $I_{5HT}$：血清素刺激输入，归一化到 $[-I_{max}, I_{max}]$

---

## 4. 刺激输入处理

### 4.1 工程约束

为了避免"爆炸/贴边/长期失真"，必须对刺激做三件事：

1. **归一化**：把各种事件映射到统一尺度（例如 $[-1,1]$）
2. **裁剪/压缩**：建议 $I = \tanh(raw / s)$ 或 winsorize
3. **双时间常数**：短期情绪回落 + 中期心境回落（两套 OU 或 OU+低通）

### 4.2 刺激归一化

#### 4.2.1 归一化策略

HDS 使用以下归一化策略：

$$
I_{normalized} = \tanh\left(\frac{raw}{s_{stimulus}}\right)
$$

其中：
- $raw$：原始刺激值
- $s_{stimulus}$：归一化尺度

**参数解释**：
- $s_{stimulus}$：刺激归一化尺度，默认值 $w_{s_{stimulus}}$

#### 4.2.2 刺激限幅

归一化后，对刺激进行限幅：

$$
I_{clamped} = \text{clamp}(I_{normalized}, -I_{max}, I_{max})
$$

其中：
- $I_{max}$：刺激最大幅度

**参数解释**：
- $I_{max}$：刺激最大幅度，默认值 $w_{I_{max}}$

### 4.3 感知刺激映射

#### 4.3.1 冒犯性到 NE 的映射

冒犯性（offensiveness）反映用户输入的攻击性程度，映射到去甲肾上腺素（NE）刺激：

$$
I_{NE} = \frac{offensiveness - w_{offense,center}}{w_{offense,scale}} \cdot w_{offense,weight}
$$

其中：
- $offensiveness$：冒犯性评分 $[0,10]$
- $w_{offense,center}$：冒犯性中心值，默认值 $w_{w_{oc}}$
- $w_{offense,scale}$：冒犯性尺度，默认值 $w_{w_{os}}$
- $w_{offense,weight}$：冒犯性权重，默认值 $w_{w_{ow}}$

**参数解释**：
- $w_{offense,center}$：冒犯性中心值，用于将冒犯性评分归一化到 $[-1,1]$
- $w_{offense,scale}$：冒犯性尺度，控制冒犯性对 NE 的影响程度
- $w_{offense,weight}$：冒犯性权重，控制冒犯性在刺激中的占比

#### 4.3.2 效价到 DA 的映射

效价（valence）反映用户输入的情感倾向，映射到多巴胺（DA）刺激：

$$
I_{DA} = valence \cdot w_{valence,weight}
$$

其中：
- $valence$：效价 $[-1,1]$
- $w_{valence,weight}$：效价权重，默认值 $w_{w_{vw}}$

**参数解释**：
- $w_{valence,weight}$：效价权重，控制效价在刺激中的占比

#### 4.3.3 边界设定到 5HT 的映射

边界设定（boundary setting）反映用户是否设定了边界，映射到血清素（5HT）刺激：

$$
I_{5HT} = \begin{cases}
p_{boundary}, & \text{if } isBoundarySetting \\
0, & \text{otherwise}
\end{cases}
$$

其中：
- $isBoundarySetting$：是否边界设定
- $p_{boundary}$：边界设定惩罚，默认值 $w_{p_b}$

**参数解释**：
- $p_{boundary}$：边界设定惩罚，控制边界设定对 5HT 的影响程度

### 4.4 双时间常数

为了避免情绪过度波动，HDS 使用双时间常数：

#### 4.4.1 短期情绪回落

短期情绪回落使用较大的回归速率 $\theta_{fast}$：

$$
X_{fast}(t+\Delta t) = X_{fast}(t) + \theta_{fast}(\mu - X_{fast}(t))\Delta t + \sigma\sqrt{\Delta t}\,\epsilon_{fast} + I_{stimulus}
$$

**参数解释**：
- $\theta_{fast}$：短期回归速率，默认值 $w_{\theta_{fast}}$

#### 4.4.2 中期心境回落

中期心境回落使用较小的回归速率 $\theta_{slow}$：

$$
X_{slow}(t+\Delta t) = X_{slow}(t) + \theta_{slow}(\mu - X_{slow}(t))\Delta t + \sigma\sqrt{\Delta t}\,\epsilon_{slow} + I_{stimulus}
$$

**参数解释**：
- $\theta_{slow}$：中期回归速率，默认值 $w_{\theta_{slow}}$

#### 4.4.3 加权组合

最终状态是短期和中期状态的加权组合：

$$
X_t = w_{fast} \cdot X_{fast}(t) + (1 - w_{fast}) \cdot X_{slow}(t)
$$

**参数解释**：
- $w_{fast}$：短期状态权重，默认值 $w_{w_f}$

---

## 5. 任务消耗模型

### 5.1 消耗拆分

HDS 将认知能量消耗拆分为可解释项：

#### 5.1.1 推理复杂度消耗

推理复杂度消耗（$cost_{reasoning}$）：

$$
cost_{reasoning} = c_{reasoning} \cdot complexity_{reasoning}
$$

其中：
- $c_{reasoning}$：推理复杂度消耗系数，默认值 $w_{c_r}$
- $complexity_{reasoning}$：推理复杂度 $[0,1]$

**参数解释**：
- $c_{reasoning}$：推理复杂度消耗系数，控制推理对认知能量的消耗程度

#### 5.1.2 输出长度消耗

输出长度消耗（$cost_{output}$）：

$$
cost_{output} = c_{output} \cdot \frac{tokens}{tokens_{max}}
$$

其中：
- $c_{output}$：输出长度消耗系数，默认值 $w_{c_o}$
- $tokens$：输出 token 数
- $tokens_{max}$：最大输出 token 数，默认值 $w_{t_m}$

**参数解释**：
- $c_{output}$：输出长度消耗系数，控制输出长度对认知能量的消耗程度
- $tokens_{max}$：最大输出 token 数，用于归一化输出长度

#### 5.1.3 工具调用消耗

工具调用消耗（$cost_{tools}$）：

$$
cost_{tools} = c_{tools} \cdot n_{tools}
$$

其中：
- $c_{tools}$：工具调用消耗系数，默认值 $w_{c_t}$
- $n_{tools}$：工具调用次数

**参数解释**：
- $c_{tools}$：工具调用消耗系数，控制工具调用对认知能量的消耗程度

### 5.2 恢复机制

#### 5.2.1 空闲恢复

空闲恢复（$recover_{idle}$）：

$$
recover_{idle} = r_{idle} \cdot \Delta t
$$

其中：
- $r_{idle}$：空闲恢复率，默认值 $w_{r_i}$
- $\Delta t$：空闲时间

**参数解释**：
- $r_{idle}$：空闲恢复率，控制空闲时认知能量的恢复速度

#### 5.2.2 睡眠恢复

睡眠恢复（$recover_{sleep}$）：

$$
recover_{sleep} = r_{sleep} \cdot \Delta t_{sleep}
$$

其中：
- $r_{sleep}$：睡眠恢复率，默认值 $w_{r_s}$
- $\Delta t_{sleep}$：睡眠时间

**参数解释**：
- $r_{sleep}$：睡眠恢复率，控制睡眠时认知能量的恢复速度

---

## 6. 实现细节

### 6.1 时间步进

#### 6.1.1 Tick 策略

HDS 的 Tick 策略：

- 按回合 tick + 空闲时补 tick
- 避免后台常驻耗电/耗资源

#### 6.1.2 Tick 实现

Tick 实现逻辑：

1. 将时间间隔转换为小时数
2. 如果时间间隔小于阈值，则不处理
3. 计算社交饱腹感自然衰减（孤独积累）
4. 计算认知能量自然恢复（空闲时）
5. 更新最后 tick 时间戳
6. 保存状态

**关键逻辑**：
- 社交饱腹感：$H_{social}(t+\Delta t) = H_{social}(t) - d_s \Delta t + w_{quality} \cdot quality$
- 认知能量：$H_{energy}(t+\Delta t) = H_{energy}(t) + r_e \Delta t - c_e \cdot complexity$
- 时间阈值：$w_{tick\_threshold}$（默认值 $w_{w_{tt}}$）

**参数解释**：
- $w_{tick\_threshold}$：时间阈值，默认值 $w_{w_{tt}}$

### 6.2 状态持久化

#### 6.2.1 存储策略

HDS 使用 SharedPreferences 实现状态持久化：

- 本地存储
- 可选加密
- 自动保存

#### 6.2.2 存储结构

存储结构示例（使用伪变量）：

```json
{
  "social": $w_{h\_social\_example}$,
  "energy": $w_{h\_energy\_example}$,
  "da": $w_{da\_example}$,
  "ne": $w_{ne\_example}$,
  "ht": $w_{ht\_example}$,
  "lastTick": "$timestamp\_example$"
}
```

**参数解释**：
- $w_{h\_social\_example}$：社交饱腹感示例值
- $w_{h\_energy\_example}$：认知能量示例值
- $w_{da\_example}$：多巴胺示例值
- $w_{ne\_example}$：去甲肾上腺素示例值
- $w_{ht\_example}$：血清素示例值
- $w_{timestamp\_example}$：时间戳示例值

### 6.3 初始化策略

#### 6.3.1 首次初始化

首次初始化时，HDS 使用默认值：

- 加载配置文件
- 创建内驱力引擎
- 创建神经递质系统
- 创建刺激归一化器
- 应用自上次 tick 以来的衰减

#### 6.3.2 后续启动

后续启动时，HDS 从持久化存储加载状态：

- 加载存储的状态
- 计算自上次 tick 以来的时间间隔
- 应用时间衰减

---

## 7. 参数默认值

### 7.1 内驱力参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $d_s$ | $w_{d_s}$ | 社交需求衰减率（每小时） |
| $r_e$ | $w_{r_e}$ | 认知能量恢复率（每小时） |
| $c_e$ | $w_{c_e}$ | 基础任务消耗 |
| $k$ | $w_k$ | 软饱和压缩系数 |
| $w_{midpoint}$ | $w_{mp}$ | sigmoid 中点值 |

### 7.2 神经递质参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $\mu_{DA}$ | $w_{\mu_{DA}}$ | 多巴胺基线水平 |
| $\theta_{DA}$ | $w_{\theta_{DA}}$ | 多巴胺回归速率 |
| $\sigma_{DA}$ | $w_{\sigma_{DA}}$ | 多巴胺噪声强度 |
| $\mu_{NE}$ | $w_{\mu_{NE}}$ | 去甲肾上腺素基线水平 |
| $\theta_{NE}$ | $w_{\theta_{NE}}$ | 去甲肾上腺素回归速率 |
| $\sigma_{NE}$ | $w_{\sigma_{NE}}$ | 去甲肾上腺素噪声强度 |
| $\mu_{5HT}$ | $w_{\mu_{5HT}}$ | 血清素基线水平 |
| $\theta_{5HT}$ | $w_{\theta_{5HT}}$ | 血清素回归速率 |
| $\sigma_{5HT}$ | $w_{\sigma_{5HT}}$ | 血清素噪声强度 |

### 7.3 刺激参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $s_{stimulus}$ | $w_{s_{stimulus}}$ | 刺激归一化尺度 |
| $I_{max}$ | $w_{I_{max}}$ | 刺激最大幅度 |
| $w_{offense,center}$ | $w_{w_{oc}}$ | 冒犯性中心值 |
| $w_{offense,scale}$ | $w_{w_{os}}$ | 冒犯性尺度 |
| $w_{offense,weight}$ | $w_{w_{ow}}$ | 冒犯性权重 |
| $w_{valence,weight}$ | $w_{w_{vw}}$ | 效价权重 |
| $p_{boundary}$ | $w_{p_b}$ | 边界设定惩罚 |

### 7.4 双时间常数参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $\theta_{fast}$ | $w_{\theta_{fast}}$ | 短期回归速率 |
| $\theta_{slow}$ | $w_{\theta_{slow}}$ | 中期回归速率 |
| $w_{fast}$ | $w_{w_f}$ | 短期状态权重 |

### 7.5 任务消耗参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $c_{reasoning}$ | $w_{c_r}$ | 推理复杂度消耗系数 |
| $c_{output}$ | $w_{c_o}$ | 输出长度消耗系数 |
| $c_{tools}$ | $w_{c_t}$ | 工具调用消耗系数 |
| $tokens_{max}$ | $w_{t_m}$ | 最大输出 token 数 |
| $r_{idle}$ | $w_{r_i}$ | 空闲恢复率 |
| $r_{sleep}$ | $w_{r_s}$ | 睡眠恢复率 |

### 7.6 Tick 参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{tick\_threshold}$ | $w_{w_{tt}}$ | 时间阈值 |

---

## 8. 总结

L1 数字生理循环是 HDS 的基础层，负责管理内驱力变量和神经递质系统的状态更新。通过生物启发的数学模型和工程化的实现细节，L1 实现了一个稳定、可控、可解释的数字生理系统。

### 8.1 核心特性

| 特性 | 体现 |
| :--- | :--- |
| **生物启发** | 内驱力理论、神经递质调节 |
| **数学严谨** | OU 过程、稳定性分析 |
| **工程可行** | 软饱和、归一化、限幅 |
| **可观测性** | 状态快照、调试信息 |

### 8.2 设计原则

| 原则 | 体现 |
| :--- | :--- |
| **连续性** | 时间步进、状态惯性 |
| **可控性** | 参数化、限幅保护 |
| **可解释性** | 状态快照、决策理由 |
| **稳定性** | 均值回归、软饱和 |

---

**[返回目录](./INDEX.md)** | **[上一篇：系统架构总览](./02-system-architecture.md)** | **[下一篇：L2 神经接口与编译器](./04-neural-interface.md)**
