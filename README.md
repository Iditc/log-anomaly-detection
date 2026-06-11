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
| 4 | DeepLog (LSTM) | ⏳ Pending |
| 5 | LogBERT | ⏳ Pending |
| 6 | LLM-based detection (Claude API) | ⏳ Pending |

## Stage 1 — Log Parsing (Drain3)

Parsed 11.2M raw log lines into 55 structured templates using Drain3.

**Example:**
ID  1 | Count:  311 | Template: <*> <*> <*> INFO dfs.DataNode$PacketResponder: PacketResponder <*> for block <*> terminating
ID  2 | Count:  314 | Template: <*> <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.addStoredBlock: blockMap updated: <*> is added to <*> size <*>
ID  3 | Count:  292 | Template: <*> <*> <*> INFO dfs.DataNode$PacketResponder: Received block <*> of size <*> from <*>
ID  4 | Count:  292 | Template: <*> <*> <*> INFO dfs.DataNode$DataXceiver: Receiving block <*> src: <*> dest: <*>
ID  5 | Count:    8 | Template: 081109 <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.allocateBlock: <*> <*>


Each log line gets a `template_id` — reducing millions of unique messages to 55 patterns.

## Stage 2 — Data Analysis

### Block size analysis
- **Normal blocks:** very consistent, almost always 19-20 lines (std=4.8)
- **Anomaly blocks:** high variance, 2-284 lines (std=12.4)
- Blocks with **<10 lines are almost always anomalies** — incomplete lifecycle

<img width="1252" height="616" alt="image" src="https://github.com/user-attachments/assets/7a3faa4d-2979-476f-a971-4a3215cd93d8" />


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
- `t_8` ↔ `t_9`: 100% correlation
- Dropped 9 redundant templates: 55 → 46 useful templates

  <img width="818" height="653" alt="image" src="https://github.com/user-attachments/assets/e0eaa68b-4fca-4398-b1d7-88033a5ddb98" />


### Negative correlation
- `t_35/t_36` vs `t_48/t_49`: correlation ~ -0.8 (when one goes up, the other goes down)

## Stage 2 — Feature Engineering

### Template count features (18)
Selected 18 templates as individual features after removing:
- 9 redundant templates (correlation >0.9)
- 28 rare templates (appear in <0.1% of blocks) — captured by `anomaly_only_count`

<img width="1248" height="610" alt="image" src="https://github.com/user-attachments/assets/b3322140-c0a9-4d8a-b17e-f6ba66e9b5d7" />


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

## Stage 3 — Isolation Forest Results

| Version | Features | Precision | Recall | F1 |
|---------|----------|-----------|--------|-----|
| V1 — per line, 1 feature | 1 | 7% | 6% | 0.06 |
| V2 — per block, 55 raw templates | 55 | 68% | 47% | 0.56 |
| **V3 — 26 engineered features** | **26** | **75%** | **75%** | **0.75** |

Feature engineering improved F1 from 0.06 to **0.75** — fewer features, much better results.

## Stack
- `drain3` — log parsing
- `scikit-learn` — Isolation Forest baseline
- `PyTorch` + `HuggingFace Transformers` — deep learning (upcoming)
- `MLflow` — experiment tracking
- `anthropic` — LLM integration (upcoming)

## Environment
- Google Colab (T4 GPU)
- Google Drive for data storage
