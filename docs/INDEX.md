# HDS 技术文档目录

> **Homeostatic Dynamics System** 完整技术文档索引

---

## 文档导航

- **[返回主页](../README.md)** - 项目概述与快速开始
- **[写作规范](./STYLE.md)** - 参数脱敏与白皮书版式规范
- **[文档总览](#文档总览)** - 查看所有章节概览
- **[阅读顺序建议](#阅读顺序建议)** - 推荐的学习路径

---

## 文档总览

### 第一部分：基础理论

| 章节 | 文档 | 内容概要 | 阅读时间 |
| :--- | :--- | :--- | :--- |
| 01 | [引言与理论基础](./01-introduction/README.md) | HDS 的科学背景、神经学、心理学、社会行为学基础 | 20 分钟 |
| 02 | [系统架构总览](./02-system-architecture/README.md) | 三层架构设计、数据流、模块职责划分 | 15 分钟 |

### 第二部分：核心模块

| 章节 | 文档 | 内容概要 | 阅读时间 |
| :--- | :--- | :--- | :--- |
| 03 | [L1 数字生理循环](./03-physiological-loop/README.md) | 内驱力系统、神经递质系统、OU 过程、刺激归一化 | 30 分钟 |
| 04 | [L2 神经接口与编译器](./04-neural-interface/README.md) | 行为向量、动态 Prompt 注入、检索偏置、采样映射 | 25 分钟 |
| 05 | [L3 记忆巩固](./05-consolidation/README.md) | 分箱记忆、记忆摄入、检索增强、离线巩固 | 25 分钟 |

### 第三部分：深入技术

| 章节 | 文档 | 内容概要 | 阅读时间 |
| :--- | :--- | :--- | :--- |
| 06 | [数学模型详解](./06-mathematical-models/README.md) | OU 过程推导、状态更新方程、参数映射、稳定性分析 | 35 分钟 |
| 07 | [实现指南](./07-implementation-guide/README.md) | API 设计、配置参数、调试监控、性能优化 | 30 分钟 |
| 08 | [应用场景](./08-applications/README.md) | 对话系统集成、人格稳定性、可解释性分析、扩展应用 | 20 分钟 |

### 附录

| 文档 | 内容概要 |
| :--- | :--- |
| [参数符号表（不含真实数值）](./appendix/parameter-symbols.md) | 全局符号、阈值命名与约束（集中管理） |

---

## 阅读顺序建议

### 快速入门路径（约 1 小时）

适合希望快速了解 HDS 核心概念的读者：

1. [引言与理论基础](./01-introduction/README.md) - 理解 HDS 的科学背景
2. [系统架构总览](./02-system-architecture/README.md) - 掌握整体设计
3. [L1 数字生理循环](./03-physiological-loop/README.md) - 了解核心机制
4. [应用场景](./08-applications/README.md) - 了解实际应用

### 系统学习路径（约 3 小时）

适合希望深入理解 HDS 设计与实现的开发者：

1. [引言与理论基础](./01-introduction/README.md)
2. [系统架构总览](./02-system-architecture/README.md)
3. [L1 数字生理循环](./03-physiological-loop/README.md)
4. [L2 神经接口与编译器](./04-neural-interface/README.md)
5. [L3 记忆巩固](./05-consolidation/README.md)
6. [实现指南](./07-implementation-guide/README.md)
7. [应用场景](./08-applications/README.md)

### 研究者路径（约 4 小时）

适合希望深入研究 HDS 理论基础和数学模型的研究者：

1. [引言与理论基础](./01-introduction/README.md)
2. [系统架构总览](./02-system-architecture/README.md)
3. [L1 数字生理循环](./03-physiological-loop/README.md)
4. [L2 神经接口与编译器](./04-neural-interface/README.md)
5. [L3 记忆巩固](./05-consolidation/README.md)
6. [数学模型详解](./06-mathematical-models/README.md)
7. [实现指南](./07-implementation-guide/README.md)
8. [应用场景](./08-applications/README.md)

---

## 附录

- [写作规范（参数脱敏与版式）](./STYLE.md)
- [参数符号表（不含真实数值）](./appendix/parameter-symbols.md)
- [术语表](./appendix/glossary.md)

---

## 更新日志

- **v1.0.0** (2026-01-31) - 初始版本发布

---

**[返回主页](../README.md)** | **[开始阅读](./01-introduction/README.md)**

