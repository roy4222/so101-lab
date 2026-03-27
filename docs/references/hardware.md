# 硬體參考資料

> 最後確認日期：2026-03-27

## 概述

SO-101 機械臂及相關硬體的規格、設定指南和購買資訊。

## 核心資源（必讀）

- [LeRobot SO-101 官方文件](https://huggingface.co/docs/lerobot/so101) — 組裝步驟、BOM 清單、馬達設定的權威指南
- [Feetech STS3215 Datasheet](https://www.feetechrc.com/STS3215.html) — SO-101 使用的伺服馬達規格
- [Intel RealSense D435 文件](https://dev.intelrealsense.com/docs/d400-series) — 深度攝影機 SDK 和規格
- [NVIDIA Jetson Orin Nano Developer Kit](https://developer.nvidia.com/embedded/jetson-orin-nano-developer-kit) — 邊緣運算平台設定指南

## 延伸資源

### 3D 列印

- [TheRobotStudio/SO-ARM100 STL](https://github.com/TheRobotStudio/SO-ARM100) — SO-ARM100 原始 3D 列印檔案
- [XLeRobot 3D 列印件](https://github.com/Vector-Wangel/XLeRobot/tree/main/hardware) — XLeRobot 專案的硬體列印件

### 接線與控制板

- Waveshare 伺服馬達控制板 — 用於驅動 Feetech 伺服馬達的控制板
- LeRobot 文件中的接線圖 — 官方文件內附的接線說明

### XLeRobot 硬體改裝

- XLeRobot 硬體 BOM — 雙臂移動平台的完整物料清單
- IKEA RASKOG 推車 — 作為移動底盤的現成方案
- Anker SOLIX C300 DC — 行動電源供電方案
- [RealSense D435 雲台 STL](https://github.com/Vector-Wangel/XLeRobot/tree/main/hardware/camera_connector) — 攝影機固定座 3D 列印檔案

### Jetson 設定

- [JetPack SDK](https://developer.nvidia.com/embedded/jetpack) — NVIDIA Jetson 官方開發套件

## 備註

- RealSense D435 在 macOS 上已知不穩定，建議使用 Linux。
- Jetson 是 ARM 架構，部分套件需額外處理。
