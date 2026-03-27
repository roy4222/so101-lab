# 2026-03-27 — Phase 1 啟動：組裝進度與開發架構決策

> Phase: 01 | Tags: #hardware, #setup

## 本週進度

- Phase 1 正式啟動
- 3D 列印件完成
- 12 顆 Feetech STS3215 馬達已裝入列印件
- 馬達間 3-pin 接線已完成
- 電源接頭待焊接（最後一步硬體工作）

## 關鍵發現

- **Seeed Studio 套件版本**：購買的是 Seeed Studio SO-ARM101 套件，驅動板和電源配置與 HuggingFace 官方教學略有不同，接線和校正請參考 [Seeed Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/)
- **先裝馬達再接線**：官方建議先插線再裝馬達，但實際上馬達裝好後再接線也可行，只是空間較小需要用鑷子輔助
- **USB-C 不供電**：USB-C 只傳資料，馬達必須接獨立 DC 電源才會動

## 遇到的問題

- 電源接口需要焊接，尚未確認具體焊接位置（DC barrel jack 或直接焊 VIN/GND）
- Mac Mini 作為開發機被排除 — LeRobot 需要直接接 USB 馬達，Jetson 更適合

## 開發架構決策

確定採用和 elder_and_dog 專案相同的 WSL2 → Jetson 同步模式：

- **WSL2** (`/home/roy422/so101-lab`)：Coding、Git、AI Agent 協作
- **Jetson** (`/home/jetson/so101-lab`)：LeRobot 執行、校正、遙操作、錄資料
- **RTX 8000 工作站**：Phase 4 訓練
- 兩個專案（elder_and_dog / so101-lab）在 Jetson 上路徑獨立、venv 獨立，不會互相污染

## 下週計畫

1. 焊接電源接頭
2. 確認驅動板上電正常
3. 在 Jetson 上安裝 LeRobot 環境
4. 執行 `lerobot-find-port` 確認 USB port
5. 設定 12 顆馬達 ID

## 相關資源

- [Seeed Studio SO-101 Wiki](https://wiki.seeedstudio.com/cn/lerobot_so100m/)
- [HuggingFace SO-101 Assembly](https://huggingface.co/docs/lerobot/en/assemble_so101)
- [ArturLab Build Tutorial](https://arturhabuda.com/2025/07/01/build-tutorial-so-101-robot-arms/)
