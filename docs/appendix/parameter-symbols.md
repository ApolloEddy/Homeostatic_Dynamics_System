# 附录 · 参数符号表（不含真实数值）

**[返回目录](../INDEX.md)** · **[写作规范](../STYLE.md)** · **[返回主页](../../README.md)**

> [!IMPORTANT]
> 本附录只提供 **符号、语义、约束与归类**。  
> **不提供** 任何来自代码/配置的真实参数值、默认值或建议数值。

---

## A.1 状态变量（State）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(H_{\mathrm{social}}(t)\) | 社交饱腹感（有界：\([H_{\min}, H_{\max}]\)） |
| \(H_{\mathrm{energy}}(t)\) | 认知能量（有界：\([H_{\min}, H_{\max}]\)） |
| \(N_{\mathrm{DA}}(t)\) | 多巴胺通道（有界：\([N_{\min}, N_{\max}]\)） |
| \(N_{\mathrm{NE}}(t)\) | 去甲肾上腺素通道（有界：\([N_{\min}, N_{\max}]\)） |
| \(N_{\mathrm{5HT}}(t)\) | 血清素通道（有界：\([N_{\min}, N_{\max}]\)） |

---

## A.2 稳态（Homeostasis）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(\lambda_{\mathrm{social}}\) | 社交自然衰减率（\(\ge 0\)） |
| \(\rho_{\mathrm{energy}}\) | 能量自然恢复率（\(\ge 0\)） |
| \(\kappa_{\mathrm{task}}\) | 任务消耗系数（\(\ge 0\)） |
| \(\gamma_{q}\) | 交互质量增益（\(\ge 0\)） |
| \(\mathrm{sat}(\cdot)\) | 软饱和函数（连续、可导、有界） |
| \(k_{\mathrm{sat}}\) | 软饱和陡峭度（伪变量） |
| \(x_{\mathrm{mid}}\) | 软饱和中点（伪变量） |

---

## A.3 神经调制（OU / Neuromodulators）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(\mu_i\) | 通道 \(i\) 的基线（慢变量） |
| \(\theta_i\) | 通道 \(i\) 的回归速率（\(\ge 0\)） |
| \(\sigma_i\) | 通道 \(i\) 的噪声强度（\(\ge 0\)） |
| \(\varepsilon_t\) | 高斯噪声样本（分布参数用符号表示） |
| \(I_{i,t}\) | 通道 \(i\) 的刺激输入（经归一化/压缩/限幅） |

---

## A.4 观测与刺激归一化（Perception → Stimulus）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(o_t\) | 冒犯/威胁观测（有界：\([O_{\min}, O_{\max}]\)） |
| \(v_t\) | 效价观测（有界：\([V_{\min}, V_{\max}]\)） |
| \(a_t\) | 唤醒度观测（有界：\([A_{\min}, A_{\max}]\)） |
| \(b_t\) | 边界事件指示变量（布尔或指示函数） |
| \(c_t\) | 置信度门控（有界：\([C_{\min}, C_{\max}]\)） |
| \(s_{\mathrm{stim}}\) | 归一化/压缩尺度参数（伪变量） |
| \(I_{\max}\) | 单次刺激最大幅度（有界） |
| \(q_t\) | 交互质量指标（有界：\([q_{\min}, q_{\max}]\)） |

---

## A.5 行为向量（Behavior Vector）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(\mathbf{b}_t\) | 行为向量（有界：\([B_{\min},B_{\max}]^{k}\)） |
| \(b_{\mathrm{initiative}}\) | 主动性分量 |
| \(b_{\mathrm{warmth}}\) | 温暖度分量 |
| \(b_{\mathrm{patience}}\) | 耐心分量 |
| \(b_{\mathrm{verbosity}}\) | 话量分量 |
| \(b_{\mathrm{directness}}\) | 直率分量 |
| \(b_{\mathrm{defensiveness}}\) | 防御性分量 |
| \(b_{\mathrm{curiosity}}\) | 好奇心分量 |
| \(b_{\mathrm{boundary}}\) | 边界强度分量 |

---

## A.6 采样映射（Sampling）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(T_t\) | temperature（有界：\([T_{\min},T_{\max}]\)） |
| \(P_t\) | top-p（有界：\([P_{\min},P_{\max}]\)） |
| \(M_t\) | max tokens（有界：\([M_{\min},M_{\max}]\)） |
| \(\alpha_{\bullet},\beta_{\bullet}\) | 映射系数集合（伪变量；有界） |

---

## A.7 检索偏置（Retrieval Bias）

| 符号 | 含义与约束 |
| :--- | :--- |
| \(w_{\mathrm{fact}}\) | 事实记忆权重（建议常量或极窄范围） |
| \(w_{\mathrm{relation}}\) | 关系记忆权重（有界偏置） |
| \(w_{\mathrm{affect}}\) | 情绪记忆权重（有界偏置） |
| \(w_{\mathrm{style}}\) | 风格记忆权重（有界偏置） |
| \(h_{\mathrm{half}}\) | 时间尺度/半衰期（符号化） |
| \(w_{\mathrm{state}}\) | 状态匹配权重（符号化；有界） |
| \(w_{\mathrm{rpe}}\) | 显著性/RPE 权重（符号化；有界） |

---

## A.8 阈值符号命名建议（替代 `$w_{...}$`）

> [!NOTE]
> 文档中推荐使用 \(\tau\) 表示阈值，替代“权重式占位符”以提升可读性。

| 语义 | 推荐符号 |
| :--- | :--- |
| 社交低/高阈值 | \(\tau_{\mathrm{social,low}}, \tau_{\mathrm{social,high}}\) |
| 能量低/高阈值 | \(\tau_{\mathrm{energy,low}}, \tau_{\mathrm{energy,high}}\) |
| 压力（NE）低/高阈值 | \(\tau_{\mathrm{ne,low}}, \tau_{\mathrm{ne,high}}\) |
| 动机（DA）低/高阈值 | \(\tau_{\mathrm{da,low}}, \tau_{\mathrm{da,high}}\) |
| 控制（5HT）低/高阈值 | \(\tau_{\mathrm{5ht,low}}, \tau_{\mathrm{5ht,high}}\) |

---

**[返回目录](../INDEX.md)** · **[上一篇：数学模型详解](../06-mathematical-models/README.md)** · **[下一篇：实现指南](../07-implementation-guide/README.md)**


