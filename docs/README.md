# SO-101 機器手臂學習專案

從 LeRobot SO-101 單臂機器人出發，逐步演進至 XLeRobot 雙臂移動機器人的完整學習旅程。

---

## Current Focus

| 項目 | 內容 |
|------|------|
| 現在做什麼 | Phase 1 — 硬體組裝與校正 |
| 下一步 | Phase 2 — 遙操作 + RealSense D435 設定 |
| 目前 Blocker | 無 |

---

## 硬體清單

| 設備 | 規格 | 用途 |
|------|------|------|
| SO-101 機械臂 x2 | 6-DOF, Feetech STS3215 | Leader + Follower（之後改雙臂） |
| Intel RealSense D435 | 彩色 + 深度攝影機 | 視覺感知 |
| NVIDIA Jetson Orin Nano | 8GB, SUPER Developer Kit | 邊緣推論 |
| 訓練工作站 | 5x RTX 8000 (240GB), 2x Xeon Gold 6248R, 754GB RAM | 模型訓練 |

---

## 快速導覽

| 文件 | 說明 |
|------|------|
| [願景與目標](goals.md) | 專案願景、階段目標、成功標準 |
| [Roadmap](roadmap.md) | 全階段時程、優先級、依賴關係 |
| [系統架構](architecture/overview.md) | 高層模組圖、資料流、技術細節 |
| [Phase 01 — 硬體組裝](phases/01-hardware-setup.md) | 當前進行中的階段 |
| [參考資料](references/) | 分類整理的外部資源 |
| [開發日誌](journal/_index.md) | 每週進度紀錄 |

---

## 給新手：從哪裡開始？

1. 先讀 [願景與目標](goals.md) 了解這個專案要做什麼
2. 看 [Roadmap](roadmap.md) 了解整體規劃
3. 從 [Phase 01 — 硬體組裝](phases/01-hardware-setup.md) 開始跟著做
4. 遇到問題可以查 [參考資料](references/) 或 [開發日誌](journal/_index.md) 的踩坑紀錄

---

## 最新開發日誌

> 尚未開始記錄，第一篇日誌將在 Phase 1 啟動後撰寫。
