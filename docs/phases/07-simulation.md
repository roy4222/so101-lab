# Phase 07 — 模擬環境

## 1. 目標

在 NVIDIA Isaac Sim 或 MuJoCo 中建立 SO-101 的數位孿生，使用 Isaac Lab 進行 RL 訓練，完成 Sim-to-Real 遷移。

## 2. 前置條件

- [Phase 04 — 訓練](./04-training.md) 已完成
- SO-101 URDF 模型
- 訓練工作站（Linux, 5x RTX 8000）
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | SO-101 URDF、ACT checkpoint（baseline） |
| **輸出** | 模擬環境、RL pipeline、Sim-to-Real 結果 |

## 4. 步驟

> **Phase 4 完成後補充詳細步驟。**

預期方向：

- 安裝 Isaac Sim（Linux Ubuntu 22.04）
- 匯入 URDF 模型
- 設定物理參數（摩擦力、重量、關節限制）
- Isaac Lab RL 訓練
- Domain Randomization（材質、光線、物件位置）
- Sim-to-Real 遷移與評估
- 備選方案：MuJoCo（更輕量、跨平台）

## 5. 驗證方式

- 數位孿生可正常運行
- 模擬中訓練的策略可在實體臂上執行
- Sim-to-Real gap 在可接受範圍

## 6. 關鍵連結

- [Isaac Sim 文件](https://docs.omniverse.nvidia.com/isaacsim/)
- [Isaac Lab 快速入門](https://isaac-sim.github.io/IsaacLab/)
- [XLeRobot 模擬環境](https://github.com/XLeRobot)
- [Embodied-AI-Guide 模擬器章節](https://github.com/Embodied-AI-Guide)

## 7. 成功標準

- [ ] 數位孿生可運行
- [ ] 模擬訓練策略可部署到實體臂
- [ ] Sim-to-Real gap 可接受（不低於實體訓練 70%）

## 8. 產出物

| 產出物 | 說明 |
|--------|------|
| URDF / USD 模型 | SO-101 的模擬模型 |
| 模擬環境設定檔 | 物理參數、場景設定 |
| RL checkpoint | 模擬中訓練的策略模型 |
| Sim-to-Real 測試報告 | 模擬 vs 實體的成功率對比 |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| Isaac Sim 僅支援 Linux | macOS 無法使用 Isaac Sim | 使用 Linux 工作站，或改用 MuJoCo |
| URDF 精度 | URDF 模型與實體的差異 | 實測後校正模型參數 |
| Sim-to-Real gap | 模擬與真實環境的差距 | Domain Randomization、漸進式遷移 |
| 學習曲線高 | Isaac Sim / Isaac Lab 學習成本較高 | 先從簡單場景開始，參考官方教學 |
