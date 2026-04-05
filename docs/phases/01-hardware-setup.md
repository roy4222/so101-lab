# Phase 01 — 硬體組裝

## 1. 目標

完成 SO-101 Leader 臂和 Follower 臂的組裝、馬達設定與校正，確保所有關節可正常控制。

## 2. 前置條件

- Seeed Studio SO-ARM101 套件（含 12 顆舵機、2 塊驅動板、2 條 USB-C、2 顆電源適配器、桌面夾具）
- 3D 列印件（Follower + Leader）
- Raspberry Pi 5 8GB + Active Cooler + 27W USB-C 電源
- Raspberry Pi OS 64-bit (Debian 13 Trixie)
- Python 3.10（透過 `uv` 自動管理，不碰系統 Python 3.13）
- 參考 [硬體清單](../references/hardware.md)、[Seeed Studio Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | 未組裝的零件 |
| **輸出** | 已校正的 Leader + Follower 臂、校正檔、已驗證的 USB port 設定 |

## 4. 步驟

> **本章節對齊 [Seeed Studio Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/) 教學流程。**
> Seeed 的穩定版 fork 經過驗證，建議優先使用。官方 LeRobot 倉庫更新頻繁，可能有未預期的 breaking change。

### Step 1 — 環境安裝（在 Pi 5 上執行）

> **本專案實作記錄**：主執行機是 Raspberry Pi 5 8GB（取代 Jetson）。因為 Pi 5 無 CUDA GPU，LeRobot 會用 **PyTorch CPU 版**。Seeed 原教學是基於 Jetson JetPack + conda，Pi 5 上改用更輕量的 `uv` + venv，Debian 13 Trixie 的系統 Python 3.13 不碰，由 `uv` 自動下載 Python 3.10 到 venv。

#### 1a. 系統升級（新燒錄的 SD 卡必做）

```bash
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt full-upgrade -y
sudo reboot
```

> **踩坑**：Debian 13 新鏡像可能包含 `rpi-chromium-mods`，升級時會卡 conffile 互動提示。本專案開發用不到 Chromium，可直接 `sudo apt purge -y rpi-chromium-mods chromium chromium-l10n chromium-common && sudo apt autoremove -y`。

#### 1b. 基礎開發工具

```bash
sudo DEBIAN_FRONTEND=noninteractive apt install -y \
    git python3-venv python3-pip python3-dev \
    build-essential pkg-config \
    ffmpeg libusb-1.0-0-dev libudev-dev \
    curl htop tmux
```

確認 `dialout` 群組（Pi OS 預設已加入，USB 串口免 sudo）：

```bash
groups | grep -q dialout && echo "OK: dialout member" || sudo usermod -aG dialout $USER  # 若未加入需重新登入
```

#### 1c. 安裝 uv（Astral 家的 Python 套件管理器）

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# uv 會裝到 ~/.local/bin/uv，Debian 預設已在 PATH
# 注意：non-interactive SSH 不會載入 ~/.profile，腳本化時請用絕對路徑 ~/.local/bin/uv
```

#### 1d. Clone Seeed LeRobot 並建立 Python 3.10 venv

```bash
git clone https://github.com/Seeed-Projects/lerobot.git ~/lerobot
cd ~/lerobot

# uv 會自動下載 CPython 3.10 到 ~/.local/share/uv/python/（不碰系統 Python 3.13）
uv venv --python 3.10 .venv

# 安裝 LeRobot + Feetech 依賴（約 5-15 分鐘，下載 1-2GB）
uv pip install --python .venv/bin/python -e ".[feetech]"
```

#### 1e. 驗證

```bash
cd ~/lerobot
source .venv/bin/activate
python -c "import lerobot; print('lerobot', lerobot.__version__)"
python -c "import torch; print('torch', torch.__version__)"  # 預期: 2.7.1+cpu
which lerobot-find-port  # 應指向 ~/lerobot/.venv/bin/lerobot-find-port
```

#### 1f. （選用）加 venv 啟動捷徑

```bash
cat >> ~/.bashrc << 'EOF'

# lerobot venv
export PATH="$HOME/.local/bin:$PATH"
alias lerobot-activate="source ~/lerobot/.venv/bin/activate"
EOF
```

之後 `ssh pi5` 進來打 `lerobot-activate` 就進 venv。

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

> ⚠️ **重要**：每次提示出現後**一定要真的物理換馬達再按 Enter**。腳本無法偵測你是否真的換了馬達，只會按順序寫入 ID。如果每次都按 Enter 但馬達沒換，6 次 ID 會全部寫在同一顆馬達上（最後那顆變 ID=1），其他 5 顆還是出廠 ID，看起來成功其實失敗。
>
> **驗收方式**：下方 Step 2.5 的 bus 掃描是唯一可靠的驗證。

### Step 2.5 — 驗證馬達 ID 綁定（必做）

完成 Step 2 後，把 6 顆馬達用 3-pin 線**串聯回去**，跑以下掃描腳本確認 ID 1-6 都在 bus 上：

```bash
cd ~/lerobot && source .venv/bin/activate

python -c "
import scservo_sdk as scs
port = scs.PortHandler('/dev/ttyACM0')
port.openPort(); port.setBaudRate(1_000_000)
packet = scs.PacketHandler(0)
found = []
for mid in range(1, 11):
    model, comm, _ = packet.ping(port, mid)
    if comm == scs.COMM_SUCCESS:
        found.append(mid)
        print(f'ID {mid}: model={model}')
print(f'Total: {found}')
port.closePort()
"
```

**預期輸出**：`Total: [1, 2, 3, 4, 5, 6]`

- 如果只看到 4 顆（通常是 1-4，缺 5-6）→ 串聯線在某兩顆之間斷了，通常是 wrist_flex ↔ wrist_roll 那條
- 如果只看到 1 顆 ID=1 → Step 2 你沒有物理換馬達，要重跑
- 如果看到 6 顆但 IDs 不連續 → 腳本中途有斷過，重跑

Leader 臂重複一次（USB port 可能變 `/dev/ttyACM0` 或 `/dev/ttyACM1`，視插拔順序）。

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
WSL2 (/home/roy422/so101-lab)       → 寫 code、Git、AI 協作
        ↓ rsync / ssh pi5
Raspberry Pi 5 (192.168.0.22)       → LeRobot 執行、接手臂、接相機
  user: roy422
  hostname: roy422
  ~/lerobot (Seeed fork + .venv)
```

- 主執行機：Raspberry Pi 5 8GB（Debian 13 Trixie）
- USB port 在 Pi 5 上為 `/dev/ttyACM0`、`/dev/ttyACM1`
- **不需** `sudo chmod 666`：`roy422` 已在 `dialout` 群組（Pi OS 預設）
- WSL → Pi 5 連線：`ssh pi5`（已設定 SSH key 免密 + `~/.ssh/config` alias）

## 7. 成功標準

- [x] 12 顆馬達 ID 設定完成（2026-04-05）
- [x] 雙臂 bus 掃描驗證通過（Follower + Leader 各 6 顆回應，2026-04-05）
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

#### 硬體組裝相關

- **先裝馬達再接線是可行的**，但空間會比較小。建議用鑷子或尖嘴鉗輔助插 3-pin 接頭。官方建議「先插線再裝馬達」，但不是硬性要求。
- **Seeed Studio 套件版差異**：驅動板和電源配置可能和 HuggingFace 官方教學不同，接線請對照 [Seeed Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/) 而非官方 LeRobot docs。
- **電源不能只靠 USB-C**：USB-C 只傳資料，馬達需要獨立 DC 電源（標準版 5V，Pro 版 Follower 12V / Leader 5V）。

#### 馬達 ID 綁定踩坑（2026-04-05 實作記錄）

- **按 Enter 不換馬達的烏龍**：`lerobot-setup-motors` 第一次跑只花 29 秒就印完 6 個「success」訊息——看起來成功，實際是同一顆馬達被覆蓋寫入 6 次 ID，最後變 ID=1，其他 5 顆還是出廠狀態。**教訓**：腳本無法偵測馬達是否物理更換，每個提示一定要真的拔上一顆、插下一顆、才按 Enter。第二次跑正確流程花了 3m45s（Follower）和 12m7s（Leader），才是正常時間。
- **掃描驗證的重要性**：用 `scservo_sdk.PacketHandler.ping()` 對 ID 1-10 掃描是最快抓出綁定問題的方法。沒掃就進校正會在奇怪的地方失敗。本文件 Step 2.5 有完整腳本。
- **3-pin 串聯線鬆脫 = bus 上「後面所有」馬達消失**：Leader 第一次掃只找到 1-4 號，缺 5、6。**不是**馬達壞掉，而是 wrist_flex (4) → wrist_roll (5) 的串聯線訊號針沒插到底。訊號線一旦斷，daisy chain 上**所有在斷點後面的馬達**都會從 bus 消失。但**電源針獨立**，所以馬達 LED 還會亮——不能用 LED 來判斷 bus 是否通。重插後 6 顆全數回歸。

#### LeRobot 執行時錯誤

- **`Motor 'gripper' was not found`**：檢查通訊線是否插好、電源是否正確供電。
- **`Could not connect on port`**：確認 `roy422` 在 `dialout` 群組（`groups | grep dialout`）；或檢查是否別的程式佔用 `/dev/ttyACM0`。
- **`Magnitude exceeds 2047`**：斷電重新上電再試校正。如果反覆出現，需要重新寫入馬達 ID。
- **`Failed to sync read 'Present_Position'`**：檢查電源是否接通、哪顆馬達的燈不亮（前面那顆的線鬆了）。

#### Pi 5 環境踩坑（2026-04-05 實作記錄）

- **mDNS 解析失敗**：WSL2 內和 Windows 原生都可能解析不到 `<hostname>.local`。用路由器後台找 IP，或 `arp -a` + ping sweep。Pi 5 MAC 是 globally unique，但 Wi-Fi MAC 隱私功能可能產生 locally administered MAC，不能只看 OUI 判斷。
- **Hostname 意外**：Imager GUI 的「Set hostname」如果沒改，預設會跟「username」一樣；本專案的 Pi 5 hostname 是 `roy422` 而非 `so101-pi`。不影響功能，但 SSH alias 更重要（`ssh pi5`）。
- **`rpi-chromium-mods` 卡 apt**：Debian 13 鏡像的 Chromium 修補套件在升級時會彈 conffile 互動提示，`apt full-upgrade` 會卡住。直接 purge 解決。
- **non-interactive SSH 的 PATH 問題**：`ssh pi5 'uv ...'` 會失敗因為 `~/.local/bin` 沒載入。腳本化時用絕對路徑 `~/.local/bin/uv`，或用 `ssh pi5 'bash -lc "uv ..."'`。
- **Python 3.13 vs LeRobot**：Debian 13 Trixie 系統 Python 是 3.13，LeRobot 雖然 `requires-python = ">=3.10"`，但某些深度學習依賴（PyTorch、OpenCV）對 3.13 的 ARM64 wheel 支援仍在追趕。保險作法：用 `uv venv --python 3.10` 讓 uv 自己抓 3.10。

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| USB port 辨識 | Linux port 名在重新插拔後可能改變 | 每次操作前先執行 `lerobot-find-port` 確認 |
| 馬達 ID 衝突 | 多顆馬達同時接上會造成 ID 碰撞 | 嚴格遵守「一次只接一顆」原則 |
| 馬達方向錯誤 | 馬達裝反會導致校正失敗 | 組裝前對照官方文件確認方向標記 |
| 電源接頭焊接 | Seeed Studio 套件可能需要焊接電源接頭 | 焊接前確認電壓（標準版 5V / Pro 版 Leader 5V、Follower 12V），注意正負極 |
