# 数学模型详解

> **Homeostatic Dynamics System** 的数学模型与稳定性分析

---

## 章节导航

**[返回目录](./INDEX.md)** | **[上一篇：L3 记忆巩固](./05-consolidation.md)** | **[下一篇：实现指南](./07-implementation-guide.md)**

---

## 1. 概述

### 1.1 数学基础

HDS 的数学基础来源于随机微分方程、稳定性理论和参数优化。通过严谨的数学建模，HDS 实现了一个稳定、可控、可解释的数字人格系统。

### 1.2 核心数学模型

HDS 的核心数学模型包括：

- Ornstein-Uhlenbeck（OU）过程：神经递质动力学
- 状态更新方程：内驱力系统
- 参数映射公式：行为向量与采样参数
- 稳定性分析：系统收敛性与有界性

---

## 2. Ornstein-Uhlenbeck 过程

### 2.1 定义

Ornstein-Uhlenbeck（OU）过程是具有均值回归特性的随机过程，其随机微分方程为：

$$
dX_t = \theta(\mu - X_t)dt + \sigma dW_t + I_{stimulus}
$$

其中：
- $X_t$：状态变量（如神经递质水平）
- $\mu$：均值（基线）
- $\theta$：回归速率（弹性）
- $\sigma$：扩散系数（噪声强度）
- $W_t$：维纳过程（布朗运动）
- $I_{stimulus}$：外界刺激输入

### 2.2 离散化实现

#### 2.2.1 欧拉-丸山方法

OU 过程的离散化形式（欧拉-丸山方法）：

$$
X_{t+\Delta t} = X_t + \theta(\mu - X_t)\Delta t + \sigma\sqrt{\Delta t}\,\epsilon + I_{stimulus}
$$

其中：
- $\Delta t$：时间步长
- $\epsilon \sim \mathcal{N}(0,1)$：标准正态随机变量

#### 2.2.2 高斯随机数生成

标准正态随机数生成（Box-Muller 变换）：

$$
\epsilon = \sqrt{-2\ln(u_1)} \cos(2\pi u_2)
$$

其中：
- $u_1, u_2 \sim \mathcal{U}(0,1)$：均匀分布随机变量

### 2.3 稳定性分析

#### 2.3.1 均值回归

OU 过程具有均值回归特性：

$$
\lim_{t \to \infty} \mathbb{E}[X_t] = \mu
$$

**证明**：

解 OU 过程的随机微分方程：

$$
X_t = \mu + (X_0 - \mu)e^{-\theta t} + \sigma \int_0^t e^{-\theta(t-s)} dW_s
$$

取期望：

$$
\mathbb{E}[X_t] = \mu + (X_0 - \mu)e^{-\theta t}
$$

当 $t \to \infty$ 时：

$$
\lim_{t \to \infty} \mathbb{E}[X_t] = \mu
$$

#### 2.3.2 方差收敛

OU 过程的方差收敛：

$$
\lim_{t \to \infty} \text{Var}(X_t) = \frac{\sigma^2}{2\theta}
$$

**证明**：

计算方差：

$$
\text{Var}(X_t) = \frac{\sigma^2}{2\theta}(1 - e^{-2\theta t})
$$

当 $t \to \infty$ 时：

$$
\lim_{t \to \infty} \text{Var}(X_t) = \frac{\sigma^2}{2\theta}
$$

#### 2.3.3 平稳分布

OU 过程的平稳分布：

$$
X_t \sim \mathcal{N}\left(\mu, \frac{\sigma^2}{2\theta}\right)
$$

**证明**：

当 $t \to \infty$ 时，OU 过程收敛到平稳分布：

$$
X_{\infty} \sim \mathcal{N}\left(\mu, \frac{\sigma^2}{2\theta}\right)
$$

### 2.4 参数影响

#### 2.4.1 回归速率 $\theta$

回归速率 $\theta$ 控制：

- 回归速度：$\theta$ 越大，回归越快
- 波动抑制：$\theta$ 越大，波动越小
- 半衰期：$t_{1/2} = \ln(2)/\theta$

#### 2.4.2 噪声强度 $\sigma$

噪声强度 $\sigma$ 控制：

- 波动程度：$\sigma$ 越大，波动越大
- 平稳方差：$\text{Var}(X_{\infty}) = \sigma^2/(2\theta)$

#### 2.4.3 均值 $\mu$

均值 $\mu$ 控制：

- 长期均值：$\lim_{t \to \infty} \mathbb{E}[X_t] = \mu$
- 人格基线：反映人格特质

---

## 3. 状态更新方程

### 3.1 内驱力系统

#### 3.1.1 社交饱腹感

社交饱腹感更新方程：

$$
H_{social}(t+\Delta t) = \sigma_{soft}(H_{social}(t) - d_s \Delta t + w_{quality} \cdot quality)
$$

其中：
- $d_s$：社交需求衰减率（每小时）
- $w_{quality}$：互动质量权重
- $quality$：互动质量 $[0,1]$
- $\sigma_{soft}$：软饱和函数

#### 3.1.2 认知能量

认知能量更新方程：

$$
H_{energy}(t+\Delta t) = \sigma_{soft}(H_{energy}(t) + r_e \Delta t - c_e \cdot complexity)
$$

其中：
- $r_e$：认知能量恢复率（每小时）
- $c_e$：基础任务消耗
- $complexity$：任务复杂度倍率

### 3.2 软饱和函数

#### 3.2.1 Sigmoid 函数

Sigmoid 函数：

$$
\sigma_{soft}(x) = \frac{1}{1 + e^{-k(x - w_{midpoint})}}
$$

其中：
- $k$：压缩系数
- $w_{midpoint}$：中点位置

**参数解释**：
- $w_{midpoint}$：sigmoid函数的中点位置，通常设为$w_{midpoint\_default}$

#### 3.2.2 特性分析

Sigmoid 函数的特性：

- **单调性**：$\sigma_{soft}'(x) > 0$
- **有界性**：$\sigma_{soft}(x) \in (0,1)$
- **对称性**：$\sigma_{soft}(w_{midpoint} + \delta) = 1 - \sigma_{soft}(w_{midpoint} - \delta)$
- **导数最大**：$\sigma_{soft}'(w_{midpoint}) = k/4$

#### 3.2.3 压缩系数选择

压缩系数 $k$ 的选择：

- $k$ 越大，饱和越快
- $k$ 越小，饱和越慢
- 推荐值：$k \in [4,8]$

---

## 4. 参数映射公式

### 4.1 线性插值

#### 4.1.1 定义

线性插值：

$$
\text{lerp}(a, b, t) = a + (b - a) \cdot t
$$

其中：
- $a$：下限
- $b$：上限
- $t$：插值参数 $[0,1]$

#### 4.1.2 特性

线性插值的特性：

- **单调性**：$\text{lerp}(a, b, t)$ 关于 $t$ 单调
- **边界性**：$\text{lerp}(a, b, 0) = a$, $\text{lerp}(a, b, 1) = b$
- **线性**：$\text{lerp}(a, b, t)$ 关于 $t$ 线性

### 4.2 行为向量映射

#### 4.2.1 $H_{social}$ 映射

$H_{social}$ 到行为向量的映射：

$$
b_1 = \text{lerp}(b_{1,low}, b_{1,high}, H_{social})
$$

$$
b_2 = \text{lerp}(b_{2,low}, b_{2,high}, H_{social})
$$

#### 4.2.2 $H_{energy}$ 映射

$H_{energy}$ 到行为向量的映射：

$$
b_3 = \text{lerp}(b_{3,low}, b_{3,high}, H_{energy})
$$

$$
b_4 = \text{lerp}(b_{4,low}, b_{4,high}, H_{energy})
$$

#### 4.2.3 $P_{NE}$ 映射

$P_{NE}$ 到行为向量的映射：

$$
b_5 = \text{lerp}(b_{5,low}, b_{5,high}, P_{NE})
$$

$$
b_6 = \text{lerp}(b_{6,low}, b_{6,high}, P_{NE})
$$

#### 4.2.4 $P_{DA}$ 映射

$P_{DA}$ 到行为向量的映射：

$$
b_7 = \text{lerp}(b_{7,low}, b_{7,high}, P_{DA})
$$

#### 4.2.5 $P_{5HT}$ 映射

$P_{5HT}$ 到行为向量的映射：

$$
b_8 = \text{lerp}(b_{8,low}, b_{8,high}, P_{5HT})
$$

### 4.3 采样参数映射

#### 4.3.1 温度映射

温度映射：

$$
T = \text{clamp}(T_{min}, T_{max}, T_{base} - w_{T,NE} \cdot P_{NE})
$$

#### 4.3.2 Top-P 映射

Top-P 映射：

$$
top\_p = \text{clamp}(p_{min}, p_{max}, p_{base} - w_{p,NE} \cdot P_{NE} + w_{p,5HT} \cdot P_{5HT})
$$

#### 4.3.3 最大 Token 数映射

最大 Token 数映射：

$$
max\_tokens = \text{clamp}(tokens_{min}, tokens_{max}, tokens_{base} \cdot f_{energy} \cdot f_{verbosity})
$$

其中：

$$
f_{energy} = w_{f_{e,base}} + w_{f_{e,scale}} \cdot H_{energy}
$$

$$
f_{verbosity} = w_{f_{v,base}} + w_{f_{v,scale}} \cdot b_4
$$

---

## 5. 刺激归一化

### 5.1 Tanh 归一化

#### 5.1.1 定义

Tanh 归一化：

$$
I_{normalized} = \tanh\left(\frac{raw}{s_{stimulus}}\right)
$$

其中：
- $raw$：原始刺激值
- $s_{stimulus}$：归一化尺度

#### 5.1.2 特性

Tanh 函数的特性：

- **奇函数**：$\tanh(-x) = -\tanh(x)$
- **有界性**：$\tanh(x) \in (-1,1)$
- **单调性**：$\tanh'(x) > 0$
- **渐近线**：$\lim_{x \to \infty} \tanh(x) = 1$, $\lim_{x \to -\infty} \tanh(x) = -1$

### 5.2 限幅函数

#### 5.2.1 定义

限幅函数：

$$
\text{clamp}(x, x_{min}, x_{max}) = \min(\max(x, x_{min}), x_{max})
$$

其中：
- $x$：输入值
- $x_{min}$：下限
- $x_{max}$：上限

#### 5.2.2 特性

限幅函数的特性：

- **有界性**：$\text{clamp}(x, x_{min}, x_{max}) \in [x_{min}, x_{max}]$
- **单调性**：$\text{clamp}(x, x_{min}, x_{max})$ 关于 $x$ 单调
- **恒等性**：$\text{clamp}(x, x_{min}, x_{max}) = x$ 当 $x \in [x_{min}, x_{max}]$

---

## 6. 记忆检索

### 6.1 余弦相似度

#### 6.1.1 定义

余弦相似度：

$$
\cos(\mathbf{q}, \mathbf{m}) = \frac{\mathbf{q} \cdot \mathbf{m}}{\|\mathbf{q}\| \|\mathbf{m}\|}
$$

其中：
- $\mathbf{q}$：查询向量
- $\mathbf{m}$：记忆向量

#### 6.1.2 特性

余弦相似度的特性：

- **有界性**：$\cos(\mathbf{q}, \mathbf{m}) \in [-1,1]$
- **对称性**：$\cos(\mathbf{q}, \mathbf{m}) = \cos(\mathbf{m}, \mathbf{q})$
- **归一化不变性**：$\cos(\alpha\mathbf{q}, \beta\mathbf{m}) = \cos(\mathbf{q}, \mathbf{m})$

### 6.2 时间衰减

#### 6.2.1 指数衰减

指数衰减：

$$
w_{decay} = e^{-age/h_{half\_life}}
$$

其中：
- $age$：记忆年龄
- $h_{half\_life}$：半衰期

#### 6.2.2 半衰期

半衰期：

$$
t_{1/2} = h_{half\_life} \cdot \ln(2)
$$

### 6.3 检索分数

#### 6.3.1 基础分数

基础检索分数：

$$
score_{base} = sim \cdot w_{category}
$$

#### 6.3.2 状态匹配分数

状态匹配分数：

$$
score_{state} = w_{state\_bias} \cdot (1 + e^{-age/h_{half\_life}} + |RPE| \cdot w_{rpe})
$$

#### 6.3.3 最终分数

最终检索分数：

$$
score = score_{base} + score_{state}
$$

---

## 7. 稳定性分析

### 7.1 系统收敛性

#### 7.1.1 OU 过程收敛性

OU 过程的收敛性：

$$
\lim_{t \to \infty} X_t \sim \mathcal{N}\left(\mu, \frac{\sigma^2}{2\theta}\right)
$$

**结论**：OU 过程收敛到平稳分布。

#### 7.1.2 内驱力系统收敛性

内驱力系统的收敛性：

$$
\lim_{t \to \infty} H_{social}(t) = \sigma_{soft}(\mu_{social})
$$

$$
\lim_{t \to \infty} H_{energy}(t) = \sigma_{soft}(\mu_{energy})
$$

其中：
- $\mu_{social}$：社交饱腹感均值
- $\mu_{energy}$：认知能量均值

**结论**：内驱力系统收敛到软饱和的均值。

### 7.2 系统有界性

#### 7.2.1 OU 过程有界性

OU 过程的有界性：

$$
X_t \in [X_{min}, X_{max}]
$$

其中：
- $X_{min}$：下限
- $X_{max}$：上限

**结论**：通过限幅，OU 过程有界。

#### 7.2.2 内驱力系统有界性

内驱力系统的有界性：

$$
H_{social}(t) \in (0,1)
$$

$$
H_{energy}(t) \in (0,1)
$$

**结论**：通过软饱和，内驱力系统有界。

### 7.3 系统稳定性

#### 7.3.1 李雅普诺夫稳定性

李雅普诺夫稳定性：

**定义**：系统 $\dot{x} = f(x)$ 在平衡点 $x^*$ 是稳定的，如果对于任意 $\epsilon > 0$，存在 $\delta > 0$，使得 $\|x(0) - x^*\| < \delta$ 蕴含 $\|x(t) - x^*\| < \epsilon$ 对所有 $t \ge 0$。

**OU 过程的李雅普诺夫函数**：

$$
V(x) = (x - \mu)^2
$$

**导数**：

$$
\dot{V}(x) = 2(x - \mu)\dot{x} = 2(x - \mu)(-\theta(x - \mu)) = -2\theta(x - \mu)^2 \le 0
$$

**结论**：OU 过程在平衡点 $\mu$ 是李雅普诺夫稳定的。

#### 7.3.2 渐近稳定性

渐近稳定性：

**定义**：系统 $\dot{x} = f(x)$ 在平衡点 $x^*$ 是渐近稳定的，如果它是稳定的且 $\lim_{t \to \infty} \|x(t) - x^*\| = 0$。

**OU 过程的渐近稳定性**：

$$
\lim_{t \to \infty} \mathbb{E}[|X_t - \mu|] = 0
$$

**结论**：OU 过程在平衡点 $\mu$ 是渐近稳定的。

---

## 8. 参数优化

### 8.1 梯度下降

#### 8.1.1 定义

梯度下降：

$$
\theta_{t+1} = \theta_t - \alpha \nabla J(\theta_t)
$$

其中：
- $\theta$：参数向量
- $\alpha$：学习率
- $J(\theta)$：损失函数

#### 8.1.2 学习率选择

学习率 $\alpha$ 的选择：
- $\alpha$ 太大：发散
- $\alpha$ 太小：收敛慢
- 推荐值：$\alpha \in [w_{\alpha\_min}, w_{\alpha\_max}]$

**参数解释**：
- $w_{\alpha\_min}$：学习率最小值，默认值 $w_{w_{am}}$
- $w_{\alpha\_max}$：学习率最大值，默认值 $w_{w_{aM}}$

### 8.2 贝叶斯优化

#### 8.2.1 定义

贝叶斯优化适用于黑盒函数优化：

1. 构建代理模型（如高斯过程）
2. 采集函数选择下一个采样点
3. 更新代理模型
4. 重复直到收敛

#### 8.2.2 采集函数

采集函数示例：

**Expected Improvement (EI)**：

$$
EI(x) = \mathbb{E}[\max(f(x) - f(x^+), 0)]
$$

其中：
- $f(x)$：目标函数
- $f(x^+)$：当前最优值

---

## 9. 参数默认值

### 9.1 OU 过程参数

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

### 9.2 内驱力参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $d_s$ | $w_{d_s}$ | 社交需求衰减率 |
| $r_e$ | $w_{r_e}$ | 认知能量恢复率 |
| $c_e$ | $w_{c_e}$ | 基础任务消耗 |
| $k$ | $w_{k}$ | 软饱和压缩系数 |

### 9.3 刺激参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $s_{stimulus}$ | $w_{s_{stimulus}}$ | 刺激归一化尺度 |
| $I_{max}$ | $w_{I_{max}}$ | 刺激最大幅度 |

### 9.4 行为向量参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $b_{i,low}$ | $w_{b_{il}}$ | 行为向量第 $i$ 维下限 |
| $b_{i,high}$ | $w_{b_{ih}}$ | 行为向量第 $i$ 维上限 |

### 9.5 采样参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| $T_{min}$ | $w_{T_{min}}$ | 温度下限 |
| $T_{max}$ | $w_{T_{max}}$ | 温度上限 |
| $T_{base}$ | $w_{T_b}$ | 温度基线 |
| $w_{T,NE}$ | $w_{w_{TN}}$ | NE 对温度的影响权重 |
| $p_{min}$ | $w_{p_{min}}$ | Top-P 下限 |
| $p_{max}$ | $w_{p_{max}}$ | Top-P 上限 |
| $p_{base}$ | $w_{p_b}$ | Top-P 基线 |
| $w_{p,NE}$ | $w_{w_{pN}}$ | NE 对 Top-P 的影响权重 |
| $w_{p,5HT}$ | $w_{w_{p5}}$ | 5HT 对 Top-P 的影响权重 |

---

## 10. 总结

HDS 的数学模型来源于随机微分方程、稳定性理论和参数优化。通过严谨的数学建模，HDS 实现了一个稳定、可控、可解释的数字人格系统。

### 10.1 核心数学模型

| 模型 | 应用 |
| :--- | :--- |
| **OU 过程** | 神经递质动力学 |
| **状态更新方程** | 内驱力系统 |
| **参数映射公式** | 行为向量与采样参数 |
| **稳定性分析** | 系统收敛性与有界性 |

### 10.2 数学特性

| 特性 | 体现 |
| :--- | :--- |
| **均值回归** | OU 过程收敛到基线 |
| **有界性** | 软饱和与限幅 |
| **稳定性** | 李雅普诺夫稳定性 |
| **可优化性** | 梯度下降与贝叶斯优化 |

---

## 参考文献

### 随机微分方程

1. Øksendal, B. (2003). *Stochastic differential equations: an introduction with applications*. Springer.
2. Kloeden, P. E., & Platen, E. (1999). *Numerical solution of stochastic differential equations*. Springer.

### 稳定性理论

3. Khalil, H. K. (2002). *Nonlinear systems*. Prentice Hall.
4. Slotine, J. J., & Li, W. (1991). *Applied nonlinear control*. Prentice Hall.

### 参数优化

5. Nocedal, J., & Wright, S. (2006). *Numerical optimization*. Springer.
6. Rasmussen, C. E. (2006). *Gaussian processes for machine learning*. MIT Press.

---

**[返回目录](./INDEX.md)** | **[上一篇：L3 记忆巩固](./05-consolidation.md)** | **[下一篇：实现指南](./07-implementation-guide.md)**
