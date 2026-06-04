# Log Anomaly Detection

Anomaly detection system for system logs, built step-by-step from classical ML to LLMs.

## Dataset
- HDFS logs from [Loghub](https://github.com/logpai/loghub)
- 2,000 log lines (development), full dataset for final evaluation

## Project Stages

| Stage | Method | Status |
|-------|--------|--------|
| 1 | Log Parsing — Drain3 | ✅ Done |
| 2 | Baseline — Isolation Forest | 🔄 In Progress |
| 3 | DeepLog (LSTM) | ⏳ Pending |
| 4 | LogBERT | ⏳ Pending |
| 5 | LLM-based detection (Claude API) | ⏳ Pending |

## Stack
- `drain3` — log parsing
- `scikit-learn` — baseline models
- `PyTorch` + `HuggingFace Transformers` — deep learning
- `MLflow` — experiment tracking
- `anthropic` — LLM integration

## Environment
- Google Colab (T4 GPU)
- Google Drive for data storage
