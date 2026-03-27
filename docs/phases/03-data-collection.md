# Phase 03 — 資料收集

## 1. 目標

規劃資料集結構、錄製 50+ 回合的 pick & place 示範資料、驗證資料品質、上傳至 HuggingFace Hub。

## 2. 前置條件

- [Phase 02 — 遙操作](./02-teleoperation.md) 已完成
- HuggingFace 帳號已註冊
- 訓練用物件準備好（方塊、杯子等）
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 穩定的遙操作系統 + 攝影機串流 |
| **輸出** | 已上傳的 LeRobotDataset（含版本號）、資料集品質報告 |

## 4. 步驟

### Step 1 — 規劃資料集

定義資料集結構：

- **任務名**：`pick_and_place_cube`
- **Repo ID**：`<your-username>/so101_pick_and_place_cube`
- **攝影機數量**：1 台 RealSense D435（如有第二台 USB 攝影機可加入，但非必要）
- **FPS**：30
- **每回合長度**：10-20 秒

### Step 2 — 錄製示範資料

```bash
lerobot-record \
  --robot.type=so101_follower --robot.port=/dev/ttyXXX --robot.id=my_follower \
  --teleop.type=so101_leader --teleop.port=/dev/ttyYYY --teleop.id=my_leader \
  --robot.cameras.front.type=realsense \
  --robot.cameras.front.fps=30 \
  --robot.cameras.front.width=640 \
  --robot.cameras.front.height=480 \
  --robot.cameras.front.use_depth=true \
  --dataset.repo_id=<your-username>/so101_pick_and_place_cube \
  --dataset.num_episodes=50
```

錄製要點：

- 總計 **50+ 回合**
- 每回合開始和結束位置保持一致
- 動作自然流暢
- **加入位置擾動**（物件位置 ±3-5cm 變化）
- 參考 eta_0.1 經驗：**40 個有策略變化的回合 > 100 個重複回合**

### Step 3 — 品質檢查

- 回放每個 episode，確認動作完整
- 刪除失敗的回合（夾取失敗、碰撞等）
- 確認影像無損壞、無黑屏

### Step 4 — 上傳 HuggingFace Hub

```bash
huggingface-cli login
# 上傳指令（依 LeRobot 版本而定，通常錄製完成後自動上傳）
```

### Step 5 — 版本化

記錄以下資訊：

- 版本號：`v1`
- 錄製環境：macOS / Linux、Python 版本、LeRobot 版本
- 物件描述：方塊尺寸、顏色、材質
- 攝影機配置：視角、解析度、FPS

## 5. 驗證方式

- 資料集可在 HuggingFace Hub 上瀏覽
- LeRobot dataset viewer 可回放 5+ episode
- 影像清晰、動作完整

```bash
# 回放驗證
lerobot-replay --dataset.repo_id=<your-username>/so101_pick_and_place_cube --episode=0
```

## 6. 關鍵連結

- [LeRobot 資料收集文件](https://huggingface.co/docs/lerobot/so101)
- [HuggingFace Hub](https://huggingface.co/)
- [LeRobot Dataset 格式](https://github.com/huggingface/lerobot#datasets)

## 7. 成功標準

- [ ] 50+ 回合有效示範
- [ ] 已上傳 HuggingFace Hub 並有版本號
- [ ] 至少 1 個攝影機視角（RealSense D435），如有第二台可加入
- [ ] 錄製指令可重現
- [ ] 品質目視檢查通過

## 8. 產出物

| 產出物 | 說明 |
|--------|------|
| HuggingFace Dataset repo | `<your-username>/so101_pick_and_place_cube` |
| 資料集元資料 | 版本號、錄製環境、物件描述 |
| 品質檢查紀錄 | 每個 episode 的檢查結果 |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| 人類操作變異性 | 操作者動作不一致導致資料品質差 | 正式錄製前先練習 10+ 次 |
| 儲存空間不足 | 影片資料量很大（尤其含深度） | 提前確認磁碟空間，建議預留 50GB+ |
| 多攝影機幀同步 | 不同攝影機的幀可能不完全同步 | 使用 LeRobot 內建同步機制，確認 FPS 設定一致 |
