NETWORK PROTOCOL CLASSIFICATION FROM YOUTUBE LIVE PACKET CAPTURES

Author: Mounami Chowdhury
Notebook: MounamiChowdhury_ML_Project1.ipynb

=====================================================================
ABOUT THE PROJECT
=====================================================================
A machine learning project that classifies network packets by protocol
(TCP, UDP, QUIC, GQUIC, DNS, TLS, etc.) using packet-capture data
collected during YouTube Live streaming sessions. Four classifiers are
trained and compared: Logistic Regression, K-Nearest Neighbors,
Decision Tree and Random Forest.

=====================================================================
DATASET
=====================================================================
Source files: Multiple YouTube_Live_*.csv packet-capture exports
(Wireshark-style), merged into a single file called merged_dataset.csv.

Size: about 6.26 million packets (6,262,725 rows after cleaning).

Columns:
  No.          - Packet sequence number
  Time         - Timestamp of the packet
  Source       - Source IP address
  Destination  - Destination IP address
  Protocol     - Protocol of the packet (TARGET VARIABLE)
  Length       - Packet length in bytes
  Info         - Packet summary / details

Protocols present (13): TCP, DNS, QUIC, GQUIC, TLSv1, TLSv1.3, UDP,
DCP-AF, THRIFT, CLTP, ar_drone, R-GOOSE, MPEG TS.

Note: The raw CSV files and merged_dataset.csv are not included
because of their size. Place them in the paths expected by the
notebook (see "How to Run the Notebook" below).

=====================================================================
WORKFLOW
=====================================================================
1. Data loading and merging
   Reads all YouTube_Live_*.csv files (latin1 encoding, bad lines
   skipped) and joins them into one DataFrame, saved as
   merged_dataset.csv.

2. Data cleaning
   Checks for missing values and drops 463 rows that have a missing
   Info value.

3. Exploratory data analysis
   - Protocol distribution (count plot and pie chart, with small
     protocols grouped as "Other")
   - Histograms, box plots, histogram with KDE, and density plot of
     packet Length

4. Feature engineering
   The Time column is split into Hour, Minute and Second.
   Features: Length, Hour, Minute, Second
   Target: Protocol (label-encoded)

5. Preprocessing
   - Mean imputation (SimpleImputer)
   - Removal of protocol classes with fewer than 2 samples so a
     stratified split is possible
   - 80/20 stratified train/test split (random_state = 42)
   - Standardization (StandardScaler) for scale-sensitive models

6. Modeling and evaluation
   Four classifiers trained and evaluated using accuracy,
   classification report and confusion matrix.

=====================================================================
MODELS AND RESULTS
=====================================================================
Model                  Key settings                  Test accuracy
---------------------  ----------------------------  -------------
Logistic Regression    max_iter = 1000               91.6 - 92.6 %
K-Nearest Neighbors    n_neighbors = 5               99.27 %
Decision Tree          max_depth = 10                98.98 %
Random Forest          n_estimators = 100,           97.94 %
                       max_depth = 10

The Logistic Regression figure differs slightly between cells because
the train/test split was re-created in later cells after some rare
classes were handled differently.

Important notes on interpreting these results:

- The dataset is heavily imbalanced. GQUIC, UDP and QUIC make up
  almost all packets, while classes like CLTP, R-GOOSE, TLSv1, THRIFT
  and MPEG TS have only a handful of samples.

- Accuracy is dominated by the majority classes. For example,
  Logistic Regression gets precision and recall of 0 on most minority
  protocols and only predicts GQUIC and UDP. Macro-averaged F1 is
  therefore low (about 0.15 for Logistic Regression, about 0.32 for
  Random Forest) even though overall accuracy looks high.

- Features are limited. Only packet length and timestamp components
  are used. Source/destination addresses and the Info field are not
  used by the models, so they cannot reliably separate rare protocols.

=====================================================================
TECH STACK
=====================================================================
- Python 3
- pandas, NumPy
- scikit-learn
- matplotlib, seaborn
- Google Colab (the notebook was developed and run there, with Google
  Drive mounted)

=====================================================================
HOW TO RUN THE NOTEBOOK
=====================================================================
1. Open MounamiChowdhury_ML_Project1.ipynb in Google Colab or Jupyter.

2. Upload the YouTube_Live_*.csv files to /content/ (the merge cell
   uses glob('/content/YouTube_Live_*.csv')).

3. Run the merge cells to create merged_dataset.csv, then copy it to
   Google Drive at MyDrive/merged_dataset.csv. Later cells read it
   from /content/drive/MyDrive/merged_dataset.csv.

4. If you are not using Colab, remove the google.colab mount cell and
   change the file paths to local ones.

5. Run the remaining cells in order.

To install dependencies for a local setup:

   pip install pandas numpy scikit-learn matplotlib seaborn

Note: KNN prediction and plotting on about 6 million rows need a lot
of memory and time. Use a machine with plenty of RAM (or a Colab
high-RAM runtime), or work on a sample of the data.

=====================================================================
POSSIBLE IMPROVEMENTS
=====================================================================
- Address class imbalance (class weights, SMOTE/oversampling, or
  grouping rare protocols).
- Report macro/weighted F1 and per-class recall as the main metrics
  instead of accuracy alone.
- Add more features such as ports, inter-arrival time between
  packets, and encoded source/destination information.
- Tune hyperparameters (for example, cross-validated search for k,
  tree depth and number of trees).
- Try gradient boosting models (XGBoost / LightGBM).
- Check for data leakage and validate on separate capture sessions
  instead of a random packet-level split, since neighbouring packets
  in time are highly correlated.

=====================================================================
PROJECT STRUCTURE
=====================================================================
MounamiChowdhury_ML_Project1.ipynb   - Main notebook
merged_dataset.csv                   - Generated merged dataset (not included)
README.txt                           - This file