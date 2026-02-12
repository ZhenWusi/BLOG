---
title: RLHF 的三阶段流程
date: 2026-02-12
tags: [LLM]
categories: [算法]
excerpt: 深入理解 RLHF（Reinforcement Learning from Human Feedback）的三个核心阶段
---

## 基础知识
### RLHF 的三阶段流程
RLHF（Reinforcement Learning from Human Feedback）通常包含三个主要阶段：

**监督微调（SFT）→ 偏好采样 + 奖励模型学习 → 强化学习优化（RL）**

简单可以理解为：先学"怎么说话"，再学"什么话更好"，最后逼模型多说"好话"

#### 阶段一：SFT（Supervised Fine-Tuning）
RLHF 通常从对一个预训练语言模型进行监督微调开始，使用高质量的人工数据（如对话、摘要等），得到一个模型 $\pi\_{SFT}$。

**数据形式**：$(\text{prompt } x, \text{高质量回答 } y)$

**损失函数**：标准的监督学习损失函数

$$\max_{\theta} \sum_{(x,y) \in D} \log \pi_\theta(y|x)$$

+ $\pi\_\theta$：策略模型的参数化形式
+ $D$：监督微调数据集
+ $x$：输入提示
+ $y$：目标回答

**目标**：让模型学会基本的对话能力和任务完成能力，即"怎么说话"

#### 阶段二：奖励模型（Reward Modeling）
使用 SFT 模型，对同一个 prompt $x$ 采样两个回答：

$$y_1, y_2 \sim \pi_{SFT}(y|x)$$

交给人类标注者，让他们选更好的一个：

$$y_w \succ y_l \mid x$$

+ $y_w$：winner,preferred（赢家）
+ $y_l$：loser,rejected（输家）



**潜在奖励函数假设**：假设人类偏好是由一个未知的真实奖励函数 $r^*(x,y)$ 生成的。我们并不知道它，但假设它存在。



**偏好模型**：

$$p^*(y_1 > y_2 | x) = \frac{\exp(r^*(x, y_1))}{\exp(r^*(x, y_1)) + \exp(r^*(x, y_2))} \tag{1}$$

**数据集形式**：

$$D = \{(x^{(i)}, y_w^{(i)}, y_l^{(i)})\}_{i=1}^N$$

**损失函数**：

$$\mathcal{L}_\text{R}(r_\phi, \mathcal{D}) = -\mathbb{E}_{(x, y_w, y_l) \sim \mathcal{D}} \left[ \log \sigma (r_\phi(x, y_w) - r_\phi(x, y_l)) \right] \tag{2}$$

---

##### 补充解释
BT 模型：

Bradley-Terry 模型是一个用于**成对比较**的概率模型

**核心思想**：

+ 每个对象 $i$ 有一个隐藏的"实力值"参数 $\lambda_i$
+ 当两个对象 $i$ 和 $j$ 比较时，$i$ 战胜 $j$ 的概率为：

$$P(i \succ j) = \frac{\lambda_i}{\lambda_i + \lambda_j}$$

**在 DPO 中的应用**：

+ 将奖励函数 $r(x,y)$ 看作"实力值"
+ 人类偏好 $ y_w \succ y_l $ 就是"比较结果"
+ 通过 BT 模型建立奖励与偏好的桥梁



为什么不用「谁分数大就选谁」？

如果直接规定：$$y_1 \succ y_2 \iff r^*(x,y_1) > r^*(x,y_2)$$  
那问题是：人类判断有噪声，同一个人、同一个问题，不一定每次选同样的。所以我们不建模成确定性规则，而是：概率模型

把公式 (1) 改写一下：

$$p^*(y_1 \succ y_2 \mid x) = \frac{1}{1 + e^{-(r^*(x,y_1) - r^*(x,y_2))}}$$

这就是熟悉的 Sigmoid 函数：

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

于是：

$$p^*(y_1 \succ y_2 \mid x) = \sigma(r^*(x,y_1) - r^*(x,y_2))$$

因为真实数据里人确实选择了 $ y_w $，所以该事件的概率是上式，对数似然是：

$$\log p_\theta(y_w \succ y_l \mid x) = \log \sigma(r_\theta(x,y_w) - r_\theta(x,y_l))$$

最大化所有样本的对数似然：

$$\max_\theta \sum_{i=1}^N \log \sigma(r_\theta(x^{(i)}, y_w^{(i)}) - r_\theta(x^{(i)}, y_l^{(i)}))$$

意思是：对每一条数据，都算一次"模型认为人类会选赢家的概率的对数"，然后把它们加起来，让这个总和尽可能大。对数把“连乘”变成“连加”,最优解不变(对数是单调递增函数)

等价地，最小化负对数似然（Likelihood）：

$$L_R = -\mathbb{E}_{(x,y_w,y_l) \sim D} [\log \sigma(r_\theta(x,y_w) - r_\theta(x,y_l))]$$

这正是公式 (2)。意思是：对数据集里的样本取平均，然后对"对数概率"取负号，让这个值尽可能小。



对一个有限数据集 $D = \{z_1, ..., z_N\}$，如果你均匀随机从中抽一个样本，那么：

$$\mathbb{E}_{z \sim D} [f(z)] = \frac{1}{N} \sum_{i=1}^N f(z_i)$$

👉 期望 = 平均值

令 $f(x,y\_w,y\_l) = \log \sigma(r\_\theta(x,y\_w) - r\_\theta(x,y\_l))$ 那么：

$$\mathbb{E}_{(x,y_w,y_l) \sim D} [f] = \frac{1}{N} \sum_{i=1}^N \log \sigma(r_\theta(x^{(i)}, y_w^{(i)}) - r_\theta(x^{(i)}, y_l^{(i)}))$$

因为 $\sum\_{i=1}^N f\_i$ 和 $\frac{1}{N} \sum\_{i=1}^N f\_i$ 只差一个常数 $\frac{1}{N}$。而在优化中：乘以一个正的常数，不会改变最优解的位置。也就是说：$$\arg\max\_\theta \sum\_{i=1}^N f\_i = \arg\max\_\theta \frac{1}{N} \sum\_{i=1}^N f\_i$$

原目标：$$\max_\theta \mathbb{E}_D [\log \sigma(\cdots)]$$

等价于：$$\min_\theta -\mathbb{E}_D [\log \sigma(\cdots)]$$

于是定义：$$L_R = -\mathbb{E}_D [\log \sigma(\cdots)]$$

**为什么机器学习里"总是最小化"？**

因为梯度下降（Gradient Descent）默认是 minimize loss，所以大家习惯：把"想最大化的目标"，写成"要最小化的损失"。

---

#### 阶段三：RL 微调
**RL 目标函数（核心）**：

$$\max_{\pi_\theta} \mathbb{E}_{x \sim \mathcal{D}, y \sim \pi_\theta(y|x)} \left[ r_\phi(x, y) - \beta D_{\text{KL}} \left[ \pi_\theta(y|x) \middle\| \pi_{\text{ref}}(y|x) \right] \right] \tag{3}$$

---

##### 补充解释
一句话解释：👉 让模型生成 奖励高的回答 👉 但不能偏离原始 SFT 模型太远



$\pi\_{ref}$（即初始 SFT 模型 $\pi\_{SFT}$）实践中，语言模型策略 $\pi\_\theta$ 也被初始化为 $\pi\_{SFT}$。

+ **初始状态**：$\pi\_\theta = \pi\_{ref} = \pi\_{SFT}$
+ **训练目标**：在保持与原始 SFT 模型接近的前提下，优化奖励
+ **KL 约束作用**：确保模型不会偏离初始能力太远

**KL 约束：**

KL 散度（Kullback-Leibler 散度）衡量两个概率分布之间的差异：

$$D_{\text{KL}}[P \| Q] = \sum_x P(x) \log \frac{P(x)}{Q(x)}$$

$$D_{\text{KL}}[\pi_\theta(y|x) \| \pi_{SFT}(y|x)] = \mathbb{E}_{y \sim \pi_\theta} \left[ \log \frac{\pi_\theta(y|x)}{\pi_{SFT}(y|x)} \right]$$

这表示：

+ 如果 $\pi\_\theta$ 和 $\pi\_{SFT}$ 相同：KL = 0
+ 如果 $\pi\_\theta$ 偏离 $\pi\_{SFT}$：KL > 0
+ 目标函数中的 $-\beta KL$ 惩罚偏离行为



**KL 散度推导过程详解**

**KL 散度的通用定义（离散型）**：

$$D_{\text{KL}}[P \| Q] = \sum_i P(i) \log \frac{P(i)}{Q(i)}$$

**期望的定义**：  
对于随机变量 $Y \sim P$，函数 $f(Y)$ 的期望定义为：

$$\mathbb{E}_{Y \sim P}[f(Y)] = \sum_{y \in \mathcal{Y}} P(y) f(y)$$

**进行变量映射**

在 RLHF 的语境下，我们要计算的是两个模型策略（概率分布）之间的差异：

+ $P$** (当前分布)**：$\pi\_\theta(y|x)$ —— 模型当前正在学习的策略
+ $Q$** (参考分布)**：$\pi\_{SFT}(y|x)$ —— 原始的 SFT 策略
+ **样本空间**：$y$（模型生成的所有可能的回答序列）

将这些代入通用公式：

$$D_{\text{KL}}[\pi_\theta \| \pi_{SFT}] = \sum_y \pi_\theta(y|x) \log \frac{\pi_\theta(y|x)}{\pi_{SFT}(y|x)}$$

**转化为期望形式**

+ 外层的 $\sum\_y \pi\_\theta(y|x)(\cdots)$ 表示我们在按照 $\pi\_\theta(y|x)$ 的概率分布对后面的项进行加权平均
+ 括号里的项 $\log \frac{\pi\_\theta(y|x)}{\pi\_{SFT}(y|x)}$ 就是要计算期望的对象

因此，根据期望的定义 $ \sum P \cdot f = \mathbb{E}_P[f] $，可以直接写成：

$$D_{\text{KL}}[\pi_\theta \| \pi_{SFT}] = \mathbb{E}_{y \sim \pi_\theta(y|x)} \left[ \log \frac{\pi_\theta(y|x)}{\pi_{SFT}(y|x)} \right]$$

**为什么实践中要写成期望形式？**

在深度学习和强化学习中，写成期望形式有重大的工程意义：

**无法全量求和**：  
对于语言模型，可能的回答 $y$ 的组合是天文数字（由词表大小和序列长度决定），我们根本无法遍历所有的 $y$ 来计算那个 $\sum$ 符号。

**蒙特卡洛采样（Monte Carlo Sampling）**：  
期望形式告诉我们，虽然不能遍历所有结果，但可以通过采样来近似：让模型 $\pi\_\theta$ 生成（Sample）一批句子 $y$,计算这些句子的 $\log \pi\_\theta(y|x) - \log \pi\_{SFT}(y|x)$,取这些值的平均值，就是对 KL 散度的无偏估计

**对齐目标函数**：  
RL 的目标函数本身就是最大化期望奖励 $\mathbb{E}\_{y \sim \pi\_\theta}[r(x,y)]$。将 KL 约束也写成期望形式，就可以合并到一个大括号里：

$$\max_{\pi_\theta} \mathbb{E}_{y \sim \pi_\theta} \left[ r(x,y) - \beta \log \frac{\pi_\theta(y|x)}{\pi_{SFT}(y|x)} \right]$$

这样，模型在每一轮训练采样时，就可以同时优化奖励并计算 KL 惩罚。



**为什么这个目标函数不可导？**

语言模型是**离散生成**的：

+ 输出是 token 序列 $y = (y_1, y_2, \ldots)$
+ 采样 / argmax 都是离散操作

无法像普通监督学习那样，对 $\mathbb{E}\_{y \sim \pi\_\theta(y|x)}[r(x,y)]$ 直接对 $\theta$ 求梯度。

👉 所以不能用反向传播直接优化奖励  
👉 必须用强化学习（policy gradient）

这就是为什么要用 REINFORCE / PPO。

**KL 约束是怎么变成"奖励"的？**



**关键一步是：把公式 (3) KL 项写成 log-prob 的形式**

对离散策略：

$$D_{\text{KL}}(\pi_\theta \| \pi_{ref}) = \mathbb{E}_{y \sim \pi_\theta} \left[ \log \pi_\theta(y|x) - \log \pi_{ref}(y|x) \right]$$

**代回目标函数**：

$$\mathbb{E}_{y \sim \pi_\theta} \left[ r_\phi(x, y) - \beta \left( \log \pi_\theta(y|x) - \log \pi_{ref}(y|x) \right) \right]$$

**于是可以定义一个新的 reward**：

$$r(x, y) = r_\phi(x, y) - \beta \left( \log \pi_\theta(y|x) - \log \pi_{ref}(y|x) \right)$$

即论文中的：

$$r(x, y) = r_\phi(x, y) - \left( \log \pi_\theta(y|x) - \log \pi_{ref}(y|x) \right)$$

（论文里通常把 $\beta$ 省略或吸收到系数里）

**转换后的简化目标函数**

通过这个变换，原来的约束优化问题变成了无约束的奖励最大化问题：

$$\max_{\pi_\theta} \mathbb{E}_{x \sim \mathcal{D}, y \sim \pi_\theta(y|x)} \left[ r(x, y) \right]$$

其中 $r(x, y)$ 已经包含了 KL 惩罚项。

---
