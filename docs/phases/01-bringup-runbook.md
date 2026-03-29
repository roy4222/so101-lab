# Phase 01 — Bring-up Runbook

> 從組裝完成到第一次遙操作的完整執行手冊。
> 每個 milestone 包含：目標、指令、通過條件、失敗 fallback。
> 實際執行結果回填在各 milestone 的「執行紀錄」區段。

**建立日期**：2026-03-29
**Jetson 環境**：JetPack 6.2.1 / L4T R36.4.3 / CUDA 12.6 / zsh / 系統 torch 2.10.0+cpu / numpy 1.26.4
**隔離策略**：uv venv + PYTHONNOUSERSITE=1，不碰系統 Python，不改 shell rc

---

## M1 — 安全上電檢查

**目標**：確認供電正確、馬達有回應、沒有硬體異常

**步驟（人工）**：
1. 確認電壓正確（標準版全 5V）
2. 接上電源，觀察每顆馬達 LED 是否亮起
3. 檢查無異音、異味、過熱
4. 兩條 USB-C 接上 Jetson

**通過條件**：
- 12 顆馬達 LED 全亮
- 無異常現象
- `ls /dev/ttyACM*` 看到兩個裝置

**失敗 fallback**：
- LED 不亮 → 檢查電源正負極、DC barrel jack 焊接
- 只偵測到一個 ttyACM → 換 USB port 或換線
- 有異味 → 立即斷電，檢查電壓是否用錯

**執行紀錄**：
> - 2026-03-29：M1 通過
> - USB 枚舉：ttyACM0 (Leader), ttyACM1 (Follower)
> - 12 顆馬達 LED 全亮，無異音、無異味、無過熱
> - 已加 udev rule：`/etc/udev/rules.d/99-ttyacm.rules`，以後不用每次 chmod

---

## M2 — Jetson 環境就緒

分兩個 gate：先確認工具鏈，再安裝 LeRobot。

### Gate 2a — uv + venv 就緒

**目標**：uv 裝好、venv 建好、隔離確認

**指令**：
```bash
# 安裝 uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.cargo/env  # 或重新 source shell

# 驗證
uv --version

# 建 venv
uv venv ~/lerobot-env --python 3.10

# 驗證 python（不依賴 env 內有 pip）
export PYTHONNOUSERSITE=1
~/lerobot-env/bin/python --version
~/lerobot-env/bin/python -c "import site; print(site.ENABLE_USER_SITE)"  # 必須是 False
```

**通過條件**：
- `uv --version` 有輸出
- `~/lerobot-env/bin/python --version` → Python 3.10.x
- `site.ENABLE_USER_SITE` → `False`

**失敗 fallback**：
- uv 安裝失敗 → 重試官方 installer，或退回 `python3 -m venv ~/lerobot-env`
- ENABLE_USER_SITE 不是 False → 確認 PYTHONNOUSERSITE=1 有 export

**執行紀錄**：
> - 2026-03-29：Gate 2a 通過
> - uv 0.11.2 安裝於 `~/.local/bin/uv`
> - venv 建於 `~/lerobot-env/`，Python 3.10.12
> - `ENABLE_USER_SITE = False` 確認

### Gate 2b — LeRobot 安裝

**目標**：LeRobot 裝好，CLI 工具可用

**指令**：
```bash
export PYTHONNOUSERSITE=1

# 克隆（存在檢查）
if [ -d ~/lerobot ]; then
  echo "WARNING: ~/lerobot already exists"
  ls ~/lerobot/
  # 決定：沿用 or mv ~/lerobot ~/lerobot.bak.$(date +%Y%m%d%H%M%S)
else
  git clone https://github.com/Seeed-Projects/lerobot.git ~/lerobot
fi

# 安裝（用 uv 直接指定 python，不依賴 activate + which pip）
cd ~/lerobot && uv pip install --python ~/lerobot-env/bin/python -e ".[feetech]"

# 驗證 CLI
lerobot-find-port --help
lerobot-setup-motors --help
lerobot-calibrate --help
```

**通過條件**：
- `lerobot-find-port --help` 有輸出
- `lerobot-setup-motors --help` 有輸出
- `lerobot-calibrate --help` 有輸出

**系統污染檢查**：
```bash
/usr/bin/python3 -c "import torch, numpy; print(torch.__version__, numpy.__version__)"
# 必須仍是 2.10.0 1.26.4
```

**失敗 fallback**：
- `uv pip install` 失敗 → 看錯誤訊息，可能缺系統 lib（`apt install` 補）
- ffmpeg 版本不夠 → 先不管，Phase 1 不錄影，真卡住再升級
- 系統 torch/numpy 版本變了 → 有東西漏到系統 pip，需要排查

**執行紀錄**：
> - 2026-03-29：Gate 2b 通過
> - LeRobot v0.4.4（Seeed fork）安裝於 venv
> - lerobot-find-port / setup-motors / calibrate CLI 皆可用
> - 系統污染檢查通過：torch 2.10.0+cpu / numpy 1.26.4 沒變

---

## M3 — 馬達 ID 確認/重設

**目標**：確認 12 顆馬達 ID 正確（Follower 1-6, Leader 1-6 各自在獨立匯流排）

**指令**：
```bash
source ~/lerobot-env/bin/activate && export PYTHONNOUSERSITE=1
sudo chmod 666 /dev/ttyACM*
lerobot-find-port
# 拔掉其中一條 USB 按 Enter，確認哪個 port 是哪隻臂
```

**先嘗試整串接法**：
1. 先用 `lerobot-find-port` 確認兩條 port 分別對應哪隻臂
2. 嘗試進入 `lerobot-setup-motors`，觀察腳本回應
3. 若流程顯示已有可用設定且能正常辨識每顆馬達 → 繼續到 M4
4. 若遇到 ID 衝突、找不到馬達、或通訊錯誤 → 切換到單顆接法

```bash
# Follower
lerobot-setup-motors \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM<N>

# Leader
lerobot-setup-motors \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM<N>
```

**單顆接法（fallback）**：
如果上面碰到衝突或錯誤：
1. 斷電
2. 把驅動板上 6 顆馬達的 3-pin 線全拔
3. 一次只接一顆，從 gripper (ID 6) 到 shoulder_pan (ID 1)
4. 每接一顆跑一次 setup-motors
5. 全部設好再串接回去
6. 兩隻臂各做一次，共 12 顆

**通過條件**：
- setup-motors 兩隻臂都完成無報錯
- 每隻臂 6 顆馬達 ID 1-6 各自正確

**失敗 fallback**：
- `Motor not found` → 檢查 3-pin 線、電源
- `Could not connect on port` → `sudo chmod 666 /dev/ttyACM*`
- 馬達完全沒回應 → 斷電重新上電

**執行紀錄**：
> - 2026-03-29：M3 進行中，blocked on 線材
> - Port 對應：ttyACM0 = Leader, ttyACM1 = Follower
> - 掃描結果：所有馬達 ID 皆為出廠預設 1（ID 衝突），需一顆顆設定
> - 發現問題：3-pin 線在組裝後極難拔出（Molex 5264-3P 相容，2.5mm 間距）
> - 馬達輸入/輸出端可能有接反的情況（Follower bus 掃描為 0 顆）
> - **Blocker**：需購買替換線材 — FeeTech ST-3215 3P 訊號線 26cm (NT$30/條)
> - 預計買 12-15 條，到貨後重新接線 + 設 ID

---

## M4 — 雙臂校正

**目標**：Leader 和 Follower 各完成校正，產生 calibration 檔

**指令**：
```bash
source ~/lerobot-env/bin/activate && export PYTHONNOUSERSITE=1
sudo chmod 666 /dev/ttyACM*

# Follower
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM<N> \
    --robot.id=my_awesome_follower_arm

# Leader
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM<N> \
    --teleop.id=my_awesome_leader_arm
```

**校正流程（人工操作）**：
1. 將所有關節移到活動範圍中間位置
2. 按 Enter
3. 將每個關節在完整運動範圍內移動

**通過條件**：
- 兩隻臂校正完成無報錯
- 校正檔已生成：
```bash
ls ~/.cache/huggingface/lerobot/calibration/my_awesome_follower_arm/
ls ~/.cache/huggingface/lerobot/calibration/my_awesome_leader_arm/
```

**失敗 fallback**：
- `Magnitude exceeds 2047` → 斷電重新上電再試
- `Failed to sync read 'Present_Position'` → 檢查電源、哪顆馬達燈不亮
- 校正結果明顯不對 → 刪除校正檔重做：`rm -rf ~/.cache/huggingface/lerobot/calibration/my_awesome_*`

**執行紀錄**：
> （待回填）

---

## M5 — 第一次遙操作

### Safety Gate（人工，在跑指令前必做）
- [ ] Follower 臂周圍淨空，沒有會被夾到的東西
- [ ] 手不要放在連桿和關節夾點附近
- [ ] 準備好隨時斷電（電源線在手邊）
- [ ] 知道 Ctrl+C 可以停程式

**目標**：Leader 動、Follower 即時跟隨

**指令**：
```bash
source ~/lerobot-env/bin/activate && export PYTHONNOUSERSITE=1
sudo chmod 666 /dev/ttyACM*

lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM<N> \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM<N> \
    --teleop.id=my_awesome_leader_arm
```

**通過條件**：
- 移動 Leader 臂，Follower 臂即時跟隨
- 6 個關節都有反應
- 無明顯延遲或抖動

**失敗 fallback**：
- 某個關節不動 → 檢查該關節馬達 ID 和 3-pin 線
- 動作方向相反 → 可能需要重新校正
- 整體不動 → 回到 M3 確認 ID

**執行紀錄**：
> （待回填）

---

## 分工

| 里程碑 | Claude（SSH） | 操作者（現場） |
|--------|-----------|-----------|
| M1 上電 | 跑 `ls /dev/ttyACM*` | 接電、觀察馬達 LED |
| M2 環境 | 全程安裝 | 不用動 |
| M3 ID | 跑指令 | 插拔馬達線（如需要） |
| M4 校正 | 跑指令 | 移動關節 |
| M5 遙操 | 跑指令 | 操作 Leader、確認安全 |

## 隔離鐵則（貫穿全程）

1. 不改 `~/.zshrc`、`~/.bashrc`
2. 不用 `sudo pip`、不在未 activate 時跑 pip
3. 安裝用 `uv pip install --python ~/lerobot-env/bin/python`，執行用 `source ~/lerobot-env/bin/activate && export PYTHONNOUSERSITE=1`
4. 每步驗證 `which python` 在 venv 內
5. 完成後 `/usr/bin/python3 -c "import torch, numpy; print(torch.__version__, numpy.__version__)"` 確認沒變
