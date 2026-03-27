# 演算法參考資料

> 最後確認日期：2026-03-27

## 概述

機器手臂操作學習的核心演算法，從模仿學習到強化學習到 VLA 模型。

## 核心資源（必讀）

- [ACT 論文](https://tonyzhaozh.github.io/aloha/) — Action Chunking Transformer，LeRobot 主要訓練策略
- [Diffusion Policy](https://diffusion-policy.cs.columbia.edu/) — 基於 diffusion 的策略學習
- [LeRobot 支援的策略清單](https://huggingface.co/docs/lerobot/index) — 官方支援的所有策略總覽

## 延伸資源

### VLA 模型

- SmolVLA — 約 2GB，適合 Jetson 邊緣部署
- Pi0 / Pi0.5 — Physical Intelligence 的 VLA 模型
- GR00T N1.5 — NVIDIA 的通用機器人基礎模型
- [OpenVLA](https://openvla.github.io/) — 開源 Vision-Language-Action 模型

### 強化學習

- HIL-SERL — Human-in-the-Loop Sample Efficient Reinforcement Learning
- [TDMPC](https://www.tdmpc2.com/) — Temporal Difference Model Predictive Control

### 其他策略

- [VQ-BeT](https://sjlee.cc/vq-bet/) — Vector-Quantized Behavior Transformer
- DP3 — 3D Diffusion Policy

### 學習指南

- [Embodied-AI-Guide 演算法章節](https://github.com/TianxingChen/Embodied-AI-Guide) — 具身智慧演算法的系統性整理

## 備註

- 建議從 ACT 開始，最成熟且社群經驗最多。
- eta_0.1 發現 ResNet18 + 獨立編碼器 > ResNet34。
