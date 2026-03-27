# 模擬環境參考資料

> 最後確認日期：2026-03-27

## 概述

機器人模擬器和 Sim-to-Real 遷移相關的工具與教學。模擬環境讓你可以在虛擬世界中大量訓練，減少真實硬體的損耗風險。

## 核心資源（必讀）

- [NVIDIA Isaac Sim](https://developer.nvidia.com/isaac/sim) — 基於 Omniverse 的高保真模擬器（Apache 2.0）
- [NVIDIA Isaac Lab](https://developer.nvidia.com/isaac-lab) — GPU 加速機器人學習框架（BSD-3-Clause）
- [MuJoCo](https://mujoco.org/) — DeepMind 物理模擬器

## 延伸資源

### NVIDIA 生態系

- [Isaac Sim 文件](https://docs.omniverse.nvidia.com/isaacsim/latest/) — 完整的 Isaac Sim 使用指南
- [Isaac Lab GitHub](https://github.com/isaac-sim/IsaacLab) — Isaac Lab 原始碼與範例
- Isaac ROS — 針對 NVIDIA 平台最佳化的 ROS 套件
- [Omniverse](https://www.nvidia.com/omniverse/) — NVIDIA 的 3D 協作與模擬平台
- Newton 物理引擎（Beta） — NVIDIA 下一代物理引擎

### XLeRobot 模擬整合

- [MuJoCo 模擬](https://github.com/Vector-Wangel/XLeRobot/tree/main/simulation) — XLeRobot 的 MuJoCo 模擬環境
- [Isaac Sim USD](https://github.com/Vector-Wangel/XLeRobot/tree/main/simulation) — XLeRobot 的 Isaac Sim 資產
- [ManiSkill](https://github.com/Vector-Wangel/XLeRobot/tree/main/simulation) — XLeRobot 的 ManiSkill 整合

### Sim-to-Real

- Domain Randomization 指南 — 透過隨機化提升模擬到真實的遷移效果
- Isaac ROS 的 RL Policy 部署教學 — 將強化學習策略部署到真實機器人

### 學習指南

- [Embodied-AI-Guide 模擬器章節](https://github.com/TianxingChen/Embodied-AI-Guide) — 各模擬器的比較與入門教學

## 備註

- Isaac Sim 僅支援 Linux（Ubuntu 22.04）。
- MuJoCo 跨平台，適合快速原型。
- 建議先 MuJoCo 再 Isaac Sim。
