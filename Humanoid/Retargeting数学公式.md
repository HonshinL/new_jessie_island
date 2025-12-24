

 
# Dex Retargeting数学公式详解

## 1. 位置重定向公式

### 1.1 基础坐标系转换
$$
\mathbf{p}_{\text{robot}} = \mathbf{R} \cdot \mathbf{p}_{\text{human}} + \mathbf{t}
$$
其中：
- $\mathbf{p}_{\text{human}}$：人手关键点在人手坐标系中的位置向量
- $\mathbf{p}_{\text{robot}}$：机器人手对应关键点在机器人坐标系中的目标位置向量
- $\mathbf{R}$：旋转矩阵（将人手坐标系转换到机器人坐标系）
- $\mathbf{t}$：平移向量（人手坐标系原点到机器人坐标系原点的偏移）

### 1.2 逆运动学求解
位置重定向的核心是求解机器人手的关节角度，使末端执行器达到目标位置：
$$
\boldsymbol{\theta}_{\text{robot}} = \text{IK}(\mathbf{p}_{\text{robot}}, \mathbf{q}_{\text{robot}}^{\text{init}})
$$
其中：
- $\boldsymbol{\theta}_{\text{robot}}$：机器人手的关节角度向量
- $\text{IK}()$：逆运动学求解函数
- $\mathbf{q}_{\text{robot}}^{\text{init}}$：机器人手的初始关节角度（用于数值迭代）

### 1.3 多关键点优化
当需要匹配多个关键点时，使用优化方法最小化位置误差：
$$
\boldsymbol{\theta}_{\text{robot}}^* = \arg\min_{\boldsymbol{\theta}} \sum_{i=1}^{n} w_i \left\| \mathbf{f}_i(\boldsymbol{\theta}) - \mathbf{p}_{\text{robot},i} \right\|^2
$$
其中：
- $n$：关键点数量
- $w_i$：第$i$个关键点的权重
- $\mathbf{f}_i(\boldsymbol{\theta})$：机器人手第$i$个关键点的正运动学函数
- $\mathbf{p}_{\text{robot},i}$：第$i$个关键点的目标位置

## 2. 向量重定向公式

### 2.1 运动向量提取
从人手姿态序列中提取运动向量：
$$
\mathbf{v}_{\text{human}} = \frac{d\mathbf{p}_{\text{human}}(t)}{dt}
$$
其中：
- $\mathbf{v}_{\text{human}}$：人手关键点的运动向量
- $t$：时间

### 2.2 向量坐标系转换
将人手运动向量转换到机器人坐标系：
$$
\mathbf{v}_{\text{robot}} = \mathbf{R} \cdot \mathbf{v}_{\text{human}}
$$
其中：
- $\mathbf{v}_{\text{robot}}$：机器人坐标系中的运动向量
- $\mathbf{R}$：与位置重定向相同的旋转矩阵

### 2.3 运动学缩放
根据机器人手的运动学特性调整运动向量的幅度：
$$
\mathbf{v}_{\text{robot}}^{\text{scaled}} = k \cdot \mathbf{v}_{\text{robot}}
$$
其中：
- $\mathbf{v}_{\text{robot}}^{\text{scaled}}$：缩放后的运动向量
- $k$：缩放因子（考虑机器人手与人类手的尺寸和运动范围差异）

### 2.4 关节速度计算
将运动向量转换为机器人手的关节速度：
$$
\dot{\boldsymbol{\theta}}_{\text{robot}} = \mathbf{J}^\dagger(\boldsymbol{\theta}) \cdot \mathbf{v}_{\text{robot}}^{\text{scaled}}
$$
其中：
- $\dot{\boldsymbol{\theta}}_{\text{robot}}$：机器人手的关节速度向量
- $\mathbf{J}^\dagger(\boldsymbol{\theta})$：雅可比矩阵的伪逆（与当前关节角度$\boldsymbol{\theta}$相关）

## 3. 姿态优化公式

### 3.1 关节限制约束
确保生成的关节角度在物理限制范围内：
$$
\boldsymbol{\theta}_{\text{min}} \leq \boldsymbol{\theta}_{\text{robot}} \leq \boldsymbol{\theta}_{\text{max}}
$$
其中：
- $\boldsymbol{\theta}_{\text{min}}$：关节角度下限向量
- $\boldsymbol{\theta}_{\text{max}}$：关节角度上限向量

### 3.2 平滑性约束
最小化关节角度的变化率以确保动作平滑：
$$
\min_{\boldsymbol{\theta}(t)} \int_{t_0}^{t_1} \left\| \frac{d^2\boldsymbol{\theta}(t)}{dt^2} \right\|^2 dt
$$
其中：
- $t_0, t_1$：时间区间

### 3.3 综合优化目标
结合位置误差、关节限制和平滑性的综合优化：
$$
\begin{align*}
\boldsymbol{\theta}^* = \arg\min_{\boldsymbol{\theta}} &\left\{ \sum_{i=1}^{n} w_i \left\| \mathbf{f}_i(\boldsymbol{\theta}) - \mathbf{p}_{\text{robot},i} \right\|^2 + \right. \\
& \left. \lambda_1 \left\| \boldsymbol{\theta} - \boldsymbol{\theta}_{\text{prev}} \right\|^2 + \lambda_2 \left\| \ddot{\boldsymbol{\theta}} \right\|^2 \right\} \\
\text{subject to } & \boldsymbol{\theta}_{\text{min}} \leq \boldsymbol{\theta} \leq \boldsymbol{\theta}_{\text{max}}
\end{align*}
$$
其中：
- $\boldsymbol{\theta}_{\text{prev}}$：上一时刻的关节角度
- $\ddot{\boldsymbol{\theta}}$：关节加速度向量
- $\lambda_1, \lambda_2$：平滑性权重系数

## 4. MANO模型到机器人手的映射

### 4.1 MANO模型姿态参数
MANO模型使用以下参数表示人手姿态：
$$
\mathbf{p}_{\text{mano}} = \mathbf{W} \cdot (\mathbf{V} + \beta \cdot \mathbf{B}_s + \theta \cdot \mathbf{B}_p)
$$
其中：
- $\mathbf{V}$：基础手模型顶点
- $\beta$：形状参数（10个参数）
- $\theta$：姿态参数（24个关节角度）
- $\mathbf{B}_s$：形状混合矩阵
- $\mathbf{B}_p$：姿态混合矩阵
- $\mathbf{W}$：蒙皮权重矩阵

### 4.2 MANO到机器人手的映射
从MANO模型的姿态参数映射到机器人手的关节角度：
$$
\boldsymbol{\theta}_{\text{robot}} = \mathbf{M} \cdot \theta_{\text{mano}} + \mathbf{b}
$$
其中：
- $\theta_{\text{mano}}$：MANO模型的姿态参数
- $\mathbf{M}$：映射矩阵（学习得到或手动定义）
- $\mathbf{b}$：偏置向量

## 5. 实时控制公式

### 5.1 闭环反馈控制
为了实现实时控制，使用闭环反馈调整关节角度：
$$
\boldsymbol{\theta}(t+1) = \boldsymbol{\theta}(t) + K_p \cdot (\boldsymbol{\theta}_{\text{desired}}(t) - \boldsymbol{\theta}(t)) + K_d \cdot \dot{\boldsymbol{\theta}}(t)
$$
其中：
- $\boldsymbol{\theta}(t)$：当前关节角度
- $\boldsymbol{\theta}_{\text{desired}}(t)$：期望关节角度
- $\dot{\boldsymbol{\theta}}(t)$：关节角速度
- $K_p$：比例增益矩阵
- $K_d$：微分增益矩阵

这些公式构成了Dex Retargeting模块的数学基础，实现了从人类手部姿态到机器人手部关节角度的精确转换，支持位置重定向和向量重定向两种模式。
        