# 硬體參考資料

> 最後確認日期：2026-03-27

## 概述

SO-101 機械臂及相關硬體的規格、設定指南和購買資訊。

## 核心資源（必讀）

- [Seeed Studio SO-101 Wiki（本套件適用）](https://wiki.seeedstudio.com/cn/lerobot_so100m/) — **我們的版本**，組裝、校正、遙操作、訓練完整教學
- [LeRobot SO-101 官方文件](https://huggingface.co/docs/lerobot/so101) — HuggingFace 官方指南（指令相容，但硬體配置略有不同）
- [Feetech STS3215 Datasheet](https://www.feetechrc.com/STS3215.html) — SO-101 使用的伺服馬達規格
- [Intel RealSense D435 文件](https://dev.intelrealsense.com/docs/d400-series) — 深度攝影機 SDK 和規格
- [NVIDIA Jetson Orin Nano Developer Kit](https://developer.nvidia.com/embedded/jetson-orin-nano-developer-kit) — 邊緣運算平台設定指南

## 延伸資源

### 3D 列印

- [TheRobotStudio/SO-ARM100 STL](https://github.com/TheRobotStudio/SO-ARM100) — SO-ARM100 原始 3D 列印檔案
- [XLeRobot 3D 列印件](https://github.com/Vector-Wangel/XLeRobot/tree/main/hardware) — XLeRobot 專案的硬體列印件

### Seeed Studio 套件規格

| 部件 | 數量 | 備註 |
|------|------|------|
| 舵機 (STS3215) | 12 | Leader 6 顆（混合減速比）+ Follower 6 顆（1:345） |
| 舵機驅動板 | 2 | 各臂一塊 |
| USB-C 線纜 | 2 | 資料傳輸用 |
| 電源適配器 | 2 | 標準版全 5V；Pro 版 Leader 5V / Follower 12V |
| 3D 列印桌面夾具 | 4 | 固定用 |

電源接口：5.5mm x 2.1mm DC barrel jack

### 接線與控制板

- Seeed Studio 舵機驅動板 — 套件隨附，透過 3-pin TTL 串接馬達
- [Seeed Wiki 接線說明](https://wiki.seeedstudio.com/cn/lerobot_so100m/#%E6%A0%A1%E5%87%86%E8%88%B5%E6%9C%BA%E5%B9%B6%E7%BB%84%E8%A3%85%E6%9C%BA%E6%A2%B0%E8%87%82) — 舵機校準接線圖

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
