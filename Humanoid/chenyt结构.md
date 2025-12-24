---
tags:
  - chenyt
aliases:
---




## 项目结构分析

### 一、项目概述
这是一个机器人灵巧手控制项目，集成了手部检测、重定向、策略学习等功能。主要流程是从视频中提取人手姿态，重定向到机器人手，并训练扩散策略控制机器人手执行任务。

### 二、核心模块

#### 1. WiLoR (手部定位与重建)
- 位置: `WiLoR/`, `combine/WiLoR/`
- 功能: 端到端 3D 手部定位与重建
- 特性:
  - 从图像/视频检测和重建人手 3D 姿态
  - 使用 MANO 手部模型
  - 输出手部关键点、网格和姿态参数

#### 2. Dex Retargeting (手部重定向)
- 位置: `dex_retargeting/`, `combine/dex_retargeting/`
- 功能: 将人手动作映射到机器人手
- 特性:
  - 支持多种机器人手（Shadow Hand, DexHand 等）
  - 位置重定向和向量重定向两种模式
  - 用于遥操作和模仿学习

#### 3. Diffusion Policy (扩散策略学习)
- 位置: `diffusion_policy-main_lhh/`
- 功能: 基于扩散模型的机器人策略学习
- 特性:
  - 支持图像和低维状态空间
  - 用于模拟和真实机器人训练
  - 支持 Push-T 等任务

#### 4. SKIL (技能学习)
- 位置: `SKIL/`
- 功能: 基于扩散策略的机器人技能学习框架
- 特性:
  - 支持图像输入
  - 包含数据处理、训练和评估
  - 针对灵巧手抓取任务（pinch_grasp, power_grasp）

#### 5. Combine (组合模块)
- 位置: `combine/`
- 功能: 整合 WiLoR 和 Dex Retargeting 的管道
- 主要脚本:
  - `trial_detect_from_video*.py`: 从视频中检测人手并重定向到机器人手的多个版本迭代
  - `visualization.py`: 可视化工具
  - `transformation.py`: 坐标变换工具

### 三、根目录脚本功能

#### 数据处理相关
- `action_generation.py`: 生成动作数据，将 PKL 文件和视频转换为 Zarr 格式的 replay buffer
- `replay_buffer.py`: 创建和管理 replay buffer 用于策略训练
- `pkl_multiplication.py`, `pkl_multiplication_2.py`: PKL 数据增强/复制
- `txt_to_pkl.py`: 文本数据转 PKL 格式
- `npy_read.py`: 读取 NumPy 数组数据

#### 硬件控制相关
- `gripper_hand_for_retarget_vedio.py`: 通过串口控制灵巧手执行重定向后的动作
- `gripper_hand_for_retarget_vedio_bluetooth.py`: 蓝牙版本的手部控制器

#### 评估相关
- `diffusion_policy-main_lhh/eval_video*.py`: 多个版本的视频评估脚本
- `SKIL/eval_video*.py`: SKIL 框架的视频评估脚本
- `SKIL/eval.py`: 通用评估脚本

### 四、工作流程

1. 数据采集阶段
   ```
   视频 → WiLoR检测 → 人手3D姿态 → Dex Retargeting → 机器人手动作
   ```

2. 数据预处理阶段
   ```
   PKL/视频文件 → action_generation.py → Zarr格式ReplayBuffer
   ```

3. 策略训练阶段
   ```
   ReplayBuffer → Diffusion Policy训练 → 训练好的策略模型
   ```

4. 评估/执行阶段
   ```
   新视频 → 策略推理 → 机器人手动作执行
   ```

### 五、目录结构总结

```
chenyt/
├── WiLoR/                    # 手部检测与重建（原始版本）
├── combine/                   # 整合模块
│   ├── WiLoR/                # WiLoR集成版本
│   ├── dex_retargeting/      # 重定向模块
│   ├── dexhand_v1_3/         # 机器人手URDF模型
│   └── trial_detect_from_video*.py  # 视频处理迭代版本
├── dex_retargeting/           # 重定向模块（独立版本）
├── diffusion_policy-main_lhh/ # 扩散策略框架
├── SKIL/                      # 技能学习框架
│   ├── data/                  # 训练数据（Zarr格式）
│   ├── diffusion_policy/      # 策略实现
│   └── evaluation/            # 评估结果
├── sim-web-visualizer/        # Web可视化工具
└── [根目录脚本]               # 数据处理和控制脚本
```

### 六、应用场景
1. 遥操作: 人手动作实时映射到机器人手
2. 模仿学习: 从演示视频学习抓取技能
3. 策略学习: 训练灵巧手操作策略（如 pinch grasp, power grasp）

项目结合了计算机视觉（手部检测）、运动学（重定向）和强化学习/模仿学习（策略训练），形成完整的机器人手控制研究框架。