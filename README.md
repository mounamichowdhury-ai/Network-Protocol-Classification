# Network Protocol Classification from YouTube Live Packet Captures

**Author:** Mounami Chowdhury  
**Notebook:** `MounamiChowdhury_ML_Project1.ipynb`

## About the Project

This machine learning project classifies network packets by protocol using packet-capture data collected during YouTube Live streaming sessions.

Four classifiers are trained and compared:

- Logistic Regression
- K-Nearest Neighbors (KNN)
- Decision Tree
- Random Forest

The project covers data loading, cleaning, exploratory data analysis, feature engineering, preprocessing, model training, and evaluation.

## Dataset

**Source:** Multiple `YouTube_Live_*.csv` packet-capture exports.

The CSV files are merged into a single dataset named `merged_dataset.csv`.

- **Dataset size:** Approximately 6.26 million packets
- **Rows after cleaning:** 6,262,725, as recorded in the project description

### Columns

| Column | Description |
|---|---|
| `No.` | Packet sequence number |
| `Time` | Packet timestamp |
| `Source` | Source IP address |
| `Destination` | Destination IP address |
| `Protocol` | Packet protocol; target variable |
| `Length` | Packet length in bytes |
| `Info` | Packet summary or details |

### Protocol Classes

The project description lists 13 protocol classes:

TCP, DNS, QUIC, GQUIC, TLSv1, TLSv1.3, UDP, DCP-AF, THRIFT, CLTP, ar_drone, R-GOOSE, and MPEG TS.

**Dataset availability:** The raw CSV files and `merged_dataset.csv` are not included in the repository because of their size. The source CSV files must be provided separately to reproduce the workflow.

## Project Workflow

### 1. Data Loading and Merging

- Loads CSV files matching `YouTube_Live_*.csv`.
- Uses Latin-1 encoding and skips malformed lines.
- Combines the individual DataFrames into one dataset.
- Saves the merged data as `merged_dataset.csv`.

### 2. Data Cleaning

- Checks for missing values.
- Drops 463 rows with missing values in the `Info` column, as documented in the project.

### 3. Exploratory Data Analysis

The notebook explores protocol distribution and packet length using:

- Protocol count plots and pie charts, with smaller classes grouped as `Other` for selected visualizations.
- Packet-length histograms and box plots.
- Histograms with kernel density estimation (KDE).
- Density plots.

### 4. Feature Engineering

The `Time` column is split into `Hour`, `Minute`, and `Second`.

- **Features:** `Length`, `Hour`, `Minute`, `Second`
- **Target:** `Protocol`, label-encoded

Source and destination IP addresses and the `Info` field are not used as model features in the described workflow.

### 5. Preprocessing

- Uses mean imputation with `SimpleImputer`.
- Removes protocol classes with fewer than two samples for stratified splitting.
- Uses an 80/20 stratified train/test split.
- Applies feature standardization with `StandardScaler` for scale-sensitive models.

The notebook documents `random_state=42` for the train/test split. Reproducibility also depends on whether the same data, class filtering, preprocessing, and split are used throughout the notebook.

### 6. Model Training and Evaluation

Four classifiers are trained and compared using test accuracy, classification reports, and confusion matrices.

## Models and Results

The following are the latest test accuracy values observed in the current Colab run.

| Model | Key settings | Test accuracy |
|---|---|---:|
| Logistic Regression | `max_iter=1000` | 92.60% |
| K-Nearest Neighbors | `n_neighbors=5` | 99.27% |
| Decision Tree | `max_depth=10` | 98.98% |
| Random Forest | `n_estimators=100`, `max_depth=10` | 98.27% |

**Note on reproducibility:** Earlier notebook outputs showed Logistic Regression at 91.55% and Random Forest at 97.94%. These values changed when the notebook was rerun. The results should be treated as run-specific until the same dataset, preprocessing, class filtering, train/test split, and model settings have been verified across all evaluation cells.

### Interpreting the Results

The dataset is heavily imbalanced. GQUIC, UDP, and QUIC account for most packets, while protocols such as CLTP, R-GOOSE, TLSv1, THRIFT, and MPEG TS have relatively few samples.

Consequently, high overall accuracy does not necessarily mean that every protocol is classified well. The notebook's classification reports show zero precision or recall for several minority classes in some model evaluations.

Macro-averaged F1 and per-class recall are important complementary metrics because they help reveal poor performance on less frequent protocols. The project description records approximate macro-F1 values of 0.15 for Logistic Regression and 0.32 for Random Forest; these should be checked against the latest classification reports before being presented as final results.

## Technology Stack

- Python 3
- pandas
- NumPy
- scikit-learn
- matplotlib
- seaborn
- Google Colab

The notebook was developed and run in Google Colab, with Google Drive used in later data-loading steps.

## How to Run the Notebook

1. Open `MounamiChowdhury_ML_Project1.ipynb` in Google Colab or Jupyter.
2. Upload the `YouTube_Live_*.csv` source files to `/content/` in Colab.
3. Run the data-loading and merging cells to generate `merged_dataset.csv`.
4. If later notebook cells read the merged file from Google Drive, place it at `MyDrive/merged_dataset.csv` and ensure the Drive-mounting cell has run.
5. Run the remaining notebook cells in order.

For a local environment, adapt the Google Colab and Google Drive file paths to your local directories.

### Install Dependencies

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```

**Computational requirements:** Processing and modeling millions of packets can require substantial memory and execution time. KNN prediction and plotting on the full dataset may be particularly expensive. Use a sufficiently capable machine or a suitable Colab runtime, or evaluate on a carefully selected sample when appropriate.

## Possible Improvements

- Address class imbalance using class weights, oversampling, SMOTE where appropriate, or carefully justified grouping of rare protocols.
- Report macro-F1, weighted-F1, and per-class recall alongside accuracy.
- Engineer additional features such as ports and packet inter-arrival times, where available.
- Tune hyperparameters using cross-validation.
- Evaluate gradient boosting models such as XGBoost or LightGBM.
- Investigate data leakage and evaluate on separate capture sessions rather than relying only on a random packet-level split, since nearby packets can be highly correlated.

## Project Structure

```text
Network-Protocol-Classification/
├── MounamiChowdhury_ML_Project1.ipynb
├── README.md
└── merged_dataset.csv   # Generated locally; not included in the repository
```

The source CSV files and generated merged dataset must be supplied separately to reproduce the experiments.
