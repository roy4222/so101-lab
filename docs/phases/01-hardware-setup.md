# Phase 01 — 硬體組裝

## 1. 目標

完成 SO-101 Leader 臂和 Follower 臂的組裝、馬達設定與校正，確保所有關節可正常控制。

## 2. 前置條件

- Seeed Studio SO-ARM101 套件（含 12 顆舵機、2 塊驅動板、2 條 USB-C、2 顆電源適配器、桌面夾具）
- 3D 列印件（Follower + Leader）
- NVIDIA Jetson Orin Nano SUPER 8GB（JetPack 6.0 或 6.1）
- Python 3.10
- 參考 [硬體清單](../references/hardware.md)、[Seeed Studio Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 未組裝的零件 |
| **輸出** | 已校正的 Leader + Follower 臂、校正檔、已驗證的 USB port 設定 |

## 4. 步驟

> **本章節對齊 [Seeed Studio Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/) 教學流程。**
> Seeed 的穩定版 fork 經過驗證，建議優先使用。官方 LeRobot 倉庫更新頻繁，可能有未預期的 breaking change。

### Step 1 — 環境安裝（在 Jetson 上執行）

```bash
# 安裝 Miniconda (Jetson ARM64)
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
chmod +x Miniconda3-latest-Linux-aarch64.sh
./Miniconda3-latest-Linux-aarch64.sh
source ~/.bashrc

# 建立 conda 環境
conda create -y -n lerobot python=3.10 && conda activate lerobot

# 安裝 ffmpeg
conda install ffmpeg=7.1.1 -c conda-forge

# 克隆 Seeed 穩定版 LeRobot
git clone https://github.com/Seeed-Projects/lerobot.git ~/lerobot

# 安裝 LeRobot + Feetech 依賴
cd ~/lerobot && pip install -e ".[feetech]"
```

**Jetson 專用步驟**（確保 GPU 版 PyTorch）：

```bash
# 先按 https://wiki.seeedstudio.com/reComputer_Jetson_Series_Pytorch/ 安裝 PyTorch-gpu
# 然後修復被 pip 覆蓋的套件：
conda install -y -c conda-forge "opencv>=4.10.0.84"
conda remove opencv
pip3 install opencv-python==4.10.0.84
conda uninstall numpy
pip3 install numpy==1.26.0
```

驗證 GPU：

```python
import torch
print(torch.cuda.is_available())  # 必須為 True
```

### Step 2 — 舵機 ID 設定（先於組裝）

> **Seeed 流程：先設定 ID，再組裝。** 這樣可以一次只接一顆馬達到驅動板，避免 ID 衝突。

#### SO-101 馬達對照表

建議在每顆馬達上標記 F1-F6 / L1-L6：

| 舵機型號 | 減速比 | 對應關節 |
|---------|--------|---------|
| ST-3215-C044 (7.4V) | 1:191 | L1 (Shoulder Pan) |
| ST-3215-C001 (7.4V) | 1:345 | L2 (Shoulder Lift) |
| ST-3215-C044 (7.4V) | 1:191 | L3 (Elbow Flex) |
| ST-3215-C046 (7.4V) | 1:147 | L4, L5, L6 (Wrist + Gripper) |
| ST-3215-C001 (7.4V) | 1:345 | F1–F6 (全部) |

> **電壓警告**：標準版全部用 5V。Pro 版 Leader 用 5V、Follower 用 12V。用錯電壓會燒馬達！

找 USB port：

```bash
lerobot-find-port
# 拔掉其中一條 USB 按 Enter，確認哪個 port 是哪隻臂
```

給權限：

```bash
sudo chmod 666 /dev/ttyACM*
```

設定 Follower 馬達 ID（一次只接一顆）：

```bash
lerobot-setup-motors \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0
```

設定 Leader 馬達 ID（一次只接一顆）：

```bash
lerobot-setup-motors \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1
```

依腳本提示，從 gripper (ID 6) 到 shoulder_pan (ID 1) 逐一接上單顆馬達並按 Enter。完成後再將 6 顆馬達串接回驅動板。

### Step 3 — 組裝機械臂

按 [Seeed Wiki 組裝教程](https://wiki.seeedstudio.com/cn/lerobot_so100m/#%E7%BB%84%E8%A3%85%E6%95%99%E7%A8%8B) 的圖片步驟進行：

- SO-101 雙臂組裝流程和 SO-100 相同
- SO-101 多了線纜夾，Leader 臂齒輪比不同
- **組裝前再次確認馬達型號和減速比對應正確**

> 注意：組裝時務必確認馬達方向，裝反會導致校正失敗。

### Step 4 — 校正

接上電源和 USB，將 6 顆馬達透過 3-pin 串接，底座馬達接驅動板。

```bash
sudo chmod 666 /dev/ttyACM*

# 校正 Follower 臂
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm

# 校正 Leader 臂
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm
```

校正流程：
1. 將所有關節移到活動範圍中間位置
2. 按 Enter
3. 將每個關節在完整運動範圍內移動

> **重要**：`--robot.id` 和 `--teleop.id` 決定校正檔路徑，後續遙操作、錄製、推論都必須用相同 ID。
>
> 如需重新校正，先刪除 `~/.cache/huggingface/lerobot/calibration/robots` 或 `teleoperators` 下的檔案。

## 5. 驗證方式

- `lerobot-calibrate` 雙臂皆成功完成，無報錯
- 手動測試各關節，動作流暢無卡頓
- 校正檔已生成於 `~/.cache/huggingface/lerobot/calibration/`

```bash
ls ~/.cache/huggingface/lerobot/calibration/my_awesome_follower_arm/
ls ~/.cache/huggingface/lerobot/calibration/my_awesome_leader_arm/
```

快速遙操作驗證（不含攝影機）：

```bash
sudo chmod 666 /dev/ttyACM*

lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm
```

移動 Leader 臂，Follower 臂應即時跟隨。

## 6. 關鍵連結

- [LeRobot SO-101 官方教學](https://huggingface.co/docs/lerobot/so101)
- [Seeed Studio SO-101 Wiki（本套件適用）](https://wiki.seeedstudio.com/cn/lerobot_so100m/)
- [Feetech STS3215 Datasheet](https://www.feetechrc.com/STS3215)
- [LeRobot GitHub](https://github.com/huggingface/lerobot)
- [Seeed Projects LeRobot Fork](https://github.com/Seeed-Projects/lerobot.git)

### 開發環境

```
WSL2 (/home/roy422/so101-lab)     → 寫 code、Git
        ↓ rsync
Jetson (/home/jetson/so101-lab)   → LeRobot 執行、接手臂、接相機
```

- 主執行機：NVIDIA Jetson Orin Nano SUPER 8GB
- USB port 在 Jetson 上為 `/dev/ttyACM0`、`/dev/ttyACM1`
- 需先給權限：`sudo chmod 666 /dev/ttyACM*`

## 7. 成功標準

- [ ] 12 顆馬達 ID 設定完成
- [ ] `lerobot-calibrate` 雙臂通過
- [ ] 6 個關節可獨立控制
- [ ] 校正檔已備份
- [ ] 步驟可重現（另一人依此文件可完成組裝）

## 8. 產出物

| 產出物 | 路徑 / 說明 |
|--------|-------------|
| Follower 校正檔 | `~/.cache/huggingface/lerobot/calibration/my_awesome_follower_arm/` |
| Leader 校正檔 | `~/.cache/huggingface/lerobot/calibration/my_awesome_leader_arm/` |
| USB port 記錄 | 記錄於本文件或專案 wiki |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- **先裝馬達再接線是可行的**，但空間會比較小。建議用鑷子或尖嘴鉗輔助插 3-pin 接頭。官方建議「先插線再裝馬達」，但不是硬性要求。
- **Seeed Studio 套件版差異**：驅動板和電源配置可能和 HuggingFace 官方教學不同，接線請對照 [Seeed Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/) 而非官方 LeRobot docs。
- **電源不能只靠 USB-C**：USB-C 只傳資料，馬達需要獨立 DC 電源（標準版 5V，Pro 版 Follower 12V / Leader 5V）。
- **`Motor 'gripper' was not found`**：檢查通訊線是否插好、電源是否正確供電。
- **`Could not connect on port`**：忘記給權限，`sudo chmod 666 /dev/ttyACM*`。
- **`Magnitude exceeds 2047`**：斷電重新上電再試校正。如果反覆出現，需要重新寫入馬達 ID。
- **`Failed to sync read 'Present_Position'`**：檢查電源是否接通、哪顆馬達的燈不亮（前面那顆的線鬆了）。
- **pip 安裝 lerobot 後 `torch.cuda.is_available()` 變 False**：pip 會覆蓋 GPU 版 PyTorch，需按 Jetson 教學重裝。

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| USB port 辨識 | Linux port 名在重新插拔後可能改變 | 每次操作前先執行 `lerobot-find-port` 確認 |
| 馬達 ID 衝突 | 多顆馬達同時接上會造成 ID 碰撞 | 嚴格遵守「一次只接一顆」原則 |
| 馬達方向錯誤 | 馬達裝反會導致校正失敗 | 組裝前對照官方文件確認方向標記 |
| 電源接頭焊接 | Seeed Studio 套件可能需要焊接電源接頭 | 焊接前確認電壓（標準版 5V / Pro 版 Leader 5V、Follower 12V），注意正負極 |
