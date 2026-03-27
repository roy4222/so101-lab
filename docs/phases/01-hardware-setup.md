# Phase 01 — 硬體組裝

## 1. 目標

完成 SO-101 Leader 臂和 Follower 臂的組裝、馬達設定與校正，確保所有關節可正常控制。

## 2. 前置條件

- SO-101 零件齊全（馬達、控制板、3D 列印結構件、螺絲）
- 電腦已安裝 Python 3.10+
- USB 連接線 x2
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 未組裝的零件 |
| **輸出** | 已校正的 Leader + Follower 臂、校正檔、已驗證的 USB port 設定 |

## 4. 步驟

### Step 1 — 環境安裝

```bash
uv init
uv venv
source .venv/bin/activate
uv pip install lerobot
uv pip install "lerobot[feetech]"
lerobot-info
```

### Step 2 — 組裝機械臂

按 LeRobot SO-101 官方文件進行組裝：

- **Follower 臂**：6x STS3215（齒輪比 1/345）
- **Leader 臂**：6x STS3215（混合齒輪比）

> 注意：組裝時務必確認馬達方向，裝反會導致校正失敗。

### Step 3 — 找 USB Port

```bash
lerobot-find-port
```

記錄輸出的 port 名稱（例如 `/dev/tty.usbserial-XXXX`），後續步驟會用到。

### Step 4 — 設定馬達 ID

**重要：一次只接一顆馬達，避免 ID 衝突。**

Follower 臂：

```bash
lerobot-setup-motors --robot.type=so101_follower --robot.port=/dev/ttyXXX
```

Leader 臂：

```bash
lerobot-setup-motors --teleop.type=so101_leader --teleop.port=/dev/ttyYYY
```

### Step 5 — 校正

分別對 Follower 和 Leader 執行校正：

```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyXXX
lerobot-calibrate --teleop.type=so101_leader --teleop.port=/dev/ttyYYY
```

## 5. 驗證方式

- `lerobot-calibrate` 雙臂皆成功完成，無報錯
- 手動測試各關節，動作流暢無卡頓
- 校正檔已生成於 `~/.cache/huggingface/lerobot/calibration/`

```bash
ls ~/.cache/huggingface/lerobot/calibration/my_follower/
ls ~/.cache/huggingface/lerobot/calibration/my_leader/
```

## 6. 關鍵連結

- [LeRobot SO-101 官方教學](https://huggingface.co/docs/lerobot/so101)
- [Feetech STS3215 Datasheet](https://www.feetechrc.com/STS3215)
- [LeRobot GitHub](https://github.com/huggingface/lerobot)

## 7. 成功標準

- [ ] 12 顆馬達 ID 設定完成
- [ ] `lerobot-calibrate` 雙臂通過
- [ ] 6 個關節可獨立控制
- [ ] 校正檔已備份
- [ ] 步驟可重現（另一人依此文件可完成組裝）

## 8. 產出物

| 產出物 | 路徑 / 說明 |
|--------|-------------|
| Follower 校正檔 | `~/.cache/huggingface/lerobot/calibration/my_follower/` |
| Leader 校正檔 | `~/.cache/huggingface/lerobot/calibration/my_leader/` |
| USB port 記錄 | 記錄於本文件或專案 wiki |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| USB port 辨識 | macOS port 名可能在重新插拔後改變 | 每次操作前先執行 `lerobot-find-port` 確認 |
| 馬達 ID 衝突 | 多顆馬達同時接上會造成 ID 碰撞 | 嚴格遵守「一次只接一顆」原則 |
| 馬達方向錯誤 | 馬達裝反會導致校正失敗 | 組裝前對照官方文件確認方向標記 |
