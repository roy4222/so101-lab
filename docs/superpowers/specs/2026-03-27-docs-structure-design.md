# SO-101 專案文件結構設計規格

> 定稿日期：2026-03-27

## 1. 背景與目的

本專案從 HuggingFace LeRobot SO-101 單臂機器人出發，最終目標是演進至 XLeRobot 雙臂移動機器人平台。需要一套完整的 `docs/` 資料夾，涵蓋願景、roadmap、架構、分階段指南、參考資料和開發日誌。

### 受眾

- **主要受眾**：專案擁有者本人（盧柏宇），作為實作導向的開發追蹤
- **次要受眾**：開源社群新手，需要可讀的入門指引

### 語言

繁體中文為主，專有名詞保留英文（如 ACT、Diffusion Policy、LeRobot、RealSense）。

---

## 2. 目錄結構

```
docs/
├── README.md                          # 導覽首頁（新手入口）
├── goals.md                           # 願景、目標、成功標準
├── roadmap.md                         # 全階段 roadmap（短/中/長期 + 優先級）
├── architecture/
│   └── overview.md                    # 高層系統架構（漸進補充）
├── phases/
│   ├── 01-hardware-setup.md           # 組裝、接線、馬達設定、校正
│   ├── 02-teleoperation.md            # 遙操作 + RealSense 攝影機
│   ├── 03-data-collection.md          # 錄製資料集、資料管理
│   ├── 04-training.md                 # ACT / Diffusion / VLA 訓練
│   ├── 05-deployment.md               # 推論部署（本地 + Jetson）
│   ├── 06-dual-arm.md                 # 雙臂改造 + 協調控制
│   ├── 07-simulation.md               # Isaac Sim / MuJoCo / Sim-to-Real
│   └── 08-mobile-robot.md             # XLeRobot 移動平台 + 導航
├── references/
│   ├── hardware.md                    # 硬體 BOM、接線、3D 列印
│   ├── algorithms.md                  # ACT、Diffusion、VLA、RL 論文與教學
│   ├── software.md                    # LeRobot、XLeRobot、uv、accelerate、Isaac Lab、ROS 2
│   ├── simulation.md                  # Omniverse、Isaac Sim、MuJoCo
│   └── community-projects.md          # hackathon 專案、XLeRobot、Embodied-AI-Guide
└── journal/
    ├── _index.md                      # 日誌索引頁
    └── YYYY-MM-DD-title.md            # 開發日誌（可轉部落格）
```

---

## 3. 核心文件內容規格

### 3.1 `docs/README.md` — 導覽首頁

- 專案一句話介紹
- 硬體清單速覽（SO-101、RealSense D435、Jetson Orin Nano、RTX 8000 工作站）
- **Current Focus 區塊**：只放 3 件事 — 現在做什麼、下一步是什麼、目前 blocker
- 快速導覽表：連到 goals、roadmap、當前進行中的 phase
- 給新手的「從哪裡開始」指引
- 連到最新 journal 的連結

### 3.2 `docs/goals.md` — 願景與目標

- **願景**：打造一台具備雙臂操作 + 自主移動的家庭機器人
- **階段目標**（對應 phases，每個標明「學會什麼」）：
  - Phase 1-5：掌握 LeRobot 全流程
  - Phase 6：雙臂協調
  - Phase 7：模擬環境 + Sim-to-Real
  - Phase 8：XLeRobot 移動平台
- **成功標準**：每個階段有具體可驗證的 milestone
  - 包含 task success rate
  - 包含 **reproducibility 指標**：可重新執行的指令流程、固定的資料集版本、可重現的訓練設定
- **硬體演進圖**：單臂 → 雙臂桌面 → 輪式移動

### 3.3 `docs/roadmap.md` — 全階段 Roadmap

- 表格格式，每列包含：Phase / 名稱 / 優先級 / 前置依賴 / 預期產出 / 狀態
- 優先級標註：
  - P0（現在就做）：Phase 1-3
  - P1（下一步）：Phase 4-5
  - P2（中期）：Phase 6
  - P3（長期願景）：Phase 7-8
- **狀態枚舉值**（固定集合，README 的 Current Focus 和 journal 的 Current Phase 皆依此對齊）：
  - `not-started` — 尚未開始
  - `in-progress` — 進行中（同一時間最多 1-2 個）
  - `blocked` — 被阻擋（需註明 blocker）
  - `done` — 已完成
  - `archived` — 已完成且不再活躍維護
- 底部放依賴關係圖（Mermaid 語法）

### 3.4 `docs/architecture/overview.md` — 系統架構

- 高層模組圖：硬體層 / 控制層 / 感知層 / 學習層 / 應用層
- **明確標註**每個模組是「已實作」還是「未來規劃」
- 目前只詳寫 Phase 1 會碰到的部分
- 預留 placeholder 區塊（標明「Phase N 完成後補充」）
- 資料流：遙操作 → 錄製 → 訓練 → 推論的端到端 pipeline

---

## 4. Phase 文件模板

每份 phase 文件統一使用以下 10 個區塊：

```markdown
# Phase N — 標題

## 目標
這個階段要達成什麼（具體、可驗證）

## 前置條件
需要先完成哪些 phase / 準備什麼硬體

## 輸入 / 輸出
- 輸入：進入此階段時需要什麼已就緒
- 輸出：完成後會產出什麼

## 步驟
實際操作流程（指令、設定、注意事項）

## 驗證方式
用哪個指令、哪個 log、哪個影片或 metrics 判定成功

## 關鍵連結
這個階段需要的參考資料（從 references/ 精選）

## 成功標準
怎樣算完成（含 reproducibility 指標）

## 產出物
明確列出會留下什麼（config、checkpoint、dataset repo id、校正檔、benchmark 結果）

## 踩坑紀錄
實作過程中遇到的問題與解法（漸進補充）

## 風險 / Blockers
已知風險與潛在阻礙
```

### 各 Phase 摘要

| Phase | 核心內容 | 成功標準 | 關鍵風險 |
|-------|---------|---------|---------|
| **01 硬體組裝** | 組裝、馬達 ID 設定、校正、接線 | `lerobot-calibrate` 通過，所有關節可獨立控制 | USB 頻寬、馬達 ID 衝突 |
| **02 遙操作** | Leader-Follower 控制、RealSense 設定（彩色+深度） | 操作主觀連續可控，記錄實測 latency | RealSense 驅動相容性（macOS 不穩定） |
| **03 資料收集** | 資料集規劃（命名、schema）、錄製、品質檢查 | 完成 50+ 回合 pick & place 資料集，可重現的錄製指令 | 資料品質不一致、儲存空間 |
| **04 訓練** | ACT 訓練、多 GPU（accelerate）、WandB 追蹤 | loss 收斂 + validation/replay/offline eval 有可解釋結果 | CUDA 版本相容、多 GPU 同步 |
| **05 部署** | 本地推論、Jetson 邊緣推論、非同步 PolicyServer | 機器手臂成功執行任務，成功率 > 80% | Jetson CUDA 相容性、推論延遲 |
| **06 雙臂** | 第二隻臂改裝、雙臂校正、協調控制 | 雙臂可獨立或協同執行任務 | 雙臂同步控制延遲、機械干涉 |
| **07 模擬** | Isaac Sim URDF 匯入、Isaac Lab RL、Sim-to-Real | 模擬訓練的策略可在實體臂上執行 | Sim-to-Real gap、URDF 精度 |
| **08 移動機器人** | XLeRobot 輪組、移動底盤、導航 | 機器人可移動到指定位置並執行操作任務 | 導航穩定性、底盤機械強度 |

**填充策略**：Phase 01-05 建立時就填入詳細步驟，Phase 06-08 先寫目標、輸入/輸出、架構大綱，等前面完成後再補細節。

---

## 5. References 文件規格

### 統一模板

```markdown
# 分類標題

> 最後確認日期：YYYY-MM-DD

## 概述
這個分類涵蓋什麼，為什麼重要

## 核心資源（必讀）
最重要的 3-5 個，附一句話說明

## 延伸資源
按子主題分組，附簡短說明

## 備註
使用心得、版本注意事項（漸進補充）
```

每份 reference 文件頂部標註**最後確認日期**，方便判斷資訊是否過期。

### 各文件內容

| 文件 | 核心資源 | 延伸資源 |
|------|---------|---------|
| **hardware.md** | SO-101 官方 BOM、Feetech STS3215 datasheet、RealSense D435 SDK、Jetson Orin Nano setup | 3D 列印設定、XLeRobot 硬體改裝、接線圖 |
| **algorithms.md** | ACT 論文、Diffusion Policy 論文、LeRobot 策略清單 | SmolVLA、Pi0、GR00T N1.5、HIL-SERL、VQ-BeT、Embodied-AI-Guide 演算法章節 |
| **software.md** | LeRobot 官方文件、XLeRobot repo、uv 文件、accelerate 文件 | Isaac Lab API、ROS 2 Humble、WandB robotics、HuggingFace Hub dataset 管理 |
| **simulation.md** | Isaac Sim 文件、Isaac Lab 快速入門、MuJoCo 文件 | XLeRobot 模擬後端、URDF 匯入教學、Domain Randomization、Embodied-AI-Guide 模擬器章節 |
| **community-projects.md** | XLeRobot、eta_0.1、ladybugs-robotics | Neil7281 咖啡送杯、Solo CLI、robocafe |

### `community-projects.md` 專案條目格式

每個專案固定包含：
- 名稱 + 連結
- 一句話說明
- **你可以學什麼**：從這個專案能獲得的具體知識或方法
- **和本專案的關聯**：與 SO-101 專案的對應關係

---

## 6. Journal 規格

### `journal/_index.md` — 日誌索引頁

- **Current Phase 區塊**：指向目前主線階段 + 最新一篇 journal
- 最近 5 篇的連結 + 一句話摘要
- 依月份分組的完整清單
- 標籤索引

**Journal 標籤規則：**
- 使用固定的核心標籤集合，避免語意重複（如 `#teleop` vs `#teleoperation`）
- 命名規則：全小寫、用連字號分隔、英文（與專有名詞一致）
- 核心標籤集合（可隨 phase 推進擴充，但需在此處登記）：
  - `#hardware` — 組裝、接線、馬達、機械結構
  - `#calibration` — 校正相關
  - `#teleop` — 遙操作（Leader-Follower、手把、VR）
  - `#camera` — 攝影機設定與串流（含 RealSense）
  - `#dataset` — 資料集錄製、管理、品質
  - `#training` — 模型訓練（ACT、Diffusion、VLA）
  - `#deployment` — 推論部署（本地、Jetson、非同步）
  - `#dual-arm` — 雙臂相關
  - `#simulation` — 模擬環境（Isaac Sim、MuJoCo）
  - `#mobile` — 移動底盤、導航
  - `#debug` — 除錯、踩坑
  - `#setup` — 環境安裝、依賴管理
- 每篇日誌使用 1-3 個標籤，不超過 3 個

### 日誌模板

```markdown
# YYYY-MM-DD — 標題

> Phase: N | Tags: #tag1, #tag2

## 本週進度
做了什麼（對應哪個 Phase）

## 關鍵發現
學到什麼、發現什麼

## 遇到的問題
問題描述 + 解法（或待解決）

## 下週計畫
接下來要做什麼

## 相關資源
這週參考過的連結
```

每篇日誌設計為可直接轉成部落格文章，無需額外編輯。

---

## 7. 設計原則

1. **漸進式深度**：先高層可讀，再按實作補細節
2. **可執行規格**：每個 phase 是可操作的，不只是說明文
3. **Reproducibility**：所有操作留下可重現的指令、版本、設定
4. **已實作 vs 未來規劃明確標註**：避免新手誤解
5. **向下相容**：結構可演進，phase 可拆成資料夾，不影響其他文件
6. **部落格友好**：journal 天然適合外發
7. **資訊保鮮**：references 標註最後確認日期
