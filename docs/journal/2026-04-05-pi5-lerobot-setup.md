# 2026-04-05 — Pi 5 上線 + LeRobot 環境完整安裝

> Phase: 01 | Tags: #hardware, #setup, #debug

## 本週進度

- **主執行機切換**：Jetson Orin Nano 移交 elder_and_dog 機器狗專案，so101-lab 改用 **Raspberry Pi 5 8GB**（NCC 認證台灣版，NT$4,620）
- **Pi 5 到貨**：含主板 + Active Cooler + 27W USB-C 電源 + 128GB microSD
- **電源焊接完成**：先前 blocker 解除（實際比 2026-03-27 記錄時更早完成）
- **Pi 5 從零完整上線**：
  - Raspberry Pi Imager 燒錄 Raspberry Pi OS 64-bit (Debian 13 Trixie)
  - Imager 預設定：hostname、使用者、Wi-Fi、SSH（cloud-init / `user-data`）
  - 黏貼官方 Active Cooler 後才插電第一次開機
  - HDMI 直連確認 IP 與 SSH 狀態（mDNS 失敗的備案）
  - WSL → Pi 5 SSH key-based 免密 + `~/.ssh/config` alias `pi5`
- **系統健康**：kernel 6.12.75（升級後），CPU 46.1°C，`throttled=0x0`，主動散熱器正常
- **LeRobot 0.4.4 安裝完成**：
  - Seeed fork (`~/lerobot`)
  - `uv` 管理的 Python 3.10.20 venv（不碰系統 Python 3.13）
  - PyTorch 2.7.1+cpu、torchvision、Feetech SDK
  - 16 個 `lerobot-*` CLI 全部就位

## 關鍵發現

1. **Pi 5 vs Jetson Orin Nano Super 效能差距主要在 Phase 5 推論**
   - Pi 5 無 CUDA，67 TOPS 的 Jetson 優勢主要體現在邊緣 ACT/Diffusion 推論
   - Phase 1-3（校正、遙操作、資料收集）RPi5 完全夠用
   - Phase 4 訓練本來就在 RTX 8000 工作站，不受影響
   - Phase 5 部署方案要重新設計：可能走 ONNX CPU 推論，或 RTX 工作站 PolicyServer + Pi 5 RobotClient
2. **`uv` 比 conda 在 Pi 5 上更適合**
   - Debian 13 Trixie 的系統 Python 是 3.13，LeRobot 官方支援 3.10，`uv venv --python 3.10` 完美解決
   - 不用像 Jetson 那樣處理 JetPack PyTorch/opencv/numpy 的覆蓋問題
   - 安裝流程從「conda + 手動修復」簡化成「一行 uv pip install」
3. **Pi OS 的 `dialout` 群組預設已加入 `roy422`**
   - 不用每次執行 `sudo chmod 666 /dev/ttyACM*`（Jetson 教學的慣例）
   - 所有 `gpio / i2c / spi / render / input` 群組也預設加入，未來擴充 GPIO 硬體方便
4. **Imager 的 hostname 欄位行為**
   - 如果 GUI 裡沒主動改 hostname，會被 username 欄位覆蓋
   - 本專案的 Pi 5 實際 hostname 是 `roy422` 而非計畫中的 `so101-pi`
   - 由 SSH alias 補償，不影響功能

## 遇到的問題

### 1. mDNS 解析完全失敗
- WSL2 內 ping `so101-pi.local` → 找不到
- Windows 原生也找不到（沒裝 Bonjour，Windows 原生 mDNS 在某些環境不穩）
- **解法**：用 PowerShell `.NET Ping` 對 `192.168.0.0/24` 和 `192.168.1.0/24` 做 ping sweep 刷新 ARP 表
- **額外發現**：家裡有兩個路由器（`192.168.0.x` 和 `192.168.1.x`），Pi 5 實際在 `192.168.0.22`
- **最後備案**：直接 micro-HDMI 接螢幕 + 鍵盤看 `hostname -I`，5 分鐘解決

### 2. Pi 5 綠燈恆亮（誤以為卡住）
- 插電後綠燈一直亮，以為 boot 失敗
- 實際是第一次開機 cloud-init 還在跑（套用 Wi-Fi/SSH/帳號 + 擴充 rootfs + 自動重開一次），全程約 3-5 分鐘
- **結論**：Pi 5 第一次開機要耐心等

### 3. `rpi-chromium-mods` 把 `apt full-upgrade` 卡住
- 升級時彈出 `master_preferences` conffile 互動提示，non-interactive 模式下 dpkg exit 1
- `apt install --reinstall` 會失敗因為 cache 沒檔案了
- **解法**：`sudo apt purge -y rpi-chromium-mods chromium chromium-common chromium-l10n`
- Pi 5 做機器人開發不需要瀏覽器，purge 最乾淨

### 4. non-interactive SSH 的 PATH 問題
- `ssh pi5 'uv venv ...'` → `bash: uv: command not found`
- 原因：non-interactive shell 不載入 `~/.profile`，`~/.local/bin` 沒在 PATH
- **解法**：腳本化時用絕對路徑 `~/.local/bin/uv`
- **替代**：`ssh pi5 'bash -lc "uv ..."'` 啟動 login shell

### 5. chromium-mods 連鎖阻礙後續 apt
- Purge 之前，每次 `apt install` 都會在 trigger 階段抱怨 chromium-mods 的 dpkg error
- 其他套件其實有裝成功，但 exit code 非 0 讓自動化腳本誤判
- **結論**：遇到 apt 錯誤要看實際 trigger，不是每個 error 都代表套件沒裝

## 下週計畫

1. **接驅動板驗證 USB**
   - `lerobot-find-port` 能看到 `/dev/ttyACM0`
   - `lsusb` 確認 vendor/product ID
2. **綁 12 顆馬達 ID**（Leader L1-L6 + Follower F1-F6）
   - 一次一顆按 Seeed Wiki 流程
   - 注意馬達型號對照表，Leader 有 3 種減速比
3. **兩臂組裝最後確認 + 校正**
   - `lerobot-calibrate` 雙臂通過
   - 手動測試 6 關節獨立動作
4. **Phase 2 遙操作預備**
   - `lerobot-teleoperate` 不含相機 dry-run

## 相關資源

- [Phase 01 硬體組裝](../phases/01-hardware-setup.md)（本次大幅改寫 Step 1 環境安裝）
- [Seeed Studio SO-101 Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/)
- [Raspberry Pi 5 官方文件](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html)
- [uv documentation](https://docs.astral.sh/uv/)
- [2026-03-27 Phase 1 啟動日誌](2026-03-27-phase01-kickoff.md)（當時主執行機還計畫是 Jetson）
