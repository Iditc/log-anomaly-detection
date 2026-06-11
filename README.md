# Log Anomaly Detection

Anomaly detection system for HDFS system logs — from log parsing to deep learning.

## Dataset
- **Source:** HDFS logs from [Loghub](https://github.com/logpai/loghub) (Hadoop Distributed File System)
- **Size:** 11,175,629 log lines, 575,061 blocks
- **Labels:** 558,223 Normal, 16,838 Anomaly (2.9%)
- **Origin:** Real production logs from UC Berkeley Hadoop cluster

## Project Stages

| Stage | Method | Status |
|-------|--------|--------|
| 1 | Log Parsing — Drain3 | ✅ Done |
| 2 | Feature Analysis & Engineering | ✅ Done |
| 3 | Baseline — Isolation Forest (F1=0.75) | ✅ Done |
| 4 | Random Forest (F1=0.9998) | ✅ Done |
| 5 | XGBoost (F1=0.99) | ✅ Done |
| 6 | DeepLog (LSTM) | ⏳ Pending |
| 7 | LogBERT | ⏳ Pending |
| 8 | LLM-based detection (Claude API) | ⏳ Pending |

## Model Comparison

| Model | Type | F1 | Precision | Recall | FP | FN |
|-------|------|-----|-----------|--------|-----|-----|
| Isolation Forest V1 | unsupervised | 0.06 | 7% | 6% | — | — |
| Isolation Forest V3 | unsupervised | 0.75 | 75% | 75% | — | — |
| Random Forest | supervised | 0.9998 | 100% | 100% | 15 | 4 |
| XGBoost | supervised | 0.99 | 99% | 100% | 44 | 0 |

## Stage 1 — Log Parsing (Drain3)

Parsed 11.2M raw log lines into 55 structured templates using Drain3.

**Example:**
