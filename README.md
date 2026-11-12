# Graph-Based Network Intrusion Detection on TON_IoT

**CSE475 Machine Learning · Group 07 · Section 01 · East West University · Fall 2026**
**Track 1: Graph Neural Network** — hosts are nodes, network flows are edges, and the model classifies each flow.

| Member | Student ID |
|---|---|
| Mohammed Afnan Ahmad Bhuiyan | 2024-1-60-228 |
| Md. Jakaria Masud | 2023-3-60-386 |
| Tawhid Hossain Tomal | 2023-1-60-197 |
| Naima Islam | 2022-3-60-215 |

---

## The idea

A typical intrusion detector judges each network flow on its own. Many attacks only show up in how machines connect — a scan is one host contacting many others. We build the traffic as a graph, with each host as a node and each flow as an edge, and train a graph neural network to classify the edges. The model sees both the flow's own features and the behaviour of the two hosts it connects.

## Research question

> Does a graph built from real host connections improve macro-F1 over a top-K feature-similarity graph on TON_IoT under the same split and training budget, and does any advantage hold for binary detection on source hosts unseen during training?

Ngo et al. (2025) reported that a feature-similarity graph beats real network links. No study has tested that claim on hosts the model has never seen. We do.

---

## Repository layout

```
Group7_UNSWNB15_CSE475/
├── README.md
├── report/
│   ├── task1/Group07_TONIoT_task1_report.pdf        # EDA, research gap, research question
│   └── task2/Group07_TONIoT_task2_report.pdf        # Baselines, graph construction, GNN explained, first results
├── code/
│   ├── task1/Group07_TONIoT_task1_eda.ipynb         # EDA (A–H) + graph, leakage, label and split checks
│   └── task2/
│       ├── Group07_TONIoT_task2_baselines.ipynb       # 6 ML baselines, two protocols, full metrics
│       └── Group07_TONIoT_task2_proposed_model.ipynb  # Graph construction + GNN + explanation + first results
├── related_work/
│   ├── Group07_TONIoT_related_work_table.pdf         # Five-paper comparison table
│   └── papers/                                        # The five paper PDFs (links in papers/README.md)
└── models/
    ├── Group07_TONIoT_best.pth                        # Host-graph GNN checkpoint, Protocol M (< 1 MB)
    └── label_map.json                                 # Class index → class name
```

The repository keeps its original name; the dataset was changed from UNSW-NB15 to TON_IoT with faculty approval (see below), so all files are named `Group07_TONIoT_…`.

---

## Dataset

**TON_IoT — network dataset**, Cyber Range and IoT Labs, UNSW Canberra (Moustafa, 2021). We use the training/testing subset, `train_test_network.csv`: **211,043 flows, 44 columns, 10 classes** (normal 50,000; eight attack types with 20,000 each; MITM 1,043). Worst class ratio **48 : 1**.

- Official source: https://research.unsw.edu.au/projects/toniot-datasets (folder `Train_Test_datasets`)
- The data is not stored in this repository.

### Why we changed dataset

Our first choice, UNSW-NB15, had a majority-to-minority ratio of 11,322 : 1 (Normal 600,079 vs Worms 53). On our faculty's advice we moved to TON_IoT, which the course brief lists for Track 1, keeps the source and destination IP columns needed for a host graph, and is far less imbalanced. The faculty approved the change on 17 October 2026. The earlier work is kept in `archive/unsw-nb15/`.

---

## How to run

The notebook is written for Kaggle and runs on **CPU only**.

1. On Kaggle: **File → Import Notebook**, upload `code/task1/Group07_TONIoT_task1_eda.ipynb`.
2. **+ Add Input → Datasets**, add a dataset containing `train_test_network.csv` (or upload the file from the official source as your own dataset).
3. **Run All**. Run the Task 2 notebooks the same way; both print a split fingerprint, which must match.

The Task 1 and baselines notebooks find the file by its columns. The proposed-model notebook sets `DATA_PATH` in its load cell; edit it if your dataset path differs. Outputs go to `figures/`, `tables/` and `findings.json`.

Packages: pandas, numpy, matplotlib, seaborn, scikit-learn, plotly (all preinstalled on Kaggle); `umap-learn` optional.

---

## Rules every experiment follows

1. **Two evaluation protocols.** Each attack type in TON_IoT comes from a few specific machines, so a multiclass split by machine is impossible (Task 1, §P4). Multiclass results use a stratified flow-level 70/15/15 split after removing exact duplicates; binary (normal vs attack) results use a split where train, validation and test share no source host. The test set is used once, at the end.
2. **No identifiers as features.** `ts`, `src_ip`, `dst_ip`, `src_port`, `dst_port` are never model inputs. IPs only define graph nodes.
3. **The graph is built from training flows only.** Validation and test flows are added at evaluation time.
4. **Everything is fitted on training data only** — scalers, encoders, imputers — and refitted inside every cross-validation fold.
5. **Macro-F1 and per-class recall** are the headline metrics, with weighted F1 reported for comparison with published work.
6. **Label quality is checked.** Known ToN-IoT labelling artefacts (Raskovalov et al., 2022) are measured and their effect tested.

---

## Related work

| Paper | Role in our review |
|---|---|
| E-GraphSAGE (Lo et al., 2022) | Method blueprint; comparison target (ToN-IoT multiclass weighted F1 0.87) |
| Raskovalov et al. (2022) | ToN-IoT labelling errors; cleaning moved accuracy 0.87 → 0.97 |
| TKSGF (Ngo et al., 2025) | Opposing hypothesis: similarity graph beats real topology |
| DIGNN-A (Liu & Guo, 2025) | Alternative design: dynamic, line-graph GNN |
| REAL-IoT (Zhan et al., 2025) | Published GNN scores do not transfer to new data |

Full review: `related_work/Group07_TONIoT_related_work_table.pdf`.

---

## Task 2 — first results

Same split in both notebooks (fingerprint `70dd7b8009`). Test macro-F1:

| Model | Protocol M (10 classes) | Protocol B (binary, unseen source hosts) |
|---|---|---|
| XGBoost (best baseline) | **0.9509** | **0.9872** |
| Random Forest | 0.9501 | 0.8791 |
| MLP | 0.9357 | 0.8731 |
| No-graph control (our network, 0 GNN layers) | 0.8841 | 0.9886 |
| Host-graph GNN (ours, 2 EdgeSAGE layers) | 0.6262 | 0.4122 |

In this first run the host graph does **not** help: several attack types are sent from the same few hosts, so host context blurs them (DDoS and password recall collapse), and the GNN stopped early at epoch 13. Task 3 tests longer training, a gated host/flow model and the top-K similarity graph. Full analysis: `report/task2/`.

---

## Project status

| Task | Content | Status |
|---|---|---|
| Task 1 | EDA, five-paper review, research gap, research question | ✅ Complete |
| Task 2 | Baselines, host-graph construction, GNN model + explanation, first results | ✅ Complete |
| Task 3 | Ablation, 5-fold CV, significance test, SHAP + LIME | ⏳ Upcoming |
