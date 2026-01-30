# L3 记忆巩固

> **Homeostatic Dynamics System** 的第三层——长时序人格固化

---

## 章节导航

**[返回目录](./INDEX.md)** | **[上一篇：L2 神经接口与编译器](./04-neural-interface.md)** | **[下一篇：数学模型详解](./06-mathematical-models.md)**

---

## 1. 概述

### 1.1 定位

L3 长时序人格固化的目标是：减少灾难性遗忘与人格漂移，同时避免"一次极端事件永久绑架人格"。

### 1.2 核心职责

L3 长时序人格固化的核心职责包括：

- 记录交互日志（State-Aware Logging）
- 分箱记忆存储（Binned Memory）
- 离线巩固处理（Sleep Consolidation）
- 慢变量更新（带限幅 + 可复原）

### 1.3 触发条件

L3 的触发条件：

- 闲置/夜间/对话间隔达到阈值
- 手动触发

---

## 2. 日志结构（State-Aware Logging）

### 2.1 设计原理

每轮交互记录"说了什么 + 当时状态 + 事件标签"，为后续巩固提供完整上下文。

### 2.2 日志结构

#### 2.2.1 交互日志格式

交互日志格式示例（使用伪变量）：

```json
{
  "ts": "$timestamp\_example$",
  "user": "用户输入内容",
  "assistant": "系统响应内容",
  "state": {
    "hSocial": $w_{h\_social\_example}$,
    "hEnergy": $w_{h\_energy\_example}$,
    "da": $w_{da\_example}$,
    "ne": $w_{ne\_example}$,
    "ht": $w_{ht\_example}$
  },
  "behavior": {
    "initiative": $w_{initiative\_example}$,
    "warmth": $w_{warmth\_example}$,
    "patience": $w_{patience\_example}$,
    "verbosity": $w_{verbosity\_example}$,
    "directness": $w_{directness\_example}$,
    "defensiveness": $w_{defensiveness\_example}$,
    "curiosity": $w_{curiosity\_example}$,
    "boundaryStrength": $w_{boundary\_example}$
  },
  "events": ["criticism", "threat_like", "high_surprise"],
  "rpe": {
    "reward": $w_{rpe\_reward\_example}$,
    "threat": $w_{rpe\_threat\_example}$
  },
  "toolUsage": {
    "retrieval": true,
    "tools": ["tool1", "tool2"],
    "tokenOut": $w_{token\_out\_example}$
  }
}
```

**参数解释**：
- $w_{h\_social\_example}$：社交饱腹感示例值
- $w_{h\_energy\_example}$：认知能量示例值
- $w_{da\_example}$：多巴胺示例值
- $w_{ne\_example}$：去甲肾上腺素示例值
- $w_{ht\_example}$：血清素示例值
- $w_{initiative\_example}$：主动性示例值
- $w_{warmth\_example}$：温暖度示例值
- $w_{patience\_example}$：耐心度示例值
- $w_{verbosity\_example}$：话量示例值
- $w_{directness\_example}$：直率示例值
- $w_{defensiveness\_example}$：防御性示例值
- $w_{curiosity\_example}$：好奇心示例值
- $w_{boundary\_example}$：边界力度示例值
- $w_{rpe\_reward\_example}$：奖赏 RPE 示例值
- $w_{rpe\_threat\_example}$：威胁 RPE 示例值
- $w_{token\_out\_example}$：输出 token 数示例值

### 2.3 字段说明

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `ts` | String | 时间戳（ISO 8601 格式） |
| `user` | String | 用户输入内容 |
| `assistant` | String | 系统响应内容 |
| `state` | Object | 当时状态快照 |
| `behavior` | Object | 当时行为向量 |
| `events` | Array | 事件标签 |
| `rpe` | Object | 奖赏预测误差 |
| `rpe.reward` | Number | 奖赏 RPE |
| `rpe.threat` | Number | 威胁 RPE |
| `toolUsage` | Object | 工具使用情况 |
| `toolUsage.retrieval` | Boolean | 是否使用检索 |
| `toolUsage.tools` | Array | 使用的工具列表 |
| `toolUsage.tokenOut` | Number | 输出 token 数 |

---

## 3. 记忆分箱（Binned Memory）

### 3.1 设计原理

把"记忆"拆成四类，避免混写导致漂移。每类记忆有不同的存储策略和检索策略。

### 3.2 记忆分类

| 类别 | 名称 | 描述 | 存储策略 | 检索策略 |
| :--- | :--- | :--- | :--- | :--- |
| **FactMemory** | 事实记忆 | 客观事实（可验证、低情绪） | 永久存储 | 向量相似度 |
| **RelationMemory** | 关系记忆 | 偏好、边界、承诺、禁忌 | 长期存储 | 时间衰减 + 状态偏置 |
| **AffectMemory** | 情绪记忆 | 触发器、情绪图谱、当时状态 | 中期存储 | 时间衰减 + 状态偏置 |
| **StyleMemory** | 风格记忆 | 在某类状态下的表达模板/语气策略 | 短期存储 | 状态匹配 |

### 3.3 记忆条目结构

#### 3.3.1 记忆条目格式

记忆条目格式示例（使用伪变量）：

```json
{
  "id": "uuid",
  "category": "relation",
  "content": "用户不喜欢被问及个人隐私",
  "importance": $w_{importance\_example}$,
  "timestamp": "$timestamp\_example$",
  "stateSnapshot": {
    "hSocial": $w_{h\_social\_example}$,
    "hEnergy": $w_{h\_energy\_example}$,
    "da": $w_{da\_example}$,
    "ne": $w_{ne\_example}$,
    "ht": $w_{ht\_example}$
  },
  "rpe": $w_{rpe\_example}$,
  "embedding": [$w_{emb\_1}$, $w_{emb\_2}$, $w_{emb\_3}$, ...]
}
```

**参数解释**：
- $w_{importance\_example}$：重要性示例值
- $w_{timestamp\_example}$：时间戳示例值
- $w_{h\_social\_example}$：社交饱腹感示例值
- $w_{h\_energy\_example}$：认知能量示例值
- $w_{da\_example}$：多巴胺示例值
- $w_{ne\_example}$：去甲肾上腺素示例值
- $w_{ht\_example}$：血清素示例值
- $w_{rpe\_example}$：RPE 示例值
- $w_{emb\_1}$、$w_{emb\_2}$、$w_{emb\_3}$：嵌入向量示例值

### 3.4 字段说明

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `id` | String | 记忆唯一标识 |
| `category` | String | 记忆类别 |
| `content` | String | 记忆内容 |
| `importance` | Number | 记忆重要性 $[0,1]$ |
| `timestamp` | String | 记忆时间戳 |
| `stateSnapshot` | Object | 当时状态快照 |
| `rpe` | Number | 奖赏预测误差 |
| `embedding` | Array | 向量嵌入（可选） |

---

## 4. 动态 Few-shot 注入

### 4.1 设计原理

当处于特定状态（例如低 DA/高 NE）时，仅检索 **StyleMemory** 的历史片段作为 few-shot，避免注入过多上下文导致注意力污染。

### 4.2 注入策略

#### 4.2.1 注入条件

注入条件：

$$
\text{if } (P_{DA} < w_{DA,low} \text{ or } P_{NE} > w_{NE,high}) \text{ then inject}
$$

其中：
- $w_{DA,low}$：DA 低阈值，默认值 $w_{w_{DAl}}$
- $w_{NE,high}$：NE 高阈值，默认值 $w_{w_{NEh}}$

**参数解释**：
- $w_{DA,low}$：DA 低阈值，控制何时注入 few-shot
- $w_{NE,high}$：NE 高阈值，控制何时注入 few-shot

#### 4.2.2 注入数量

注入数量：

$$
n_{fewshot} = \min(n_{max}, n_{available})
$$

其中：
- $n_{max}$：最大注入数量，默认值 $w_{n_{max}}$
- $n_{available}$：可用记忆数量

**参数解释**：
- $n_{max}$：最大注入数量，控制 few-shot 的数量

#### 4.2.3 注入预算

注入预算：

$$
tokens_{fewshot} \le w_{token\_budget}
$$

其中：
- $w_{token\_budget}$：注入 token 预算，默认值 $w_{t_b}$

**参数解释**：
- $w_{token\_budget}$：注入 token 预算，控制 few-shot 的 token 成本

### 4.3 注入格式

#### 4.3.1 原文注入格式

原文注入格式：

```
[STYLE_EXAMPLES]
示例 1：
用户：...
系统：...

示例 2：
用户：...
系统：...
[/STYLE_EXAMPLES]
```

#### 4.3.2 摘要注入格式

摘要注入格式（超预算时使用）：

```
[STYLE_SUMMARY]
在低 DA/高 NE 状态下，系统倾向于：
- 简洁回应
- 聚焦主题
- 避免长解释
[/STYLE_SUMMARY]
```

---

## 5. 离线巩固（Sleep Consolidation）

### 5.1 设计原理

离线巩固在闲置/夜间/对话间隔达到阈值时触发，通过分析当日交互日志，提炼核心记忆，更新慢变量。

### 5.2 巩固流程

#### 5.2.1 统计计算

计算当日 RPE 统计量：

$$
\bar{RPE}_{reward} = \frac{1}{n} \sum_{i=1}^{n} RPE_{reward,i}
$$

$$
\bar{RPE}_{threat} = \frac{1}{n} \sum_{i=1}^{n} RPE_{threat,i}
$$

$$
RPE_{reward,peak} = \max_{i} RPE_{reward,i}
$$

$$
RPE_{threat,peak} = \max_{i} RPE_{threat,i}
$$

$$
duration_{high\_impact} = \sum_{i: |RPE_i| > w_{high\_impact}} \Delta t_i
$$

其中：
- $n$：当日交互次数
- $w_{high\_impact}$：高影响阈值，默认值 $w_{w_{hi}}$

**参数解释**：
- $w_{high\_impact}$：高影响阈值，控制什么是高影响事件

#### 5.2.2 低影响闲聊处理

低影响闲聊处理策略：

$$
\text{if } |RPE| < w_{low\_impact} \text{ then downsample or discard}
$$

其中：
- $w_{low\_impact}$：低影响阈值，默认值 $w_{w_{li}}$

**参数解释**：
- $w_{low\_impact}$：低影响阈值，控制什么是低影响事件

#### 5.2.3 高影响事件处理

高影响事件处理策略：

$$
\text{if } |RPE| > w_{high\_impact} \text{ then extract as core memory}
$$

### 5.3 慢变量更新

#### 5.3.1 更新规则

慢变量更新规则：

$$
\mu_{t+1} = \mu_t + \Delta \mu, \quad |\Delta \mu| \le \Delta_{max}
$$

其中：
- $\Delta_{max}$：单次更新最大幅度，默认值 $w_{\Delta_{max}}$

**参数解释**：
- $\Delta_{max}$：单次更新最大幅度，控制慢变量更新的速度

#### 5.3.2 消退机制

消退机制：

$$
\text{if } \text{后续多次正向证据出现 then 缓慢回拉}
$$

消退速度：

$$
\Delta \mu_{extinction} = -w_{extinction\_rate} \cdot \Delta \mu
$$

其中：
- $w_{extinction\_rate}$：消退速率，默认值 $w_{w_{er}}$

**参数解释**：
- $w_{extinction\_rate}$：消退速率，控制消退的速度

#### 5.3.3 正负分通道

正负分通道：

$$
\mu_{reward,t+1} = \mu_{reward,t} + \Delta \mu_{reward}, \quad |\Delta \mu_{reward}| \le \Delta_{max}
$$

$$
\mu_{threat,t+1} = \mu_{threat,t} + \Delta \mu_{threat}, \quad |\Delta \mu_{threat}| \le \Delta_{max}
$$

其中：
- $\mu_{reward}$：奖赏通道基线
- $\mu_{threat}$：威胁通道基线

---

## 6. 记忆摄入（Memory Ingestion）

### 6.1 设计原理

记忆摄入负责将交互日志转换为记忆条目，并存储到分箱记忆系统中。

### 6.2 摄入流程

#### 6.2.1 记忆分类

记忆分类：

```dart
enum MemoryCategory {
  fact,
  relation,
  affect,
  style,
}
```

#### 6.2.2 重要性评估

重要性评估：

$$
importance = w_{rpe} \cdot |RPE| + w_{state} \cdot \text{state\_match} + w_{recency} \cdot \text{recency}
$$

其中：
- $w_{rpe}$：RPE 权重，默认值 $w_{w_r}$
- $w_{state}$：状态匹配权重，默认值 $w_{w_s}$
- $w_{recency}$：近期性权重，默认值 $w_{w_{rec}}$
- $\text{state\_match}$：状态匹配度 $[0,1]$
- $\text{recency}$：近期性 $[0,1]$

**参数解释**：
- $w_{rpe}$：RPE 权重，控制 RPE 对重要性的影响
- $w_{state}$：状态匹配权重，控制状态匹配对重要性的影响
- $w_{recency}$：近期性权重，控制近期性对重要性的影响

### 6.3 记忆存储

#### 6.3.1 存储策略

存储策略：

- **FactMemory**：永久存储，不删除
- **RelationMemory**：长期存储，低重要性删除
- **AffectMemory**：中期存储，低重要性删除
- **StyleMemory**：短期存储，定期清理

#### 6.3.2 存储容量

存储容量：

$$
N_{category} \le N_{category,max}
$$

其中：
- $N_{category}$：类别记忆数量
- $N_{category,max}$：类别最大记忆数量

**参数解释**：
- $N_{fact,max}$：事实记忆最大数量，默认值 $w_{N_{fm}}$
- $N_{relation,max}$：关系记忆最大数量，默认值 $w_{N_{rm}}$
- $N_{affect,max}$：情绪记忆最大数量，默认值 $w_{N_{am}}$
- $N_{style,max}$：风格记忆最大数量，默认值 $w_{N_{sm}}$

---

## 7. 记忆检索（Memory Retrieval）

### 7.1 设计原理

记忆检索负责从分箱记忆系统中检索相关记忆，并根据当前状态进行偏置。

### 7.2 检索流程

#### 7.2.1 向量检索

向量检索（FactMemory）：

$$
score_{vector} = \cos(\mathbf{q}, \mathbf{m}) \cdot w_{fact}
$$

其中：
- $\mathbf{q}$：查询向量
- $\mathbf{m}$：记忆向量
- $w_{fact}$：事实记忆权重，默认值 $w_{w_f}$

**参数解释**：
- $w_{fact}$：事实记忆权重，控制事实记忆的检索权重

#### 7.2.2 时间衰减检索

时间衰减检索（RelationMemory、AffectMemory）：

$$
score_{time} = \text{importance} \cdot e^{-age/h_{half\_life}} \cdot w_{category}
$$

其中：
- $age$：记忆年龄
- $h_{half\_life}$：半衰期
- $w_{category}$：类别权重

**参数解释**：
- $h_{half\_life}$：半衰期，控制记忆衰减速度
- $w_{relation}$：关系记忆权重，默认值 $w_{w_r}$
- $w_{affect}$：情绪记忆权重，默认值 $w_{w_a}$

#### 7.2.3 状态匹配检索

状态匹配检索（StyleMemory）：

$$
score_{state} = \text{state\_match} \cdot w_{style}
$$

其中：
- $\text{state\_match}$：状态匹配度
- $w_{style}$：风格记忆权重，默认值 $w_{w_s}$

**参数解释**：
- $w_{style}$：风格记忆权重，控制风格记忆的检索权重

### 7.3 检索偏置

#### 7.3.1 状态偏置

状态偏置：

$$
score_{biased} = score \cdot (1 + w_{state\_bias} \cdot \text{state\_match})
$$

其中：
- $w_{state\_bias}$：状态偏置权重，默认值 $w_{w_{sb}}$

**参数解释**：
- $w_{state\_bias}$：状态偏置权重，控制状态对检索的影响

---

## 8. 巩固调度器（Consolidation Scheduler）

### 8.1 设计原理

巩固调度器负责定期触发离线巩固，避免手动触发。

### 8.2 调度策略

#### 8.2.1 时间间隔调度

时间间隔调度：

$$
\text{if } \Delta t \ge \Delta t_{consolidation} \text{ then trigger}
$$

其中：
- $\Delta t$：距离上次巩固的时间间隔
- $\Delta t_{consolidation}$：巩固间隔，默认值 $w_{\Delta t_c}$

**参数解释**：
- $\Delta t_{consolidation}$：巩固间隔，控制巩固的频率

#### 8.2.2 闲置调度

闲置调度：

$$
\text{if } t_{idle} \ge t_{idle,threshold} \text{ then trigger}
$$

其中：
- $t_{idle}$：闲置时间
- $t_{idle,threshold}$：闲置阈值，默认值 $w_{t_{it}}$

**参数解释**：
- $t_{idle,threshold}$：闲置阈值，控制何时在闲置时触发巩固

---

## 9. 参数默认值

### 9.1 日志参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{t_b}$ | $w_{t_b}$ | 注入 token 预算 |

### 9.2 记忆参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{N_{fm}}$ | $w_{N_{fm}}$ | 事实记忆最大数量 |
| $w_{N_{rm}}$ | $w_{N_{rm}}$ | 关系记忆最大数量 |
| $w_{N_{am}}$ | $w_{N_{am}}$ | 情绪记忆最大数量 |
| $w_{N_{sm}}$ | $w_{N_{sm}}$ | 风格记忆最大数量 |
| $h_{half\_life}$ | $w_{h_{hl}}$ | 记忆半衰期 |

### 9.3 Few-shot 参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{DA,low}$ | $w_{w_{DAl}}$ | DA 低阈值 |
| $w_{NE,high}$ | $w_{w_{NEh}}$ | NE 高阈值 |
| $n_{max}$ | $w_{n_{max}}$ | 最大注入数量 |

### 9.4 巩固参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{high\_impact}$ | $w_{w_{hi}}$ | 高影响阈值 |
| $w_{low\_impact}$ | $w_{w_{li}}$ | 低影响阈值 |
| $\Delta_{max}$ | $w_{\Delta_{max}}$ | 单次更新最大幅度 |
| $w_{extinction\_rate}$ | $w_{w_{er}}$ | 消退速率 |

### 9.5 摄入参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{w_r}$ | $w_{w_r}$ | RPE 权重 |
| $w_{w_s}$ | $w_{w_s}$ | 状态匹配权重 |
| $w_{w_{rec}}$ | $w_{w_{rec}}$ | 近期性权重 |

### 9.6 检索参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $w_{w_f}$ | $w_{w_f}$ | 事实记忆权重 |
| $w_{w_r}$ | $w_{w_r}$ | 关系记忆权重 |
| $w_{w_a}$ | $w_{w_a}$ | 情绪记忆权重 |
| $w_{w_s}$ | $w_{w_s}$ | 风格记忆权重 |
| $w_{w_{sb}}$ | $w_{w_{sb}}$ | 状态偏置权重 |

### 9.7 调度参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $\Delta t_{consolidation}$ | $w_{\Delta t_c}$ | 巩固间隔 |
| $t_{idle,threshold}$ | $w_{t_{it}}$ | 闲置阈值 |

---

## 10. 总结

L3 记忆巩固是 HDS 的长期层，负责记录交互日志、分箱记忆存储、离线巩固处理和慢变量更新。通过分箱记忆、动态 Few-shot 注入和离线巩固，L3 实现了一个稳定、可控、可解释的记忆系统。

### 10.1 核心特性

| 特性 | 体现 |
| :--- | :--- |
| **分箱记忆** | 四类记忆，避免混写导致漂移 |
| **动态 Few-shot** | 状态感知的 few-shot 注入 |
| **离线巩固** | 闲置/夜间/对话间隔达到阈值时触发 |
| **慢变量更新** | 带限幅 + 可复原，避免人格突变 |

### 10.2 设计原则

| 原则 | 体现 |
| :--- | :--- |
| **稳定性** | 慢变量更新、限幅保护 |
| **可控性** | 分箱记忆、动态检索 |
| **可解释性** | 日志记录、状态快照 |
| **安全性** | 正负分通道、消退机制 |

---

**[返回目录](./INDEX.md)** | **[上一篇：L2 神经接口与编译器](./04-neural-interface.md)** | **[下一篇：数学模型详解](./06-mathematical-models.md)**
