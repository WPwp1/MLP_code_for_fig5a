# matrix_for_fig5a

## 1. Project Overview

This project utilized multiple gene feature matrices as input data to perform microbial classification tasks through a Multilayer Perceptron (MLP) model, combined with interpretability methods (e.g., Integrated Gradients) to reveal key classification features. It is suitable for classification and importance analysis in three types of microbiomes (e.g., PAc / NPA / PAd).

## 2. Data Description

This classification task was verified respectively on eight standardized gene feature matrices.
This section provides and details the eight input matrices for Figure 5a in the paper, the three-class microbial grouping file, and the MLP script.

### 2.1 Input Matrices

The input matrices for the MLP model are derived from the processing of 8,075 microbial genome samples. These genomes undergo a series of annotation, functional prediction, and feature extraction steps, resulting in multiple feature matrices representing different aspects of microbial functional characteristics. These include:

| Matrix | Description | Dimensions |
|------|----------|----------|
| `1AA.csv` | Relative frequency matrix of amino acids| 20×8075 |
| `2Cazymes_re.csv` | Functional class matrix annotated by CAZy (Carbohydrate-Active Enzymes) | 2881×8075 |
| `3GapMind_small_carbon.csv` | GapMind-predicted completeness matrix of small-molecule carbon source metabolic pathways | 2194×8075 |
| `4pathway8075.csv` | Highly structured metabolic pathway presence matrix | 3156×8075 |
| `5condon.csv` | Triplet codon usage frequency matrix | 64×8075 |
| `6function.csv` | Gene functional category annotation matrix (e.g., protein synthesis, metabolism, signal transduction) |  22315×8075 |
| `7mmseq8075.csv` | Feature matrix generated from MMseqs2 clustering annotation results | 3111594×8075 |
| `8COG_Type.csv` | Functional category abundance matrix based on eggNOG annotation | 109059×8075 |

All input matrices are in standard .csv format, with rows as samples and columns as features. The matrix values represent counts, frequencies, relative abundances, or binary presence (0/1). Each matrix can be independently used as model input for classification training and interpretation analysis, and correspond in sequence to the ROC results in figure5a of the paper.

### 2.2 Sample Grouping Label File

File: `group.txt`
group.txt is a tab-delimited two-column file, where the first column is the sample ID and the second column is the class label (e.g., PAc, NPA, PAd). A partial example is shown below:

GCA_009914315 NPA
GCA_000178975 PAd
GCA_013410065 PAc

Notes:

- `NPA`: Non-polar and alpine

- `PAc`: Cluster together, cold-environment specialists or psychrophiles

- `PAd`: Scatter dispersedly, cold-environment generalists or psychrotrophs

### 2.3 MLP Script

Script file: `MLP_RF.py`,`MLP_IG.py`
This project uses a deep neural network (Multilayer Perceptron, MLP) to classify gene expression data and includes the following functionalities:

- Data preprocessing and standardization
- Deep neural network model construction and training
- Model evaluation (accuracy, ROC curve, confusion matrix)
- Feature gene extraction
- Result visualization

The neural network used is a Multilayer Perceptron (MLP) with the following configurable structure:

- Input layer: Dimensions equal to the number of features
- Hidden layers: 256 → 128 → 64, fully connected, ReLU  activation, Dropout, BatchNorm
- Output layer: Softmax multiclass
- Loss function: Weighted cross-entropy (supports class imbalance)
- Optimizer: Adam
- Hyperparameter tuning: EarlyStopping, ReduceLROnPlateau

---

Explanatory notes on feature selection:
Script: `MLP_RF.py` uses random forest + chi-square test

In the early stages of this study, to reduce redundant features and enhance model interpretability, we adopted a feature selection strategy combining Random Forest (RF) feature importance scores and Chi-squared test:

- **Random Forest:** Evaluates the overall contribution of each feature to the classification task through an ensemble learning mechanism of multiple decision trees, generating feature importance rankings based on "node purity (Gini importance)" or "information gain."
- **Chi-squared Test:** A univariate statistical test measuring the independence between each feature and the class label, selecting the most significant variables for subsequent analysis.

Limitations of using the random forest + chi-square test method:

- **Decoupled from downstream models:** The feature selection process is independent of the final deep learning model (e.g., MLP), failing to capture truly critical feature combinations in the model's nonlinear decision boundaries, limiting interpretability consistency.
- **Ignores feature interactions:** Univariate methods like the Chi-squared test overlook synergistic effects between multiple features, potentially missing important joint feature patterns.
- **Susceptible to bias:** RF's Gini importance tends to favor features with larger value ranges, possibly exaggerating the contribution of certain features.

Therefore, in later stages of this project, we introduced deep neural network-integrated interpretability methods (e.g., Integrated Gradients, IG) to improve consistency and biological interpretability. In the `MLP_IG.py` script, the Integrated Gradients algorithm is used to interpret the model prediction results, and the results are provided:

- Average feature contribution plots per class (bar chart)
- Class-specific heatmaps
  
