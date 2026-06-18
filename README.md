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

ID  1 | Count:  311 | Template: <*> <*> <*> INFO dfs.DataNode$PacketResponder: PacketResponder <*> for block <*> terminating
ID  2 | Count:  314 | Template: <*> <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.addStoredBlock: blockMap updated: <*> is added to <*> size <*>
ID  3 | Count:  292 | Template: <*> <*> <*> INFO dfs.DataNode$PacketResponder: Received block <*> of size <*> from <*>
ID  4 | Count:  292 | Template: <*> <*> <*> INFO dfs.DataNode$DataXceiver: Receiving block <*> src: <*> dest: <*>
ID  5 | Count:    8 | Template: 081109 <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.allocateBlock: <*> <*>
ID  6 | Count:   20 | Template: <*> <*> 13 INFO dfs.DataBlockScanner: Verification succeeded for <*>
ID  7 | Count:  263 | Template: <*> <*> <*> INFO dfs.FSDataset: Deleting block <*> file <*>
ID  8 | Count:    9 | Template: 081109 <*> <*> INFO dfs.DataNode$DataXceiver: <*> Served block <*> to <*>
ID  9 | Count:   80 | Template: <*> <*> <*> WARN dfs.DataNode$DataXceiver: <*> exception while serving <*> to <*>
ID 10 | Count:   52 | Template: 081110 <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.allocateBlock: <*> <*>
ID 11 | Count:  224 | Template: <*> <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.delete: <*> is added to invalidSet of <*>
ID 12 | Count:   59 | Template: 081110 <*> <*> INFO dfs.DataNode$DataXceiver: <*> Served block <*> to <*>
ID 13 | Count:    1 | Template: 081110 211541 18 INFO dfs.DataNode: 10.250.15.198:50010 Starting thread to transfer block blk_4292382298896622412 to 10.250.15.240:50010
ID 14 | Count:    2 | Template: 081110 <*> 19 INFO dfs.FSNamesystem: BLOCK* ask <*> to delete <*>
ID 15 | Count:   12 | Template: 081111 <*> <*> INFO dfs.DataNode$DataXceiver: <*> Served block <*> to <*>
ID 16 | Count:   55 | Template: 081111 <*> <*> INFO dfs.FSNamesystem: BLOCK* NameSystem.allocateBlock: <*> <*>
ID 17 | Count:    2 | Template: 081111 <*> <*> INFO dfs.DataNode$DataXceiver: Received block <*> src: <*> dest: <*> of size 67108864
ID 18 | Count:    1 | Template: 081111 065254 19 INFO dfs.FSNamesystem: BLOCK* ask 10.250.17.177:50010 to delete blk_-8570780307468499817 blk_-9122557405432088649 blk_-4393063808227796056 blk_8767569714374844347 blk_7079754042611867581 blk_7608961006114219538 blk_-5017273584996436939 blk_-6537833125980536955 blk_7610838808763810123 blk_3300803097775546532 blk_-5120750586032922592 blk_1577274266662884430 blk_765879159867598347 blk_-9076085976403711202 blk_-3198963348573340497 blk_-4645750029177277209 blk_-5136142986912961316 blk_5677959846373741243 blk_2107477892986152528 blk_-4235116161537008844 blk_6082535783543982566 blk_-4809870147222033236 blk_8818706925296961012 blk_-5203577173046267127 blk_189089569009261656 blk_446299976487589160 blk_-3916247521166632303 blk_-3324962406687427922 blk_-1807424528783081572 blk_-6858401049333055963 blk_6036564204960295926 blk_-8140723044408248078 blk_-3800132731140204959 blk_1716344083117307767 blk_-5194808114606613364 blk_-5473871016976323232 blk_2920934363167004552 blk_8736689095894369097 blk_-7642734632751940776 blk_3408482260833769309 blk_118013751374560901 blk_7963891081239759520 blk_3813114133944383323 blk_3042818489384932576 blk_-4570173726231458270 blk_-1564644006975920581 blk_338095650783321996 blk_3150135312641203550 blk_4285859645577726288 blk_3438772130782939627 blk_2634772258588877972 blk_-6795664812575964130 blk_3923069610304693233 blk_-1782996202120067721 blk_2004418049430157212 blk_1932147224007687756 blk_-582901062969027153 blk_5072240701440032119 blk_-7919006477393039068 blk_-7318022361288598312 blk_-6974693594143537436 blk_-5435767047126325206 blk_-5805500288959332434 blk_-7109885589081848850 blk_2161580591957523893 blk_7240227881194993860 blk_-8298405680648445349 blk_-4253026248821272215 blk_8377661448601579317 blk_8029153852899017155 blk_-8754388319080705916 blk_-7844092300527332901 blk_710178463364063355 blk_-5136849989188547884 blk_8393887138377503163 blk_-6950176077776664217 blk_-6488701068659548195 blk_2537458728254532453 blk_364441107933628577 blk_6207861897580168557 blk_8814943807366894581 blk_-4150682644311695471 blk_9174833667156726933 blk_649427218152856001 blk_-7403541028238011236 blk_-334982586592048773 blk_61908781908925992 blk_6385574357371832424 blk_-66376131060945541 blk_1372596948297458670 blk_-3389135155401857220 blk_-6035411221441929663 blk_-5127580069634421247 blk_-5685246533892022418 blk_4977937528993040451 blk_5680538862600094527 blk_-8378747462487962732 blk_425101290285860876 blk_6306622708327890839 blk_-1067866602168873257
ID 19 | Count:    1 | Template: 081111 065303 19 INFO dfs.FSNamesystem: BLOCK* ask 10.250.10.213:50010 to delete blk_4029139044660806713 blk_-5471189807977280544 blk_6708643067868168687 blk_-500678958150296008 blk_-8597840983621849778 blk_-3610057702150392748 blk_-1709606535283888232 blk_-4154362211643572668 blk_-8892080524136798472 blk_5356427838869009345 blk_-6987238639050161133 blk_-5215128860160823363 blk_7186692462976470823 blk_-6538449588297475521 blk_-2165930080589343952 blk_-5524899010031625427 blk_6384439316405471171 blk_-2965258329365213675 blk_118950937507976810 blk_-1717088081766373300 blk_-3911466865418055820 blk_1237334407720045724 blk_-760015977981369567 blk_-6802007379650646616 blk_-7667535133893574689 blk_6865645438678864855 blk_4633996820313194570 blk_7225301266481603731 blk_-4930257130609958866 blk_-4124845864570823487 blk_4927011145115127531 blk_7234346856930822716 blk_7159969052744592746 blk_1296823600557793869 blk_2209319141644287774 blk_-622218131799806364 blk_-8154516246083521409 blk_4466433199471909449 blk_8406894133999850666 blk_991075908349619367 blk_-2081474832657208733 blk_-5573393775847919985 blk_2004177185950968695 blk_4041319486058127641 blk_6449230045010995668 blk_5978265573904271474 blk_-4813738732036414715 blk_4389340532803855247 blk_-857151863616763327 blk_-7200136644339435027 blk_-1454962873426270839 blk_-5012294311590635938 blk_7112727670634942639 blk_3335012758760643328 blk_3382627815322561484 blk_825124020036421636 blk_-8040559034239258688 blk_-5415591001139074826 blk_-1052513063506891954 blk_-1155882018729560343 blk_-5679835604685169040 blk_-4498808851217768984 blk_8345415947062862337 blk_8521655806854586696 blk_7602939593939794410 blk_-4833650023923869528 blk_7237730029042141635 blk_2860897425785746911 blk_-1937193099911148343 blk_5740615689780260922 blk_963252337613423037 blk_5537011318013544619 blk_2626057344048606017 blk_8296499240199635880 blk_7211071078501521087 blk_8823112510768971040 blk_-3366974935992288326 blk_-2947778702643296262 blk_7693891282153136044 blk_4644812717442758529 blk_-5724970555730638200 blk_-3039294462945223064 blk_-1729755380346651221 blk_-6448673813272428418 blk_-7724282460846954976 blk_2698691234887375588 blk_-4043525878322523713 blk_-5195120009388265 blk_8879208244602324204 blk_-5784376901556131897 blk_-5201149273969117873 blk_5253889604362640423 blk_7067050654303940677 blk_8992626816092659826 blk_-488462739843441981 blk_8543991617360374935 blk_1943146154560599630 blk_-9194660123773136535 blk_3351984198891394382 blk_-6759123807563555545
ID 20 | Count:    1 | Template: 081111 080934 19 INFO dfs.FSNamesystem: BLOCK* ask 10.250.14.38:50010 to replicate blk_-7571492020523929240 to datanode(s) 10.251.122.38:50010
ID 21 | Count:    1 | Template: 081111 091733 19 INFO dfs.FSNamesystem: BLOCK* ask 10.251.126.5:50010 to delete blk_-9016567407076718172 blk_-8695715290502978219


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

### Approach 2: Supervised LSTM classifier
- Fed full template sequences directly into LSTM for binary classification
- **Result:** model predicted everything as Normal (F1=0 for anomaly class)
- With class weights (33x for anomaly): loss stuck at 0.693 (random guessing)

### Why LSTM failed on HDFS
- HDFS log sequences have **high variance in order** even within normal blocks
- The **presence/absence** of templates matters, not their **order**
- This explains why tabular models (Random Forest, XGBoost) achieved near-perfect results while LSTM failed
- **Important finding:** not every problem benefits from deep learning

## Stage 7 — Zero-Shot LLM Detection (Claude API)

Sent raw log lines directly to Claude Haiku with no training, no features, no labels — just a prompt asking "is this block normal or anomalous?"

### Prompt engineering iterations

| Version | Prompt | Precision | Recall | F1 |
|---------|--------|-----------|--------|-----|
| V1 | Simple: "classify as Normal or Anomaly" | **95%** | 58% | 0.72 |
| V2 | Added HDFS context + lifecycle description + examples | 74% | **85%** | **0.79** |
| V3 | Added confidence threshold (>5) | 62% | 95% | 0.75 |

### Key findings
- **V2 (domain context) performed best** — F1=0.79 with balanced precision/recall
- Adding HDFS lifecycle knowledge to the prompt improved recall from 58% to 85%
- Confidence thresholds added parsing complexity without improving F1
- Simple, clear prompts with domain context outperform complex prompt engineering
- **Zero-shot LLM (F1=0.79) beats unsupervised Isolation Forest (F1=0.75)** with no training at all

### Trade-offs
- **LLM advantage:** no training data, no feature engineering, works on any log type
- **LLM disadvantage:** slower (API calls), costs money, lower accuracy than supervised models
- **Best use case:** initial triage when no labeled data is available, or as a complement to ML models

## Project Conclusions

### What worked best
1. **Feature engineering was the single biggest factor.** The same Isolation Forest improved from F1=0.06 to 0.75 just by creating better features. Understanding the data matters more than model complexity.
2. **Supervised tabular models dominate** when labeled data is available. Random Forest (F1=0.9998) and XGBoost (F1=0.99) achieved near-perfect results with 26 engineered features.
3. **Zero-shot LLM detection is surprisingly effective** (F1=0.79) with no training at all — useful when you have no labels.

### What didn't work
4. **LSTM failed** because HDFS log sequences don't have strong sequential patterns — template order is too variable even in normal blocks. The lesson: always check if your data structure matches your model's assumptions.

### The right tool for each scenario

| Scenario | Best approach | Expected F1 |
|----------|--------------|-------------|
| Labeled data available | Random Forest / XGBoost | 99%+ |
| No labels, can train | Isolation Forest + feature engineering | ~75% |
| No labels, no training | LLM zero-shot | ~79% |
| Need to understand why | LLM (provides reasoning) | ~79% |

### Skills demonstrated
- Log parsing (Drain3)
- Exploratory data analysis
- Feature engineering (correlation analysis, domain knowledge)
- Classical ML (Isolation Forest, Random Forest, XGBoost)
- Deep learning (PyTorch LSTM)
- LLM integration (Anthropic API, prompt engineering)
- Honest evaluation of negative results

## Stack
- `drain3` — log parsing
- `scikit-learn` — Isolation Forest, Random Forest
- `xgboost` — gradient boosting
- `PyTorch` — LSTM experiments
- `anthropic` — LLM zero-shot detection

## Environment
- Google Colab (T4 GPU)
- Google Drive for data storage
