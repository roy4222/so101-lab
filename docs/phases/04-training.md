# Phase 04 — 訓練

## 1. 目標

使用收集的資料集訓練 ACT 策略模型，設定多 GPU 訓練環境，追蹤實驗結果，產出可用的 checkpoint。

## 2. 前置條件

- [Phase 03 — 資料收集](./03-data-collection.md) 已完成
- 訓練工作站：5x RTX 8000
- WandB 帳號（建議）
- 參考 [硬體清單](../references/hardware.md)

## 3. 輸入 / 輸出

| 項目 | 說明 |
|------|------|
| **輸入** | HuggingFace 上的 LeRobotDataset |
| **輸出** | 訓練好的 ACT checkpoint、訓練 log、超參數記錄 |

## 4. 步驟

### Step 1 — 訓練環境設定

```bash
uv pip install lerobot "lerobot[feetech]" accelerate wandb

# 確認 GPU 數量
python -c "import torch; print(torch.cuda.device_count())"
# 預期輸出：5
```

### Step 2 — 單 GPU 訓練（先驗證流程）

先用單張 GPU 跑一個短訓練，確認流程暢通：

```bash
lerobot-train \
  --policy=act \
  --dataset.repo_id=<your-username>/so101_pick_and_place_cube
```

### Step 3 — 多 GPU 訓練

設定 accelerate：

```bash
accelerate config
```

啟動多 GPU 訓練：

```bash
accelerate launch \
  --multi_gpu \
  --num_processes=5 \
  $(which lerobot-train) \
  --policy=act \
  --dataset.repo_id=<your-username>/so101_pick_and_place_cube
```

> **注意事項：**
> - Effective batch size = `batch_size` x `num_gpus`
> - Learning rate 和 training steps **不會自動調整**，需手動 scale
> - WandB logging 只在 main process 執行

### Step 4 — 實驗追蹤

```bash
wandb login
```

記錄項目：

- 超參數（batch size, LR, epochs, chunk size 等）
- 訓練曲線（loss, learning rate schedule）
- 驗證結果

### Step 5 — 評估 Checkpoint

- Offline evaluation：回放動作軌跡，確認合理性
- 模擬 eval（如有模擬環境）
- 檢查 checkpoint 可正常載入

```python
from lerobot.common.policies.act.modeling_act import ACTPolicy
policy = ACTPolicy.from_pretrained("path/to/checkpoint")
```

## 5. 驗證方式

- Loss 持續下降並收斂
- WandB 有完整訓練記錄
- Offline eval 動作軌跡合理
- Checkpoint 可正常載入並推論

## 6. 關鍵連結

- [LeRobot 訓練文件](https://huggingface.co/docs/lerobot)
- [ACT 論文 (Action Chunking with Transformers)](https://arxiv.org/abs/2304.13705)
- [accelerate 多 GPU 設定](https://huggingface.co/docs/accelerate)
- eta_0.1 經驗：小模型 + 獨立編碼器優於大模型

## 7. 成功標準

- [ ] Loss 收斂
- [ ] Validation / replay / offline eval 有可解釋結果
- [ ] Checkpoint 可載入和推論
- [ ] 超參數已完整記錄
- [ ] WandB 實驗已儲存

## 8. 產出物

| 產出物 | 說明 |
|--------|------|
| ACT checkpoint | 訓練完成的模型權重 |
| WandB 實驗連結 | 完整訓練記錄 |
| 超參數記錄 | YAML / JSON 格式 |
| 訓練環境版本記錄 | Python, PyTorch, CUDA, LeRobot 版本 |

## 9. 踩坑紀錄

> 本區段於實作過程中持續更新。

- （待補充）

## 10. 風險 / Blockers

| 風險 | 說明 | 緩解措施 |
|------|------|----------|
| CUDA 版本相容性 | Runtime 13.0 / nvcc 12.0 需確認 PyTorch 相容 | 安裝前確認 PyTorch CUDA 支援矩陣 |
| 多 GPU 梯度同步瓶頸 | 5 張 GPU 間的通訊可能成為瓶頸 | 監控 GPU utilization，調整 batch size |
| 50 回合可能過擬合 | 資料量偏少，模型容易 overfit | 加入 data augmentation、early stopping、監控 validation loss |
