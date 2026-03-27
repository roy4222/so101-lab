# 軟體工具參考資料

> 最後確認日期：2026-03-27

## 概述

本專案使用的軟體框架、工具和平台。

## 核心資源（必讀）

- [Seeed Projects LeRobot Fork](https://github.com/Seeed-Projects/lerobot.git) — **本專案使用的穩定版**，經 Seeed 驗證
- [LeRobot 官方文件](https://huggingface.co/docs/lerobot/index) — 完整的 LeRobot 使用說明與 API 文件
- [LeRobot GitHub](https://github.com/huggingface/lerobot) — LeRobot 原始碼與 issue 追蹤（官方最新版，可能有 breaking change）
- [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) — 擴展版 LeRobot，支援雙臂與移動平台

## 延伸資源

### 訓練加速

- [HuggingFace Accelerate](https://huggingface.co/docs/accelerate/index) — 分散式訓練與混合精度加速
- [WandB](https://wandb.ai/) — 實驗追蹤與視覺化平台

### 資料管理

- [HuggingFace Hub](https://huggingface.co/docs/hub/index) — 模型與資料集託管平台
- LeRobotDataset 格式 — LeRobot 專用的資料集結構與格式規範

### 邊緣部署

- [NVIDIA Isaac ROS](https://nvidia-isaac-ros.github.io/) — 針對 NVIDIA 平台最佳化的 ROS 套件
- Isaac Lab Python API — 機器人學習框架的 Python 介面

### 機器人中介軟體

- [ROS 2 Humble](https://docs.ros.org/en/humble/) — 機器人作業系統長期支援版本
- XLeRobot Web Dashboard — 基於網頁的機器人控制與監控介面

### 攝影機 SDK

- pyrealsense2 — Intel RealSense 的 Python 綁定
- OpenCV — 電腦視覺基礎函式庫

## 環境管理

- **Jetson**：使用 Miniconda（conda 環境 `lerobot`，Python 3.10）
- **WSL2 開發機**：可用 uv 或 conda
- LeRobot 版本更新頻繁，本專案釘選 Seeed Fork 以確保穩定
- Jetson JetPack 6.0/6.1（暫不支援 6.2）
- PyTorch GPU 版需手動安裝（pip install lerobot 會覆蓋成 CPU 版）
