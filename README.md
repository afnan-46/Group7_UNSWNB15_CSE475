# Graph-Based Network Intrusion Detection on UNSW-NB15

**CSE475 Machine Learning · Group 7 · Section 01 · East West University · Fall 2026**

**Track 1: Graph Neural Network.** Hosts are nodes, network flows are edges, and the model classifies each flow.

| Member | Student ID |
|---|---|
| Mohammed Afnan Ahmad Bhuiyan | 2024-1-60-228 |
| Md. Jakaria Masud | 2023-3-60-386 |
| Tawhid Hossain Tomal | 2023-1-60-197 |
| Naima Islam | 2022-3-60-215 |

---

## The idea

A typical intrusion detector judges each network flow alone, as one row of numbers. But many attacks only show up in how hosts connect. A port scan is one host contacting many others. A denial-of-service attack is many hosts converging on one target.

We build the traffic as a graph, with each host as a node and each flow as an edge, and train a graph neural network to classify the edges. When the model classifies a flow, it uses both the flow's own features and the behaviour of the two hosts it connects.

## Research question

> Does a host-topology graph improve macro-F1 over a k-nearest-neighbour feature-similarity graph on UNSW-NB15, under the same three-way host-disjoint split and the same training budget?

---

## Repository contents

```
Group7_UNSWNB15_CSE475/
├── README.md
└── TASK-1/
    ├── group7-unswnb15-task1-eda.ipynb          # EDA notebook, run with all outputs
    ├── Group7_UNSWNB15_task1_report.pdf         # Task 1 report
    └── Group7_UNSWNB15_related_work_table.pdf   # Five-paper literature review
```

Task 2 and Task 3 folders will be added as each task is completed.

---

## Dataset

**UNSW-NB15**, created by the Australian Centre for Cyber Security, UNSW Canberra (Moustafa & Slay, 2015).

We use the **full release** (`UNSW-NB15_1.csv` to `UNSW-NB15_4.csv`, 49 features), because only this release includes the source and destination IP addresses needed to build a host graph. The smaller partitioned release (training and testing CSVs) has no IP columns.

- **Kaggle:** [mrwellsdavid/unsw-nb15](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)
- **Official source:** [UNSW-NB15 dataset page](https://research.unsw.edu.au/projects/unsw-nb15-dataset)

The data is not stored in this repository because of its size (about 590 MB).

---

## How to run

The notebook is written for Kaggle and runs on **CPU only**. It does not need a GPU.

1. Open the notebook on Kaggle: **File → Import Notebook** and upload `TASK-1/group7-unswnb15-task1-eda.ipynb`.
2. Attach the data: **+ Add Input → Datasets**, search `mrwellsdavid/unsw-nb15`, and click **Add**.
3. Click **Run All**. A full run takes about 10 minutes.

The notebook finds the dataset files automatically, so no paths need editing. Figures are saved to `/kaggle/working/figures/` and summary tables to `/kaggle/working/tables/`.

**Settings you can change** (cell 0.3):
- `MAX_ROWS = 700_000`: rows to load. The default takes the first 175,000 rows of each of the four files. Set it to `None` to load all 2,540,044 rows if memory allows.

**Python packages:** pandas, numpy, matplotlib, seaborn, scikit-learn, plotly. These are all preinstalled on Kaggle. `umap-learn` is optional.

---

## Task 1 results

From 700,000 flows of the full release:

| Finding | Result |
|---|---|
| Class balance | 85.73% normal, 14.27% attack |
| Largest vs smallest class | Normal 600,079 vs Worms 53 (11,322 : 1) |
| Majority-class baseline | 85.73% accuracy, but only about 9.2% macro-F1 |
| Duplicate rows | 142,474 (20.35%) |
| Missing values | `is_ftp_login` 49.3%, `ct_flw_http_mthd` 47.2% |
| Highly correlated pairs (\|r\| > 0.95) | 11 |
| Host graph | 48 hosts, 292 host pairs, 700,000 flows |
| Attack sources | All attacks come from 4 IP addresses |
| Leakage test | Using only the IP addresses gives 0.965 macro-F1 |
| Host-disjoint split | 72 / 13 / 15%, zero host overlap, all 10 classes in every part |

### What this means for the model

- **We judge models on macro-F1, not accuracy.** With an 11,322 : 1 imbalance, a model that always predicts "normal" still gets 85.73% accuracy.
- **IP addresses are never used as model input.** A model can reach 0.965 macro-F1 from the IP addresses alone, just by recognising the four attacking machines. IP addresses are used only to decide which flows share a node in the graph.
- **The data is split by host.** No host appears in more than one of the training, validation and test sets, so the model is always tested on machines it has never seen.
- **Duplicates are removed before splitting**, so the same row cannot appear in both the training and test sets.

The full analysis is in the Task 1 report.

---

## Rules every experiment follows

1. Separate training, validation and test sets, split by host. The test set is used only once, for the final evaluation.
2. Identifier columns (`srcip`, `dstip`, `sport`, `dsport`, `stime`, `ltime`) are never model features.
3. The graph is built from training flows only. Validation and test flows are added at evaluation time.
4. Every scaler, imputer and encoder is fitted on the training set only, and refitted inside each cross-validation fold.
5. Results are reported as macro-F1 and per-class recall, plus weighted F1 so they can be compared with published papers.

---

## Project status

| Task | Content | Status |
|---|---|---|
| Task 1 | EDA, five-paper review, research gap, research question | ✅ Complete |
| Task 2 | Baselines, host-graph construction, GNN model, first results | ⏳ Upcoming |
| Task 3 | Ablation, 5-fold CV, significance test, SHAP + LIME | ⏳ Upcoming |

---

## Key references

1. N. Moustafa and J. Slay, "UNSW-NB15: A comprehensive data set for network intrusion detection systems," MilCIS, 2015.
2. W. W. Lo et al., "E-GraphSAGE: A graph neural network based intrusion detection system for IoT," IEEE/IFIP NOMS, 2022. [arXiv:2103.16329](https://arxiv.org/abs/2103.16329)
3. D.-H. Tran and M. Park, "FN-GNN," *Applied Sciences*, 14(16), 6932, 2024.
4. J. Liu and M. Guo, "DIGNN-A," *Computers, Materials & Continua*, 82(1), 2025.
5. P. Ansah et al., "Enhancing network intrusion detection using graph convolutional networks fused with traditional systems," *IJDSA*, 21(1), 2025.
6. R. C. Nwachukwu, "Rethinking graph-based approaches," *JTIF*, 14(1), 2026.

The full list is in the Task 1 report.
