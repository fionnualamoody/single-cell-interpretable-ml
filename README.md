# Interpretable Deep Learning for Single-Cell RNA-seq Classification

An independent machine-learning project investigating cell-type classification
and model interpretability using single-cell RNA sequencing data.

The project compares a classical Logistic Regression baseline with a feed-forward
neural network implemented in PyTorch. Generalisation is evaluated across
experimental batches, and Integrated Gradients is used to investigate which
gene-expression features contribute to neural-network predictions.

## Research questions

1. Can cell types be classified accurately from single-cell RNA-seq expression data?
2. Does a neural network improve performance over a classical linear baseline?
3. Do the models generalise to a held-out experimental batch?
4. Do neural-network attributions identify biologically meaningful gene-level patterns?

## Dataset

The analysis uses a PBMC single-cell RNA-seq dataset loaded through `scvi-tools`.

- 11,990 cells
- 3,346 initial gene features
- 9 cell-type labels
- 2 experimental batches

Batch 0 was used for model development and was split into training and validation
sets. Batch 1 was kept completely separate for final evaluation.

After filtering, normalisation and log transformation, the 1,000 most highly
variable genes were selected using the training set only.

## Experimental design

To reduce information leakage:

- the train/validation split was created using batch 0;
- batch 1 was reserved as the held-out test set;
- gene selection was performed using training data only;
- feature standardisation parameters were learned from training data only;
- model selection and early stopping used validation performance rather than
  held-out test performance.

## Models

### Logistic Regression

A class-weighted Logistic Regression model was used as a classical machine-learning
baseline.

### Neural network

A feed-forward neural network was implemented in PyTorch:

- 1,000 input gene features
- hidden layers of 256 and 64 units
- ReLU activation
- dropout regularisation
- class-weighted cross-entropy loss
- AdamW optimisation
- early stopping based on validation macro-F1

## Results

| Model | Validation Macro F1 | Held-out Accuracy | Held-out Macro F1 |
|---|---:|---:|---:|
| Logistic Regression | 0.9063 | 0.9496 | 0.9200 |
| Neural Network | 0.9135 | 0.9528 | 0.9365 |

Both models generalised strongly to the held-out experimental batch.

The neural network provided only a modest improvement in overall accuracy but
a clearer improvement in macro-F1, indicating more balanced performance across
cell types.

Training loss continued to decrease after validation performance had peaked,
highlighting the importance of early stopping.

## Model interpretation

Integrated Gradients was applied to correctly classified cells from the held-out
batch to investigate gene-level contributions to neural-network predictions.

Biologically plausible attribution patterns were observed across several cell
populations, including:

- **B cells:** CD79A, CD79B, BANK1, CD19
- **CD14+ monocytes:** S100A12, CD14, FCN1
- **CD8 T cells:** GZMK, NKG7, CTSW, CD8A, GZMH
- **NK cells:** KLRF1, GZMB, PRF1, CD160, NKG7
- **Megakaryocytes:** TUBB1, PPBP, GP9, PF4

The heterogeneous `Other` category included genes such as LILRA4, SPIB, IRF8
and GZMB, suggesting that some cells may have a plasmacytoid-dendritic-cell-like
expression profile.

Integrated Gradients attributions describe contributions to model predictions.
They should not be interpreted as differential gene expression, causal
relationships or definitive biological markers.

## Limitations

- The analysis uses one PBMC dataset containing two experimental batches.
- Generalisation to independent datasets, laboratories or sequencing protocols
  was not evaluated.
- The neural-network architecture was deliberately simple and extensive
  hyperparameter optimisation was not performed.
- Cell-type labels were pre-assigned and therefore depend on the quality and
  biological resolution of the original annotations.
- Integrated Gradients results depend on both the trained model and the chosen
  attribution baseline.

## Tools

Python, PyTorch, Scanpy, scvi-tools, scikit-learn, Captum, pandas, NumPy and
Matplotlib.

## Repository structure

```text
single-cell-interpretable-ml/
├── notebooks/
│   └── single_cell_data_exploration.ipynb
├── figures/
├── README.md
└── .gitignore