
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
- `t_8` ↔ `t_9`: 100% correlation
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
