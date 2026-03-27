# Phase 02 — 遙操作

## 1. 目標

建立 Leader-Follower 即時遙操作系統，整合 Intel RealSense D435 攝影機（彩色 + 深度），確保控制迴圈穩定。

## 2. 前置條件

- [Phase 01 — 硬體組裝](./01-hardware-setup.md) 已完成
- Intel RealSense D435 已連接
- RealSense SDK 已安裝
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 已校正的雙臂、RealSense D435 |
| **輸出** | 穩定運行的遙操作系統、攝影機串流驗證、latency 記錄 |

## 4. 步驟

### Step 1 — 安裝 RealSense

```bash
uv pip install pyrealsense2
lerobot-find-cameras realsense
```

確認 D435 被正確偵測到，記錄 camera index。

### Step 2 — 測試遙操作（無攝影機）

先不接攝影機，確認 Leader-Follower 動作鏡像正常：

```bash
lerobot-teleoperate \
  --robot.type=so101_follower --robot.port=/dev/ttyXXX \
  --teleop.type=so101_leader --teleop.port=/dev/ttyYYY
```

驗證：手動操作 Leader 臂，Follower 臂應即時鏡像，動作連續且可控。

### Step 3 — 加入 RealSense

設定攝影機參數：

- 解析度：`640x480` 或 `1280x720`
- FPS：`30`
- `use_depth=True`

```bash
lerobot-teleoperate \
  --robot.type=so101_follower --robot.port=/dev/ttyXXX \
  --teleop.type=so101_leader --teleop.port=/dev/ttyYYY \
  --robot.cameras.front.type=realsense \
  --robot.cameras.front.fps=30 \
  --robot.cameras.front.width=640 \
  --robot.cameras.front.height=480 \
  --robot.cameras.front.use_depth=true
```

### Step 4 — 記錄 Latency

測試並記錄以下指標：

- Leader → Follower 延遲（目標 < 50ms，但非硬性 gate）
- 攝影機幀率穩定性（是否維持 30 FPS）
- 是否有掉幀現象

## 5. 驗證方式

- Leader 動作 → Follower 即時鏡像（目視連續可控）
- `lerobot-find-cameras` 偵測到 D435
- 攝影機串流穩定，無頻繁掉幀
- 深度影像正常顯示

## 6. 關鍵連結

- [LeRobot SO-101 官方教學](https://huggingface.co/docs/lerobot/so101)
- [Intel RealSense D435 文件](https://www.intelrealsense.com/depth-camera-d435/)
- [pyrealsense2 文件](https://pypi.org/project/pyrealsense2/)

## 7. 成功標準

- [ ] 遙操作穩定運行
- [ ] 操作主觀連續可控
- [ ] 實測 latency 已記錄（目標 < 50ms，但非硬性 gate）
- [ ] RealSense 彩色 + 深度正常
- [ ] 攝影機可持續運行 5 分鐘以上無當機

## 8. 產出物

| 產出物 | 說明 |
|--------|------|
| Latency 測試記錄 | Leader → Follower 延遲數據 |
| 攝影機設定參數 | 解析度、FPS、depth 設定 |
| 遙操作測試影片 | （可選）留存供後續參考 |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| RealSense macOS 驅動不穩定 | macOS 下 RealSense 可能需要 `sudo` 或有相容性問題 | 可能需改用 Linux 環境 |
| USB 頻寬不足 | 攝影機 + 雙臂同時佔用 USB 頻寬 | 建議分開 USB controller（不同 USB port 群組） |
| D435 近距離深度精度下降 | 距離 < 20cm 時深度資料不可靠 | 調整攝影機擺放位置，保持 > 20cm 工作距離 |
