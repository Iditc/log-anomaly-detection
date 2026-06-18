# Log Anomaly Detection

Anomaly detection system for HDFS system logs — from log parsing to LLM-based detection.

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
| 6 | DeepLog LSTM — sequence approach | ✅ Done (negative result) |
| 7 | Zero-shot LLM detection (F1=0.79) | ✅ Done |

## Model Comparison

| Model | Type | Training needed | F1 | Precision | Recall |
|-------|------|----------------|-----|-----------|--------|
| Isolation Forest V1 | unsupervised | features, no labels | 0.06 | 7% | 6% |
| Isolation Forest V3 | unsupervised | features, no labels | 0.75 | 75% | 75% |
| Random Forest | supervised | features + labels | **0.9998** | 100% | 100% |
| XGBoost | supervised | features + labels | 0.99 | 99% | 100% |
| DeepLog LSTM | semi-supervised | sequences | ❌ Failed | — | — |
| Claude Haiku (zero-shot) | LLM | **nothing** | 0.79 | 74% | 85% |

## Stage 1 — Log Parsing (Drain3)

Parsed 11.2M raw log lines into 55 structured templates using Drain3.

**Example:**
