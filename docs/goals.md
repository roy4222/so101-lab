# 願景與目標

## 願景

打造一台具備**雙臂操作**與**自主移動**能力的家庭機器人，從 LeRobot SO-101 單臂出發，逐步演進至 XLeRobot 移動平台。

本專案同時是一份**可重現的學習紀錄**，讓其他想入門機器手臂的開發者能按照相同路徑，從零開始完成整個流程。

---

## 階段目標

### Phase 1-5：掌握 LeRobot 全流程（單臂）

| Phase | 學會什麼 |
|-------|---------|
| 01 硬體組裝 | Feetech 伺服馬達控制、UART 通訊、機械臂校正 |
| 02 遙操作 | Leader-Follower 即時控制、RealSense D435 彩色+深度串流 |
| 03 資料收集 | LeRobotDataset 格式、資料品質管理、HuggingFace Hub 上傳 |
| 04 訓練 | ACT / Diffusion Policy 訓練、多 GPU 加速、實驗追蹤 |
| 05 部署 | 本地推論、Pi 5 邊緣部署、非同步 PolicyServer/RobotClient |

### Phase 6：雙臂協調

- 將 Leader 臂改裝為第二隻 Follower 臂
- 學會雙臂校正、協調控制、解析逆運動學

### Phase 7：模擬環境 + Sim-to-Real

- 在 Isaac Sim / MuJoCo 中建立 SO-101 數位孿生
- 使用 Isaac Lab 進行 RL 訓練
- 完成 Sim-to-Real 遷移

### Phase 8：XLeRobot 移動平台

- 將雙臂裝上輪式底盤（IKEA RASKOG 推車）
- 整合導航（LIDAR / RGBD）
- 實現移動 + 操作的端到端任務

---

## 成功標準

每個階段需滿足兩類指標：

### 任務成功率

| Phase | 指標 |
|-------|------|
| 01 | `lerobot-calibrate` 通過，所有 6 個關節可獨立控制 |
| 02 | 遙操作主觀連續可控，實測 latency 有記錄 |
| 03 | 完成 50+ 回合 pick & place 資料集 |
| 04 | loss 收斂 + validation/replay/offline eval 有可解釋結果 |
| 05 | 機器手臂執行訓練任務成功率 > 80% |
| 06 | 雙臂可獨立或協同執行任務 |
| 07 | 模擬訓練的策略可在實體臂上執行 |
| 08 | 機器人可移動到指定位置並執行操作任務 |

### Reproducibility 指標

每個階段完成後必須留下：
- **可重新執行的指令流程**：從零開始照做就能重現結果
- **固定的資料集版本**：資料集有版本號，已上傳 HuggingFace Hub
- **可重現的訓練設定**：超參數、隨機種子、環境版本皆記錄在案
- **驗證影片或 metrics**：可視化的成功證據

---

## 硬體演進圖

```
Phase 1-5                Phase 6                   Phase 7-8
┌─────────────┐     ┌─────────────────┐     ┌──────────────────────┐
│  SO-101     │     │  雙臂桌面版      │     │  XLeRobot 移動機器人  │
│  單臂       │ ──→ │  2x SO-101      │ ──→ │  雙臂 + 輪式底盤      │
│  Leader +   │     │  雙 Follower    │     │  + 導航 + VLA         │
│  Follower   │     │  + RealSense    │     │  + Pi 5 邊緣控制      │
└─────────────┘     └─────────────────┘     └──────────────────────┘
```

---

## 硬體清單

| 設備 | 用途 | 啟用階段 |
|------|------|---------|
| SO-101 Leader 臂 | 遙操作輸入 → Phase 6 改裝為 Follower | Phase 1 |
| SO-101 Follower 臂 | 任務執行 | Phase 1 |
| Intel RealSense D435 | 彩色 + 深度感知 | Phase 2 |
| 5x Quadro RTX 8000 (240GB) | 模型訓練 | Phase 4 |
| Raspberry Pi 5 8GB | 主執行機（遙操作、校正、錄資料、邊緣控制） | Phase 1 |
| IKEA RASKOG 推車 + 輪組 | 移動底盤 | Phase 8 |
