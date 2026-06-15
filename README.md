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
| 6 | DeepLog LSTM — sequence approach | ✅ Done (negative result) |
| 7 | LogBERT | ⏳ Pending |
| 8 | LLM-based detection (Claude API) | ⏳ Pending |

## Model Comparison

| Model | Type | Approach | F1 | FP | FN |
|-------|------|----------|-----|-----|-----|
| Isolation Forest V1 | unsupervised | per line, 1 feature | 0.06 | — | — |
| Isolation Forest V3 | unsupervised | per block, 26 features | 0.75 | — | — |
| Random Forest | supervised | per block, 26 features | 0.9998 | 15 | 4 |
| XGBoost | supervised | per block, 26 features | 0.99 | 44 | 0 |
| DeepLog LSTM | semi-supervised | sequence prediction | ❌ Failed | — | — |
| LSTM Classifier | supervised | sequence classification | ❌ Failed | — | — |

## Stage 1 — Log Parsing (Drain3)

Parsed 11.2M raw log lines into 55 structured templates using Drain3.

**Example:**


Each log line gets a `template_id` — reducing millions of unique messages to 55 patterns.

## Stage 2 — Data Analysis

### Block size analysis
- **Normal blocks:** very consistent, almost always 19-20 lines (std=4.8)
- **Anomaly blocks:** high variance, 2-284 lines (std=12.4)
- Blocks with **<10 lines are almost always anomalies** — incomplete lifecycle

### Template analysis
- **28 templates appear ONLY in anomaly blocks**, never in normal
- Strongest anomaly indicators:
  - `t_32` (WARN: error deleting block) — appears in **30.4%** of anomaly blocks, **0%** of normal
  - `t_16` — **19.2%** of anomaly blocks, 0% of normal
  - `t_6, t_11, t_12` (replication flow) — **~20%** of anomaly blocks, **0.3%** of normal

### Lifecycle analysis
- Templates `t_4` and `t_5` (block completion) appear in **100%** of normal blocks
- Only **63%** of anomaly blocks have them — 37% failed before completing

### Correlation analysis
- 9 template pairs with correlation >0.9 (redundant information)
- `t_11` ↔ `t_12`: 100% correlation (replication request always triggers transfer)
- Dropped 9 redundant templates: 55 → 46 useful templates

### Negative correlation
- `t_35/t_36` vs `t_48/t_49`: correlation ~ -0.8 (when one goes up, the other goes down)

## Stage 2 — Feature Engineering

### Template count features (18)
Selected 18 templates as individual features after removing:
- 9 redundant templates (correlation >0.9)
- 28 rare templates (appear in <0.1% of blocks) — captured by `anomaly_only_count`

### Engineered features (8)

| Feature | Type | Description |
|---------|------|-------------|
| `total_lines` | numeric | Total log lines in the block |
| `short_block` | binary | 1 if total_lines < 10 |
| `unique_templates` | numeric | Number of different templates in the block |
| `has_error_template` | binary | 1 if block contains t_32 (deletion error) |
| `has_replication` | binary | 1 if block contains t_11 (replication request) |
| `last_template` | numeric | Which template ends the block |
| `missing_lifecycle` | binary | 1 if block is missing t_4 or t_5 (incomplete) |
| `anomaly_only_count` | numeric | Count of templates that only appear in anomaly blocks |

**Total: 26 features** (18 template counts + 8 engineered)

## Stage 3 — Isolation Forest (Unsupervised Baseline)

Three iterations showing the impact of feature engineering:

| Version | Features | F1 |
|---------|----------|-----|
| V1 — per line, 1 feature | 1 | 0.06 |
| V2 — per block, 55 raw templates | 55 | 0.56 |
| V3 — 26 engineered features | 26 | **0.75** |

## Stage 4 — Random Forest (Supervised)

- **F1: 0.9998** — 15 false positives, 4 false negatives out of 115,013 test samples
- Top features: `t_32` (error template), `has_error_template`, `total_lines`
- Train/test accuracy nearly identical (99.99% / 99.98%) — no overfitting

## Stage 5 — XGBoost (Supervised)

- **F1: 0.99** — 44 false positives, **0 false negatives** out of 115,013 test samples
- `scale_pos_weight` used to handle class imbalance — model never misses an anomaly
- Top features: `t_4` (lifecycle completion, 46%), `unique_templates` (26%)
- In cybersecurity, zero false negatives is often preferred over fewer false positives

## Feature Importance Comparison

| Rank | Random Forest | XGBoost |
|------|--------------|---------|
| 1 | t_32 (error template) | t_4 (lifecycle completion) |
| 2 | has_error_template | unique_templates |
| 3 | total_lines | t_32 (error template) |

Different models found different paths to the same goal — Random Forest focuses on error signals, XGBoost on lifecycle completeness.

## Stage 6 — DeepLog LSTM (Negative Result)

Tested two LSTM-based approaches on template sequences:

### Approach 1: Next-template prediction (original DeepLog)
- Trained LSTM to predict the next template given a window of 5
- **Result:** model achieved only 3.3% accuracy at top-5 prediction even on normal blocks
- Even with larger model (128 hidden, 64 embedding) and 20 epochs — no improvement
- The next template is unpredictable because normal HDFS blocks have high variance in template order

### Approach 2: Supervised LSTM classifier
- Fed full template sequences directly into LSTM for binary classification
- **Result:** model predicted everything as Normal (F1=0 for anomaly class)
- With class weights (33x for anomaly): loss stuck at 0.693 (random guessing)
- The sequential patterns don't contain enough discriminative signal

### Why LSTM failed on HDFS
- HDFS log sequences have **high variance in order** even within normal blocks
- The **presence/absence** of templates matters, not their **order**
- Count-based features (what happened) outperform sequence-based features (in what order)
- This explains why tabular models (Random Forest, XGBoost) achieved near-perfect results while LSTM failed

**This is an important finding:** not every problem benefits from deep learning. Understanding the data structure matters more than model complexity.

## Stack
- `drain3` — log parsing
- `scikit-learn` — Isolation Forest, Random Forest
- `xgboost` — gradient boosting
- `PyTorch` — LSTM experiments
- `HuggingFace Transformers` — LogBERT (upcoming)
- `anthropic` — LLM integration (upcoming)

## Environment
- Google Colab (T4 GPU)
- Google Drive for data storage
