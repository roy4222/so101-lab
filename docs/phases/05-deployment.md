# Phase 05 — 部署

## 1. 目標

將訓練好的策略模型部署到真實機器手臂，驗證任務成功率，並設定 Jetson Orin Nano 邊緣推論環境。

## 2. 前置條件

- [Phase 04 — 訓練](./04-training.md) 已完成
- Jetson Orin Nano 已完成基本設定（JetPack SDK）
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 訓練好的 ACT checkpoint |
| **輸出** | 可運行的推論系統、任務成功率報告、Jetson 部署設定 |

## 4. 步驟

### Step 1 — 本地推論測試

載入 checkpoint 控制 Follower 執行 pick & place。LeRobot 的推論是透過 `lerobot-record` 加上 `--control.policy.path` 參數來執行：

```bash
lerobot-record \
  --robot.type=so101_follower --robot.port=/dev/ttyXXX --robot.id=my_follower \
  --robot.cameras.front.type=realsense \
  --robot.cameras.front.fps=30 \
  --robot.cameras.front.width=640 \
  --robot.cameras.front.height=480 \
  --control.policy.path=path/to/checkpoint \
  --dataset.repo_id=<your-username>/so101_eval_v1 \
  --dataset.num_episodes=10
```

> **說明**：LeRobot 目前沒有獨立的 `lerobot-infer` 指令，推論是透過 record 路徑搭配 policy checkpoint 完成。這同時會錄下推論過程的資料，方便後續分析。

測試 10+ 次，記錄每次成功 / 失敗及失敗原因。

### Step 2 — 非同步推論架構

```bash
uv pip install "lerobot[async]"
```

架構說明：

- **PolicyServer**（GPU 機器）：負責模型推論
- **RobotClient**（機器人端）：負責感測器讀取和馬達控制

非同步推論將推論和執行分離到不同程序（可在不同機器上）：

```bash
# GPU 機器上啟動 PolicyServer（參考 LeRobot async 文件）
python -m lerobot.scripts.server.policy_server \
  --policy.path=path/to/checkpoint

# 機器人端啟動 RobotClient
python -m lerobot.scripts.server.robot_client \
  --robot.type=so101_follower --robot.port=/dev/ttyXXX --robot.id=my_follower \
  --robot.cameras.front.type=realsense
```

> **注意**：非同步推論的具體指令格式可能隨 LeRobot 版本更新而變化，執行前請確認 [LeRobot async 文件](https://huggingface.co/docs/lerobot) 的最新版本。

### Step 3 — Jetson 部署

在 Jetson Orin Nano 上：

```bash
# ARM 架構安裝 LeRobot
pip install lerobot "lerobot[feetech]"
```

- 設定 Jetson 為 **RobotClient**
- PolicyServer 可跑在工作站或 Jetson 上
  - Jetson 上推薦使用 SmolVLA（約 2GB VRAM）

### Step 4 — 成功率測試

連續執行 20 次 pick & place：

- 記錄每次結果（成功 / 失敗 / 失敗原因）
- 全程錄影留存
- 計算成功率

## 5. 驗證方式

- 本地推論成功執行
- 非同步架構正常運作
- Jetson 可作為 RobotClient
- 20 次測試中 > 80% 成功

## 6. 關鍵連結

- [LeRobot 部署文件](https://huggingface.co/docs/lerobot)
- [Jetson Orin Nano 開發者指南](https://developer.nvidia.com/embedded/jetson-orin-nano)
- [SmolVLA 模型](https://huggingface.co/lerobot/smolvla)

## 7. 成功標準

- [ ] 本地推論可執行
- [ ] Pick & place 成功率 > 80%（20 次測試）
- [ ] 非同步推論架構可運行
- [ ] Jetson 可作為 RobotClient
- [ ] 測試已記錄，含影片

## 8. 產出物

| 產出物 | 說明 |
|--------|------|
| 成功率測試報告 | 20 次測試結果統計 |
| 測試影片 | 每次測試的錄影 |
| Jetson 部署設定 | 安裝步驟、設定檔 |
| 非同步推論網路設定 | PolicyServer / RobotClient 設定 |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| Jetson CUDA 相容性 | Jetson 的 CUDA 版本可能與 PyTorch 不完全相容 | 使用 NVIDIA 官方 PyTorch wheel for Jetson |
| 非同步網路延遲 | PolicyServer 與 RobotClient 間的網路延遲 | 使用有線連接，確認延遲 < 20ms |
| ARM 套件相容性 | 部分 Python 套件不支援 ARM 架構 | 提前測試關鍵套件安裝 |
| Jetson 8GB VRAM 限制 | 大型模型無法載入 | 使用 SmolVLA 或量化模型 |
