# Homeostatic Dynamics System (HDS)

> **生存稳态动力学系统** —— 基于生物神经动力学的数字人格稳定性框架

---

## 项目概述

**Homeostatic Dynamics System (HDS)** 是一个受人体神经动力学和心理学启发的仿生系统，旨在解决人工智能人格长期稳定性问题。该系统通过模拟生物体的稳态调节机制，构建了一个包含数字生理循环、神经接口与记忆巩固的闭环控制系统。

HDS 的核心理念是将人工智能系统视为一个具有"虚拟生理—认知策略—记忆巩固"闭环的有机体，通过数值化的状态变量（内驱力、神经递质、行为向量）实现连续的、可解释的、可调控的人格演化。

### 核心特性

- **生物启发的稳态机制**：基于内驱力理论和神经递质调节的动态平衡系统
- **连续性状态演化**：跨回合稳定的状态惯性，避免人格突变
- **可解释性调节**：每次行为输出均可追溯至底层状态变量
- **分层架构设计**：L1生理循环、L2神经接口、L3记忆巩固的三层清晰分离
- **参数化控制**：所有映射关系均可调参、可限幅、可回滚
- **模块化接口**：可独立打包为 `hds_core`，对外暴露稳定 API

---

## 文档导航

### 快速开始

- [完整文档目录](docs/INDEX.md) - 查看所有技术文档
- [写作规范（参数脱敏与版式）](docs/STYLE.md) - 白皮书化排版与脱敏规则
- [引言与理论基础](docs/01-introduction/README.md) - 了解 HDS 的科学背景
- [系统架构总览](docs/02-system-architecture/README.md) - 掌握整体设计

### 核心模块

- [L1 数字生理循环](docs/03-physiological-loop/README.md) - 内驱力与神经递质系统
- [L2 神经接口与编译器](docs/04-neural-interface/README.md) - 行为向量与动态控制
- [L3 记忆巩固](docs/05-consolidation/README.md) - 分箱记忆与离线巩固

### 深入学习

- [数学模型详解](docs/06-mathematical-models/README.md) - OU 过程与稳定性分析
- [实现指南](docs/07-implementation-guide/README.md) - API 设计与配置说明
- [应用场景](docs/08-applications/README.md) - 对话系统与人格控制

---

## 系统概览

HDS 遵循生物闭环逻辑：

```
外界刺激 → 稳态偏移 → 内驱力/神经调制 → 行为向量（policy）
→ 编译输出（prompt/检索/采样）→ 交互结果 → 日志与巩固 → 慢变量更新
```

### 三层架构

| 层级 | 名称 | 职责 |
| :--- | :--- | :--- |
| **L1** | 数字生理循环 | 管理内驱力（H）和神经递质（N）的状态更新 |
| **L2** | 神经接口与编译器 | 将状态翻译为行为向量和控制指令 |
| **L3** | 长时序人格固化 | 离线整理与慢变量更新，解决漂移与遗忘 |

---

## 核心概念

> [!IMPORTANT]
> **参数脱敏原则（强制）**：本文档与 `docs/` 不展示任何来自代码/配置的真实参数值；  
> 数学公式与表格只使用符号变量表示可调参数与边界（详见 `docs/STYLE.md`）。

### 内驱力系统（Homeostasis）

内驱力是生物体维持生存的基本需求驱动。在 HDS 中，内驱力变量被约束在有界区间 $[H_{\min}, H_{\max}]$，采用软饱和而非硬截断，避免贴边后策略失真。

**核心内驱力变量**：

- **$H_{social}$（社交饱腹感）**：时间衰减导致孤独积累，高质量互动回升
- **$H_{energy}$（认知能量）**：随推理/工具调用/长文本生成消耗，随空闲/睡眠恢复

### 神经递质系统（Neuromodulators）

神经递质遵循 Ornstein–Uhlenbeck（OU）随机过程，具备惯性和回归特性：

$$
dX_t = \theta(\mu - X_t)dt + \sigma dW_t + I_{stimulus}
$$

其中：
- $\mu$：人格/基线（慢变量）
- $\theta$：回归速率（弹性）
- $\sigma$：噪声强度（个体差异/波动）
- $I_{stimulus}$：外界刺激输入（归一化与限幅）

**核心神经递质**：

- **\(N_{\mathrm{DA}}\)（Dopamine）**：奖赏预测误差（RPE）驱动的动机/学习率调制
- **\(N_{\mathrm{NE}}\)（Norepinephrine）**：唤醒/压力与注意力集中
- **\(N_{\mathrm{5HT}}\)（Serotonin）**：抑制/控制与长期规划能力

### 行为向量（Behavior Vector）

行为向量是 HDS 与 LLM 之间的稳定中间层，$B \in [B_{\min}, B_{\max}]^k$：

| 维度 | 名称 | 含义 |
| :--- | :--- | :--- |
| $b_1$ | initiative | 主动性 |
| $b_2$ | warmth | 温暖/亲和 |
| $b_3$ | patience | 耐心/解释意愿 |
| $b_4$ | verbosity | 话量 |
| $b_5$ | directness | 直率/委婉 |
| $b_6$ | defensiveness | 防御性 |
| $b_7$ | curiosity | 提问/探索 |
| $b_8$ | boundary_strength | 边界力度 |

---

## 技术亮点

### 1. 稳定性保障

- **软饱和边界**：使用 sigmoid 压缩而非硬截断，避免策略失真
- **刺激归一化**：所有外界刺激经过归一化、裁剪、双时间常数处理
- **限幅保护**：所有映射关系均设置上下限，防止极端值传播

### 2. 可解释性

- **状态快照**：每次输出附带完整状态快照和决策理由
- **追溯链**：可从最终输出追溯至底层状态变量
- **调试接口**：提供丰富的调试信息和监控指标

### 3. 可扩展性

- **模块化设计**：L1/L2/L3 三层清晰分离，可独立演进
- **接口化封装**：对外暴露稳定 API（tick / observe / compile / consolidate）
- **配置驱动**：所有参数可通过配置文件热加载

### 4. 工程化

- **本地持久化**：使用 SharedPreferences 实现状态持久化
- **调度器支持**：内置巩固调度器，支持定时触发
- **监控指标**：提供状态饱和率、恢复半衰期等关键指标

---

## 应用场景

HDS 可作为可插拔的"大脑接口模块"，为上层对话代理（LLM）输出可控、可解释、可监控的行为调度信号。

### 对话系统集成

- 动态调节对话语气、节奏、深度
- 根据用户情绪调整回应策略
- 维持长期对话的一致性

### 人格稳定性控制

- 避免人格突变和极端行为
- 实现渐进式人格演化
- 支持人格参数的精细调优

### 可解释性分析

- 追溯每次决策的底层原因
- 监控人格演化轨迹
- 诊断异常行为模式

---

## 技术栈

- **语言**：Dart 3.0+
- **框架**：Flutter 3.10+
- **存储**：SharedPreferences
- **数学**：随机微分方程、向量计算
- **架构**：分层架构、模块化设计

---

## 快速开始

### 安装依赖

```bash
flutter pub get
```

### 初始化 HDS 服务

```dart
final hds = await HDSService.init(
  memoryService: memoryService,
  configOverride: customConfig,
);
```

### 使用示例

```dart
// 观测事件
hds.observe(HDSEvent.fromPerception(
  offensiveness: offensivenessScore,
  valence: valenceScore,
  arousal: arousalScore,
));

// 编译控制输出
final bundle = hds.compile();
final promptInjection = bundle.promptInjection;
final sampling = bundle.sampling;
```

- `offensivenessScore`：观测到的冒犯/威胁强度（取值域用符号变量表示）
- `valenceScore`：观测到的效价（取值域用符号变量表示）
- `arousalScore`：观测到的唤醒度（取值域用符号变量表示）

---

## 贡献指南

欢迎贡献代码、提出问题或改进建议。请遵循以下原则：

- 保持代码风格一致性
- 添加必要的注释和文档
- 确保所有测试通过
- 提交前进行代码审查

---

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件

---

## 联系方式

- **作者**：ApolloEddy
- **项目主页**：[Homeostatic Dynamics System](https://github.com/ApolloEddy/Homeostatic_Dynamics_System)

---

## 致谢

本系统的设计灵感来源于神经科学、心理学、社会行为学、数学和计算机科学等多个领域的理论研究。特别感谢以下领域的先驱者：

- **神经科学**：多巴胺奖赏系统、去甲肾上腺素唤醒理论、血清素情绪调节
- **心理学**：内驱力理论、情绪调节理论、人格稳定性研究
- **社会行为学**：社交互动理论、边界设定理论、关系维护机制
- **数学**：随机微分方程、稳定性理论、参数优化
- **计算机科学**：系统架构设计、API 设计、数据结构

---

**[返回目录](docs/INDEX.md)** | **[开始阅读](docs/01-introduction/README.md)**

