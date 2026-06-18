# Flow-Level Network Traffic Characterization Using Unsupervised Learning

## Project Overview

### Objective

Analyze 3.57 million real network flows and discover meaningful traffic behavior patterns using flow-level QoS telemetry, dimensionality reduction, clustering, and protocol validation.

Instead of predicting future traffic, the project aims to answer:

* What types of traffic exist in the network?
* Which traffic behaviors dominate bandwidth consumption?
* Can unsupervised learning discover meaningful traffic groups?
* How do discovered clusters relate to known application protocols?

---

# Dataset

## Source

Dataset-Unicauca-Version2-87Atts.csv

## Characteristics

| Property           | Value                        |
| ------------------ | ---------------------------- |
| Rows               | ~3.57 Million                |
| Features           | 87                           |
| Time Span          | 26 Apr 2017 → 15 May 2017    |
| Labels             | BENIGN                       |
| Application Labels | ProtocolName, L7Protocol     |
| Data Type          | Flow-Level Network Telemetry |

---

# Project Architecture

Raw Flows
↓
Data Cleaning
↓
EDA
↓
Feature Engineering
↓
Feature Scaling
↓
Dimensionality Reduction (PCA)
↓
Clustering (MiniBatchKMeans)
↓
Cluster Validation
↓
Traffic Profile Discovery
↓
Protocol Validation
↓
Interactive Dashboard

---

# Phase 1 — Exploratory Data Analysis

## Goals

Understand:

* Protocol distribution
* Throughput distribution
* Packet size behavior
* Flow duration characteristics
* Temporal patterns

---

## Analyses

### Dataset Structure

```python
df.shape
df.info()
df.describe()
```

### Protocol Distribution

```python
df['ProtocolName'].value_counts()
```

Questions:

* Which protocols dominate?
* What percentage of traffic is HTTP?
* What percentage is HTTP_PROXY?

---

### Throughput Analysis

```python
Flow.Bytes.s
Flow.Packets.s
```

Visualizations:

* Histogram
* Boxplot
* Log-Histogram

Questions:

* Is throughput heavily skewed?
* Are there outliers?

---

### Duration Analysis

```python
Flow.Duration
```

Visualizations:

* Histogram
* Log-Histogram

Questions:

* Are most flows short-lived?
* Is there a long-tail distribution?

---

### Packet Size Analysis

```python
Average.Packet.Size
Packet.Length.Mean
Packet.Length.Std
```

Questions:

* Are flows dominated by small packets?
* Are large packets common?

---

### Temporal Analysis

Create:

```python
Hour
Day
Weekday
```

Visualize:

* Traffic over time
* Protocol activity over time

---

# Phase 2 — Feature Engineering

## Objective

Reduce 87 features to a meaningful behavioral feature set.

---

## Remove Identifier Features

Drop:

```text
Flow.ID
Source.IP
Destination.IP
Source.Port
Destination.Port
Timestamp
```

Reason:

Identifiers contain no behavioral information.

---

## Retain Core Behavioral Features

### Throughput Features

```text
Flow.Bytes.s
Flow.Packets.s
```

### Duration Features

```text
Flow.Duration
```

### Packet Features

```text
Average.Packet.Size
Packet.Length.Mean
Packet.Length.Std
Packet.Length.Variance
```

### Timing Features

```text
Flow.IAT.Mean
Flow.IAT.Std
Flow.IAT.Max

Fwd.IAT.Mean
Bwd.IAT.Mean
```

### Directionality Features

```text
Down.Up.Ratio
```

### Activity Features

```text
Active.Mean
Idle.Mean
```

### TCP Features

```text
SYN.Flag.Count
ACK.Flag.Count
RST.Flag.Count
PSH.Flag.Count
```

---

## Heavy-Tail Correction

Network telemetry is highly skewed.

Apply:

```python
np.log1p()
```

to:

```text
Flow.Duration
Flow.Bytes.s
Flow.Packets.s
Total.Fwd.Packets
Total.Backward.Packets
```

before scaling.

---

## Correlation Analysis

Generate:

```python
corr_matrix = df.corr()
```

Visualize:

```python
sns.heatmap()
```

Remove highly redundant features.

---

# Phase 3 — Data Scaling

## Objective

Normalize all retained features.

---

## Method

```python
from sklearn.preprocessing import StandardScaler
```

Transformation:

z = (x - μ) / σ

---

## Validation

Verify:

```python
mean ≈ 0
std ≈ 1
```

for all scaled features.

---

# Phase 4 — Dimensionality Reduction

## Objective

Understand the latent structure of network traffic.

---

## PCA

```python
from sklearn.decomposition import PCA
```

---

## Explained Variance Analysis

Determine:

* Number of components explaining 80% variance
* Number of components explaining 90% variance

Visualization:

```python
Explained Variance Curve
```

---

## 2D Projection

Generate:

```python
PCA(n_components=2)
```

Visualize:

```python
PC1 vs PC2
```

---

## Interpretation

Analyze component loadings.

Example:

```text
PC1 → Throughput Dominance
PC2 → Packet Timing Dominance
```

---

# Phase 5 — Clustering

## Objective

Discover traffic behavior groups.

---

## Model

Use:

```python
MiniBatchKMeans
```

instead of standard KMeans.

Reason:

Dataset contains millions of flows.

---

## Cluster Selection

### Elbow Method

Evaluate:

```python
K = 2 → 15
```

using:

```python
inertia
```

---

### Silhouette Analysis

Use:

```python
silhouette_score()
```

on a sampled subset.

Recommended sample:

```text
50,000–100,000 rows
```

---

## Final Clustering

Fit:

```python
MiniBatchKMeans
```

on the complete dataset.

---

## Visualization

Plot:

```python
PCA Components
```

colored by cluster.

---

# Phase 6 — Traffic Profile Discovery

## Objective

Interpret cluster behavior.

---

## Cluster Statistics

```python
df.groupby('cluster').mean()
```

---

## Example Profiles

### Cluster 0

Characteristics:

```text
Large packets
Long duration
High throughput
```

Interpretation:

```text
Bulk Transfer Traffic
```

---

### Cluster 1

Characteristics:

```text
Small packets
High packet rate
Low duration
```

Interpretation:

```text
Interactive Traffic
```

---

### Cluster 2

Characteristics:

```text
High Down.Up.Ratio
Large packets
```

Interpretation:

```text
Streaming-Oriented Traffic
```

---

## Deliverable

Traffic Behavior Cards:

* Cluster Name
* Throughput
* Packet Size
* Duration
* Dominant Features

---

# Phase 7 — Protocol Validation

## Objective

Validate whether clusters correspond to real applications.

---

## Method

```python
pd.crosstab(
    df['cluster'],
    df['ProtocolName']
)
```

---

## Questions

* Does Cluster 0 correspond to HTTP?
* Does Cluster 1 correspond to HTTP_PROXY?
* Are clusters protocol-specific?
* Do multiple protocols share similar behaviors?

---

## Significance

This step validates whether unsupervised learning has discovered meaningful traffic classes.

This is the strongest section of the project.

---

# Phase 8 — Cluster Explainability

## Objective

Identify which features define each cluster.

---

## Method

Train:

```python
RandomForestClassifier
```

to predict:

```python
cluster
```

using the engineered features.

---

## Feature Importance

Analyze:

```python
feature_importances_
```

Questions:

* What creates Cluster 0?
* What separates Cluster 1 from Cluster 2?
* Which QoS metrics matter most?

---

# Phase 9 — Anomaly Exploration (Optional)

## Objective

Discover unusual traffic flows.

---

## Models

### Isolation Forest

```python
IsolationForest
```

### DBSCAN

```python
DBSCAN
```

---

## Important Note

Run anomaly detection on:

```text
Original Feature Space
```

not solely on PCA-reduced features.

Reason:

Anomalies often exist in low-variance dimensions removed by PCA.

---

# Phase 10 — Interactive Dashboard

## Technology

```python
Streamlit
Plotly
```

---

## Dashboard Sections

### Overview

* Total Flows
* Protocol Distribution
* Throughput Metrics

---

### Protocol Analytics

* Traffic by Protocol
* Bandwidth by Protocol

---

### PCA Visualization

Interactive scatter plot.

---

### Cluster Explorer

Display:

* Cluster Profiles
* Dominant Characteristics
* Throughput Statistics

---

### Endpoint Explorer

Top:

* Destination IPs
* Bandwidth Consumers

---

# Repository Structure

```text
network_traffic_project/

├── data/
├── notebooks/
│   ├── 01_EDA.ipynb
│   ├── 02_Feature_Engineering_and_PCA.ipynb
│   ├── 03_Clustering_and_Validation.ipynb
│   ├── 04_Traffic_Profile_Analysis.ipynb
│
├── dashboard/
│   └── app.py
│
├── reports/
│   └── final_report.pdf
│
└── README.md
```

---

# Resume Bullet

Analyzed 3.57M network flows using flow-level QoS telemetry, engineered behavioral features, applied PCA and MiniBatchKMeans to discover traffic behavior groups, validated clusters against application-layer protocols, and developed an interactive analytics dashboard for large-scale network traffic exploration.

---

# Expected Technical Outcomes

* Large-scale data processing
* Feature engineering
* Statistical analysis
* PCA
* MiniBatchKMeans
* Cluster validation
* Explainable ML
* Network traffic analytics
* Interactive visualization

---


