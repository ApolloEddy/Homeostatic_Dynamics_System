# L2 神经接口与编译器

> **Homeostatic Dynamics System** 的第二层——神经接口与编译器

---

## 章节导航

**[返回目录](./INDEX.md)** | **[上一篇：L1 数字生理循环](./03-physiological-loop.md)** | **[下一篇：L3 记忆巩固](./05-consolidation.md)**

---

## 1. 概述

### 1.1 定位

L2 神经接口与编译器是连接数值系统与商业 LLM API 的桥梁。它的核心原则是：**采样参数只做粗调**，语义级人格与"失控表现"主要靠"风格编译器"控制。

### 1.2 核心职责

L2 神经接口与编译器的核心职责包括：

- 将生理状态翻译为行为向量
- 编译动态系统指令
- 生成采样参数建议
- 计算检索偏置权重

### 1.3 核心原则

L2 神经接口与编译器遵循以下核心原则：

- **采样参数只做粗调**：避免将"人格"变成"胡言乱语/重复"
- **语义级人格主要靠编译器控制**：通过风格指令实现"失控表现"
- **限幅保护**：所有映射关系均设置上下限

---

## 2. 行为向量（Behavior Vector）

### 2.1 设计原理

行为向量是 HDS 与 LLM 之间的稳定中间层，$B \in [B_{\min}, B_{\max}]^{k}$。它将复杂的生理状态映射为可解释的行为特征。

### 2.2 行为向量维度

| 维度 | 名称 | 含义 | 取值范围 |
| :--- | :--- | :--- | :--- |
| $b_1$ | initiative | 主动性 | $[B_{\min}, B_{\max}]$ |
| $b_2$ | warmth | 温暖/亲和 | $[B_{\min}, B_{\max}]$ |
| $b_3$ | patience | 耐心/解释意愿 | $[B_{\min}, B_{\max}]$ |
| $b_4$ | verbosity | 话量 | $[B_{\min}, B_{\max}]$ |
| $b_5$ | directness | 直率/委婉 | $[B_{\min}, B_{\max}]$ |
| $b_6$ | defensiveness | 防御性 | $[B_{\min}, B_{\max}]$ |
| $b_7$ | curiosity | 提问/探索 | $[B_{\min}, B_{\max}]$ |
| $b_8$ | boundary_strength | 边界力度 | $[B_{\min}, B_{\max}]$ |

### 2.3 状态到行为向量的映射

#### 2.3.1 $H_{social}$ 的映射

$H_{social}$（社交饱腹感）影响主动性和温暖度：

$$
b_1 = \text{lerp}(b_{1,low}, b_{1,high}, H_{social})
$$

$$
b_2 = \text{lerp}(b_{2,low}, b_{2,high}, H_{social})
$$

其中：
- $\text{lerp}(a, b, t) = a + (b - a) \cdot t$：线性插值
- $b_{1,low}$、$b_{1,high}$：主动性的上下限
- $b_{2,low}$、$b_{2,high}$：温暖度的上下限

**参数解释**：
- $b_{1,low}$：主动性下限（不展示真实数值）
- $b_{1,high}$：主动性上限（不展示真实数值）
- $b_{2,low}$：温暖度下限（不展示真实数值）
- $b_{2,high}$：温暖度上限（不展示真实数值）

**映射逻辑**：
- $H_{social}$ 低 $\rightarrow$ $b_1 \uparrow$、$b_2 \uparrow$（更主动、更温暖）
- $H_{social}$ 高 $\rightarrow$ $b_1 \downarrow$、$b_2 \downarrow$（更独立、更冷淡）

#### 2.3.2 $H_{energy}$ 的映射

$H_{energy}$（认知能量）影响耐心度和话量：

$$
b_3 = \text{lerp}(b_{3,low}, b_{3,high}, H_{energy})
$$

$$
b_4 = \text{lerp}(b_{4,low}, b_{4,high}, H_{energy})
$$

其中：
- $b_{3,low}$、$b_{3,high}$：耐心度的上下限
- $b_{4,low}$、$b_{4,high}$：话量的上下限

**参数解释**：
- $b_{3,low}$：耐心度下限（不展示真实数值）
- $b_{3,high}$：耐心度上限（不展示真实数值）
- $b_{4,low}$：话量下限（不展示真实数值）
- $b_{4,high}$：话量上限（不展示真实数值）

**映射逻辑**：
- $H_{energy}$ 低 $\rightarrow$ $b_3 \downarrow$、$b_4 \downarrow$（更不耐烦、更简洁）
- $H_{energy}$ 高 $\rightarrow$ $b_3 \uparrow$、$b_4 \uparrow$（更有耐心、更详细）

#### 2.3.3 $P_{NE}$ 的映射

$P_{NE}$（去甲肾上腺素）影响直率和防御性：

$$
b_5 = \text{lerp}(b_{5,low}, b_{5,high}, P_{NE})
$$

$$
b_6 = \text{lerp}(b_{6,low}, b_{6,high}, P_{NE})
$$

其中：
- $b_{5,low}$、$b_{5,high}$：直率的上下限
- $b_{6,low}$、$b_{6,high}$：防御性的上下限

**参数解释**：
- $b_{5,low}$：直率下限（不展示真实数值）
- $b_{5,high}$：直率上限（不展示真实数值）
- $b_{6,low}$：防御性下限（不展示真实数值）
- $b_{6,high}$：防御性上限（不展示真实数值）

**映射逻辑**：
- $P_{NE}$ 高 $\rightarrow$ $b_5 \uparrow$、$b_6 \uparrow$（更直率、更防御）
- $P_{NE}$ 低 $\rightarrow$ $b_5 \downarrow$、$b_6 \downarrow$（更委婉、更开放）

#### 2.3.4 $P_{DA}$ 的映射

$P_{DA}$（多巴胺）影响好奇心：

$$
b_7 = \text{lerp}(b_{7,low}, b_{7,high}, P_{DA})
$$

其中：
- $b_{7,low}$、$b_{7,high}$：好奇心的上下限

**参数解释**：
- $b_{7,low}$：好奇心下限（不展示真实数值）
- $b_{7,high}$：好奇心上限（不展示真实数值）

**映射逻辑**：
- $P_{DA}$ 高 $\rightarrow$ $b_7 \uparrow$（更愿意探索、更愿意提问）
- $P_{DA}$ 低 $\rightarrow$ $b_7 \downarrow$（更保守、更少提问）

#### 2.3.5 $P_{5HT}$ 的映射

$P_{5HT}$（血清素）影响边界力度：

$$
b_8 = \text{lerp}(b_{8,low}, b_{8,high}, P_{5HT})
$$

其中：
- $b_{8,low}$、$b_{8,high}$：边界力度的上下限

**参数解释**：
- $b_{8,low}$：边界力度下限（不展示真实数值）
- $b_{8,high}$：边界力度上限（不展示真实数值）

**映射逻辑**：
- $P_{5HT}$ 高 $\rightarrow$ $b_8 \uparrow$（边界更清晰、拒绝更坚决）
- $P_{5HT}$ 低 $\rightarrow$ $b_8 \downarrow$（边界更模糊、拒绝更温和）

---

## 3. 动态系统指令编译

### 3.1 设计原理

动态系统指令编译将行为向量编译为**短小、可控、无主观叙事膨胀**的元指令，注入 system/developer 允许的区域。

### 3.2 注入长度预算

为了避免注意力污染，注入长度预算建议：

$$
tokens_{injection} \le w_{token\_budget}
$$

其中：
- $w_{token\_budget}$：注入 token 预算（不展示真实数值）

**参数解释**：
- $w_{token\_budget}$：注入 token 预算，控制注入指令的长度

### 3.3 注入格式

#### 3.3.1 状态注入格式

状态注入格式：

```
<hds_state>
initiative=$b_1$ warmth=$b_2$ patience=$b_3$
verbosity=$b_4$ directness=$b_5$ defensiveness=$b_6$
curiosity=$b_7$ boundary=$b_8$
</hds_state>
```

#### 3.3.2 风格注入格式

风格注入格式：

```
style: $style\_description$
```

其中：
- $style\_description$：风格描述字符串

### 3.4 风格描述生成

#### 3.4.1 风格描述规则

风格描述根据行为向量生成，使用伪变量表示阈值：

| 行为向量 | 风格描述 |
| :--- | :--- |
| $b_4 < w_{verbosity\_low}$ | concise |
| $b_4 > w_{verbosity\_high}$ | expanded |
| $b_2 > w_{warmth\_high}$ | warm |
| $b_2 < w_{warmth\_low}$ | cool |
| $b_6 > w_{defensiveness\_high}$ | guarded |
| $b_3 < w_{patience\_low}$ | low_patience |
| $b_1 > w_{initiative\_high}$ | proactive |

**参数解释**：
- $w_{verbosity\_low}$：话量低阈值（不展示真实数值）
- $w_{verbosity\_high}$：话量高阈值（不展示真实数值）
- $w_{warmth\_high}$：温暖度高阈值（不展示真实数值）
- $w_{warmth\_low}$：温暖度低阈值（不展示真实数值）
- $w_{defensiveness\_high}$：防御性高阈值（不展示真实数值）
- $w_{patience\_low}$：耐心度低阈值（不展示真实数值）
- $w_{initiative\_high}$：主动性高阈值（不展示真实数值）

---

## 4. 检索与注意力偏置

### 4.1 设计原理

检索与注意力偏置（Cognitive Filter）是安全版本的记忆检索机制。它通过调整检索权重，实现状态感知的记忆检索，避免"低 DA 忽略正面互动"导致关系崩坏。

### 4.2 记忆分类

HDS 将记忆分为四类：

| 类别 | 名称 | 描述 | 偏置策略 |
| :--- | :--- | :--- | :--- |
| **FactMemory** | 事实记忆 | 客观事实（可验证、低情绪） | 不偏置 |
| **RelationMemory** | 关系记忆 | 偏好、边界、承诺、禁忌 | 轻微偏置（$\pm w_{relation\_bias}$） |
| **AffectMemory** | 情绪记忆 | 触发器、情绪图谱、当时状态 | 可偏置（用于氛围一致性） |
| **StyleMemory** | 风格记忆 | 在某类状态下的表达模板/语气策略 | 可偏置（用于 few-shot） |

**参数解释**：
- $w_{relation\_bias}$：关系记忆偏置范围（不展示真实数值）

### 4.3 检索偏置计算

#### 4.3.1 基础检索分数

基础检索分数：

$$
score_{base} = sim \cdot w_{category}
$$

其中：
- $sim$：余弦相似度
- $w_{category}$：类别权重

#### 4.3.2 状态匹配分数

状态匹配分数：

$$
score_{state} = w_{state\_bias} \cdot (1 + e^{-age/h_{half\_life}} + |RPE| \cdot w_{rpe})
$$

其中：
- $age$：记忆年龄
- $h_{half\_life}$：半衰期
- $RPE$：奖赏预测误差
- $w_{state\_bias}$：状态偏置权重（不展示真实数值）
- $w_{rpe}$：RPE 权重（不展示真实数值）

**参数解释**：
- $h_{half\_life}$：记忆半衰期，控制记忆衰减速度
- $w_{state\_bias}$：状态偏置权重，控制状态对检索的影响
- $w_{rpe}$：RPE 权重，控制 RPE 对检索的影响

#### 4.3.3 最终检索分数

最终检索分数：

$$
score = score_{base} + score_{state}
$$

### 4.4 偏置权重范围

#### 4.4.1 事实记忆偏置

事实记忆不偏置：

$$
w_{fact} = w_{fact\_default}
$$

**参数解释**：
- $w_{fact\_default}$：事实记忆默认权重

#### 4.4.2 关系记忆偏置

关系记忆轻微偏置：

$$
w_{relation} = \text{clamp}(w_{relation\_base} + (P_{DA} - w_{DA\_midpoint}) \cdot w_{relation\_bias}, w_{relation\_base} - w_{relation\_bias}, w_{relation\_base} + w_{relation\_bias})
$$

**参数解释**：
- $w_{DA\_midpoint}$：多巴胺中点值
- $w_{relation\_base}$：关系记忆基础权重
- $w_{relation\_bias}$：关系记忆偏置范围

#### 4.4.3 情绪记忆偏置

情绪记忆可偏置：

$$
w_{affect} = \text{clamp}(w_{affect\_base} + (P_{DA} - w_{DA\_midpoint}) \cdot w_{affect\_bias}, w_{affect\_base} - w_{affect\_bias}, w_{affect\_base} + w_{affect\_bias})
$$

其中：
- $w_{DA\_midpoint}$：多巴胺中点值
- $w_{affect\_base}$：情绪记忆基础权重
- $w_{affect\_bias}$：情绪记忆偏置范围（不展示真实数值）

**参数解释**：
- $w_{DA\_midpoint}$：多巴胺中点值
- $w_{affect\_base}$：情绪记忆基础权重
- $w_{affect\_bias}$：情绪记忆偏置范围，控制情绪记忆的偏置程度

#### 4.4.4 风格记忆偏置

风格记忆可偏置：

$$
w_{style} = \text{clamp}(w_{style\_base} + (P_{DA} - w_{DA\_midpoint}) \cdot w_{style\_bias}, w_{style\_base} - w_{style\_bias}, w_{style\_base} + w_{style\_bias})
$$

**参数解释**：
- $w_{DA\_midpoint}$：多巴胺中点值
- $w_{style\_base}$：风格记忆基础权重
- $w_{style\_bias}$：风格记忆偏置范围

其中：
- $w_{style\_bias}$：风格记忆偏置范围（不展示真实数值）

**参数解释**：
- $w_{style\_bias}$：风格记忆偏置范围，控制风格记忆的偏置程度

---

## 5. 采样超参映射

### 5.1 设计原理

采样超参映射将神经递质状态映射为 LLM 采样参数。为了避免将"人格"变成"胡言乱语/重复"，采样参数只做粗调，且限幅。

### 5.2 温度（Temperature）映射

#### 5.2.1 映射公式

温度映射：

$$
T = \text{clamp}(T_{min}, T_{max}, T_{base} - w_{T,NE} \cdot P_{NE})
$$

其中：
- $T_{min}$：温度下限
- $T_{max}$：温度上限
- $T_{base}$：温度基线（不展示真实数值）
- $w_{T,NE}$：NE 对温度的影响权重（不展示真实数值）

**参数解释**：
- $T_{min}$：温度下限（不展示真实数值）
- $T_{max}$：温度上限（不展示真实数值）
- $T_{base}$：温度基线，控制默认温度
- $w_{T,NE}$：NE 对温度的影响权重，控制 NE 对温度的影响程度

**映射逻辑**：
- $P_{NE}$ 高 $\rightarrow$ $T \downarrow$（更保守、更确定）
- $P_{NE}$ 低 $\rightarrow$ $T \uparrow$（更发散、更随机）

#### 5.2.2 限幅保护

温度限幅：

$$
T \in [T_{min}, T_{max}]
$$

### 5.3 Top-P 映射

#### 5.3.1 映射公式

Top-P 映射：

$$
top\_p = \text{clamp}(p_{min}, p_{max}, p_{base} - w_{p,NE} \cdot P_{NE} + w_{p,5HT} \cdot P_{5HT})
$$

其中：
- $p_{min}$：Top-P 下限
- $p_{max}$：Top-P 上限
- $p_{base}$：Top-P 基线（不展示真实数值）
- $w_{p,NE}$：NE 对 Top-P 的影响权重（不展示真实数值）
- $w_{p,5HT}$：5HT 对 Top-P 的影响权重（不展示真实数值）

**参数解释**：
- $p_{min}$：Top-P 下限（不展示真实数值）
- $p_{max}$：Top-P 上限（不展示真实数值）
- $p_{base}$：Top-P 基线，控制默认 Top-P
- $w_{p,NE}$：NE 对 Top-P 的影响权重，控制 NE 对 Top-P 的影响程度
- $w_{p,5HT}$：5HT 对 Top-P 的影响权重，控制 5HT 对 Top-P 的影响程度

**映射逻辑**：
- $P_{NE}$ 高 $\rightarrow$ $top\_p \downarrow$（更保守、更确定）
- $P_{NE}$ 低 $\rightarrow$ $top\_p \uparrow$（更发散、更随机）
- $P_{5HT}$ 高 $\rightarrow$ $top\_p \uparrow$（更发散、更随机）
- $P_{5HT}$ 低 $\rightarrow$ $top\_p \downarrow$（更保守、更确定）

#### 5.3.2 限幅保护

Top-P 限幅：

$$
top\_p \in [p_{min}, p_{max}]
$$

### 5.4 最大 Token 数映射

#### 5.4.1 映射公式

最大 Token 数映射：

$$
max\_tokens = \text{clamp}(tokens_{min}, tokens_{max}, tokens_{base} \cdot f_{energy} \cdot f_{verbosity})
$$

其中：
- $tokens_{min}$：最小 Token 数
- $tokens_{max}$：最大 Token 数
- $tokens_{base}$：基础 Token 数（不展示真实数值）

#### 5.4.2 能量因子

能量因子：

$$
f_{energy} = w_{f_{e,base}} + w_{f_{e,scale}} \cdot H_{energy}
$$

其中：
- $w_{f_{e,base}}$：能量因子基线（不展示真实数值）
- $w_{f_{e,scale}}$：能量因子尺度（不展示真实数值）

**参数解释**：
- $w_{f_{e,base}}$：能量因子基线，控制基础能量因子
- $w_{f_{e,scale}}$：能量因子尺度，控制能量对 Token 数的影响程度

#### 5.4.3 话量因子

话量因子：

$$
f_{verbosity} = w_{f_{v,base}} + w_{f_{v,scale}} \cdot b_4
$$

其中：
- $w_{f_{v,base}}$：话量因子基线（不展示真实数值）
- $w_{f_{v,scale}}$：话量因子尺度（不展示真实数值）

**参数解释**：
- $w_{f_{v,base}}$：话量因子基线，控制基础话量因子
- $w_{f_{v,scale}}$：话量因子尺度，控制话量对 Token 数的影响程度

#### 5.4.4 限幅保护

最大 Token 数限幅：

$$
max\_tokens \in [tokens_{min}, tokens_{max}]
$$

---

## 6. 控制输出包（Control Bundle）

### 6.1 设计原理

控制输出包（Control Bundle）是 L2 的最终输出，包含所有上层 LLM/Agent 需要的控制信息。

### 6.2 输出结构

#### 6.2.1 控制输出包

控制输出包包含以下字段：

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `promptInjection` | String | 动态系统指令注入 |
| `sampling` | Object | 采样参数 |
| `sampling.temperature` | Number | 温度参数 |
| `sampling.topP` | Number | Top-P 参数 |
| `sampling.maxTokens` | Number | 最大 Token 数 |
| `retrievalBias` | Object | 检索偏置权重 |
| `retrievalBias.factMemory` | Number | 事实记忆偏置 |
| `retrievalBias.relationMemory` | Number | 关系记忆偏置 |
| `retrievalBias.affectMemory` | Number | 情绪记忆偏置 |
| `retrievalBias.styleMemory` | Number | 风格记忆偏置 |
| `behavior` | Object | 行为向量 |
| `styleGuidance` | String | 风格描述 |
| `debug` | Object | 调试信息 |
| `debug.stateSnapshot` | Object | 状态快照 |
| `debug.reasons` | Array | 决策理由 |

---

## 7. 参数符号表（不含真实数值）

> [!IMPORTANT]
> 本节只给出**符号与语义**，不提供任何来自代码/配置的真实参数值。  
> 代码块与表格规范见 `docs/STYLE.md`。

### 7.1 行为向量边界（BehaviorVector）

> [!NOTE]
> 为提升可读性，这里使用“语义化下标”而不是 `$w_{...}$` 形式的占位符。

<table>
  <tr>
    <td width="50%">
      <table>
        <tr><th>符号</th><th>含义与约束</th></tr>
        <tr><td>\(b_{\mathrm{initiative},\min}\)</td><td>主动性下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{initiative},\max}\)</td><td>主动性上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{warmth},\min}\)</td><td>温暖度下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{warmth},\max}\)</td><td>温暖度上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{patience},\min}\)</td><td>耐心下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{patience},\max}\)</td><td>耐心上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{verbosity},\min}\)</td><td>话量下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{verbosity},\max}\)</td><td>话量上界（有界）</td></tr>
      </table>
    </td>
    <td width="50%">
      <table>
        <tr><th>符号</th><th>含义与约束</th></tr>
        <tr><td>\(b_{\mathrm{directness},\min}\)</td><td>直率下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{directness},\max}\)</td><td>直率上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{defensiveness},\min}\)</td><td>防御性下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{defensiveness},\max}\)</td><td>防御性上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{curiosity},\min}\)</td><td>好奇心下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{curiosity},\max}\)</td><td>好奇心上界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{boundary},\min}\)</td><td>边界强度下界（有界）</td></tr>
        <tr><td>\(b_{\mathrm{boundary},\max}\)</td><td>边界强度上界（有界）</td></tr>
      </table>
    </td>
  </tr>
</table>

### 7.2 注入预算（Prompt Injection Budget）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(B_{\mathrm{inject}}\) | 注入预算（长度/Token 预算，以符号表示上界与策略） |

### 7.3 检索偏置（Retrieval Bias）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(w_{\mathrm{fact}}\) | 事实记忆权重（建议为常量或极窄范围） |
| \(w_{\mathrm{relation}}\) | 关系记忆权重（有界偏置） |
| \(w_{\mathrm{affect}}\) | 情绪记忆权重（有界偏置） |
| \(w_{\mathrm{style}}\) | 风格记忆权重（有界偏置） |
| \(h_{\mathrm{half}}\) | 时间衰减的半衰期/时间尺度（符号化） |
| \(w_{\mathrm{state}}\) | 状态匹配权重（符号化） |
| \(w_{\mathrm{rpe}}\) | 显著性/RPE 权重（符号化且必须有界） |

### 7.4 采样映射边界（Sampling Bounds）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(T_{\min}, T_{\max}\) | temperature 上下界（有界） |
| \(P_{\min}, P_{\max}\) | top-p 上下界（有界） |
| \(M_{\min}, M_{\max}\) | max tokens 上下界（有界） |
| \(\alpha_T,\beta_{T,NE}\) | NE 到 temperature 的线性映射系数（符号化） |
| \(\alpha_P,\beta_{P,NE},\beta_{P,5HT}\) | NE/5HT 到 top-p 的映射系数（符号化） |
| \(\alpha_M,\beta_{M,E},\beta_{M,V}\) | 能量/话量到长度预算的映射系数（符号化） |

### 7.5 风格阈值（离散风格标签）

> [!NOTE]
> 风格标签属于有限离散结果，允许直接列出全集（不涉及数值泄露）。

| 标签字段 | 允许取值（示例） |
| :--- | :--- |
| `verbosityStyle` | `concise` \| `expanded` \| `balanced` |
| `toneStyle` | `warm` \| `cool` \| `neutral` |
| `guardStyle` | `guarded` \| `open` |
| `patienceStyle` | `low_patience` \| `normal` |
| `initiativeStyle` | `proactive` \| `reactive` |

---

## 8. 总结

L2 神经接口与编译器是 HDS 的核心层，负责将生理状态翻译为可执行的控制指令。通过行为向量、动态指令编译、检索偏置和采样参数映射，L2 实现了一个稳定、可控、可解释的神经接口。

### 8.1 核心特性

| 特性 | 体现 |
| :--- | :--- |
| **采样参数只做粗调** | 避免将"人格"变成"胡言乱语/重复" |
| **语义级人格主要靠编译器控制** | 通过风格指令实现"失控表现" |
| **限幅保护** | 所有映射关系均设置上下限 |
| **可解释性** | 行为向量、风格描述、决策理由 |

### 8.2 设计原则

| 原则 | 体现 |
| :--- | :--- |
| **稳定性** | 限幅保护、软饱和 |
| **可控性** | 参数化、可调参 |
| **可解释性** | 状态快照、决策理由 |
| **安全性** | 检索偏置、避免关系崩坏 |

---

**[返回目录](./INDEX.md)** | **[上一篇：L1 数字生理循环](./03-physiological-loop.md)** | **[下一篇：L3 记忆巩固](./05-consolidation.md)**


