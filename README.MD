<div align="center">

![Banner](https://capsule-render.vercel.app/api?type=waving&color=0:3776AB,50:F7931E,100:4C72B0&height=200&section=header&text=Network%20Traffic%20Characterization&fontSize=42&fontColor=ffffff&desc=Behavioral%20Clustering%20with%20PCA%20%26%20K-Means&descSize=18&descAlignY=62&animation=fadeIn)

# 🌐 Behavioral Characterization of Large-Scale Network Traffic
### Using PCA & Unsupervised Clustering

*Discovering how network traffic actually behaves — beyond protocol labels.*

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=plotly&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=for-the-badge)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

![Flows Analyzed](https://img.shields.io/badge/Flows_Analyzed-3.57M-blue?style=flat-square)
![Features](https://img.shields.io/badge/Features-82-blue?style=flat-square)
![Protocols](https://img.shields.io/badge/Protocol_Categories-78-blue?style=flat-square)
![Variance Retained](https://img.shields.io/badge/Variance_Retained-90%25-success?style=flat-square)
![Clusters](https://img.shields.io/badge/Behavioral_Clusters-7-success?style=flat-square)

</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Research Question](#-research-question)
- [Key Results](#-key-results)
- [Repository Structure](#-repository-structure)
- [Setup & Installation](#-setup--installation)
- [Dataset](#-dataset)
- [Project Pipeline](#-project-pipeline)
- [Exploratory Data Analysis](#-exploratory-data-analysis)
- [Feature Engineering](#-feature-engineering)
- [Principal Component Analysis](#-principal-component-analysis-pca)
- [Clustering & Behavioral Archetypes](#-behavioral-traffic-archetypes)
- [Cluster Validation](#-cluster-validation)
- [Key Findings](#-key-findings)
- [Applications](#-applications)
- [Limitations](#-limitations)
- [Future Work](#-future-work)
- [Tech Stack](#-tech-stack)
- [Author](#-author)

---

## 🔎 Overview

Modern networks generate millions of flows across a wide variety of applications and services. Traditional traffic analysis often relies on protocol labels, application signatures, or port numbers — but these identifiers don't necessarily reflect *how* traffic actually behaves.

This project asks a different question: **can network traffic be characterized by its behavior instead of its label?**

Using flow-level network statistics, the pipeline applies exploratory data analysis, feature engineering, dimensionality reduction (PCA), and unsupervised learning (K-Means) to uncover latent behavioral structures across **3.57 million network flows**.

---

## ❓ Research Question

> **Can large-scale network traffic be more meaningfully characterized through behavioral patterns than through protocol labels alone?**

Rather than recovering application identities, the project identifies behavioral traffic archetypes such as:

| Archetype | Description |
|---|---|
| 🔵 Bulk Data Transfers | High throughput, large payloads |
| 🟢 Interactive Communication | Bidirectional, user-driven |
| 🟡 Control-Plane Traffic | ACK-dominated, connection management |
| 🟣 Persistent Sessions | Long-lived, stable |
| ⚪ Idle Connections | Low throughput, long duration |

The goal: understand *how* traffic behaves, not just *what* generated it.

---

## 🏆 Key Results

- Identified **5 dominant behavioral dimensions** underlying network activity: volume, session persistence, protocol semantics, directionality, and burstiness
- Reduced **82 raw features → 18 principal components**, retaining **90% of total variance**
- Segmented traffic into **7 interpretable behavioral archetypes**, independent of protocol labels
- Demonstrated that **protocol identity alone fails to explain traffic behavior** — multiple protocols consistently span several behavioral clusters

---

## 📁 Repository Structure

```text
Network-traffic-characterization/
│
├── network_characterisation_eda.ipynb                    # Exploratory data analysis
├── network_characterisation_feature_engineering...ipynb  # Feature engineering, PCA & clustering
├── requirements.txt                                       # Python dependencies
├── plan.MD                                                 # Project planning & methodology notes
├── README.md
└── images/                                                  # Figures used in README (PCA plots, clusters, etc.)
```

---

## ⚙️ Setup & Installation

```bash
# Clone the repository
git clone https://github.com/AakankshReddy/Network-traffic-characterization.git
cd Network-traffic-characterization

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter and run the notebooks in order
jupyter notebook
# 1. network_characterisation_eda.ipynb
# 2. network_characterisation_feature_engineering...ipynb
```

> 💡 See [`plan.MD`](plan.MD) for the full project planning document and methodology walkthrough.

---

## 📊 Dataset

| Metric | Value |
|---|---|
| Total Flows | ~3.57 Million |
| Features | 82 flow-level features |
| Protocol Categories | 78 |
| Sample used for PCA/Clustering | 100,000 flows |

**Example features:** Flow Duration · Packet Counts · Byte Counts · Packet Length Statistics · Inter-Arrival Times (IAT) · TCP Flag Counts · Active/Idle Time Metrics · TCP Window Sizes

### Sampling Strategy

Due to the computational cost of PCA and K-Means at full scale, a random sample of **100,000 flows** was used for dimensionality reduction and clustering, while the **full 3.57M-flow dataset** informed exploratory analysis (protocol distribution, feature distributions, temporal characteristics, correlation structure). This preserved the dominant traffic behaviors while keeping the modeling stage tractable.

---

## 🔄 Project Pipeline

```text
Raw Network Flows
        │
        ▼
Exploratory Data Analysis
        │
        ▼
Feature Engineering
        │
        ▼
Logarithmic Transformation
        │
        ▼
Standardization
        │
        ▼
Principal Component Analysis
        │
        ▼
K-Means Clustering
        │
        ▼
Behavioral Traffic Profiling
```

---

## 🔬 Exploratory Data Analysis

- **Protocol Distribution:** Highly imbalanced — a small number of applications account for most observed traffic
- **Flow Duration:** Strong heavy-tailed behavior — many short-lived flows alongside persistent long-duration sessions
- **Throughput:** `Flow.Bytes.s` and `Flow.Packets.s` show substantial skewness and outliers
- **Correlation:** Many features are strongly correlated due to shared underlying traffic characteristics (packet/byte/session statistics)

These observations directly motivated the preprocessing and dimensionality-reduction steps below.

---

## 🛠 Feature Engineering

<details>
<summary><b>Removing Non-Behavioral Identifiers</b></summary>
<br>

Removed before modeling: `Flow.ID`, `Source.IP`, `Destination.IP`, `Timestamp`, `Label` — these uniquely identify flows but don't describe behavior.
</details>

<details>
<summary><b>Infinite Value Audit</b></summary>
<br>

Certain metrics can theoretically produce infinite values as flow duration approaches zero. An explicit audit confirmed **no infinite values** in the sampled dataset.
</details>

<details>
<summary><b>Skewness Correction</b></summary>
<br>

Highly skewed features (Flow Duration, Throughput, Packet Counts, IATs) were transformed via:

```python
np.log1p(x)
```

This compresses extreme outliers, preserves ordering, reduces variance imbalance, and improves clustering stability.
</details>

<details>
<summary><b>Binary Feature Audit</b></summary>
<br>

TCP flag features represent binary/count-based events, not continuous measurements — excluded from log-transformation to preserve meaning.
</details>

<details>
<summary><b>Special Handling: Window Size Features</b></summary>
<br>

`Init_Win_bytes_backward` contains sentinel values for unavailable measurements and was excluded from the log-transform stage to avoid distortion.
</details>

<details>
<summary><b>Standardization</b></summary>
<br>

All numerical features were standardized via `StandardScaler()` to mean = 0, std = 1, ensuring equal contribution during PCA and clustering.
</details>

---

## 📉 Principal Component Analysis (PCA)

### Variance Retention

| Variance Retained | Components |
|:---:|:---:|
| 90% | **18** |
| 95% | 25 |
| 99% | 39 |

The project retained **18 principal components**, preserving ~**90% of total variance** while significantly reducing feature-space complexity.

![PCA Variance Plot](image.png)

> Although 18 components were retained, only the **first five** were interpreted in detail — they captured the most dominant, interpretable dimensions. Later components refined these patterns (TCP flag combinations, protocol-specific effects, window-size behavior) rather than introducing new ones.

### Component Interpretation

| Component | Name | Dominant Features | Interpretation |
|---|---|---|---|
| **PC1** | Traffic Volume & Session Intensity | Total packet length, packet counts, subflow byte stats | Overall data exchanged during a flow |
| **PC2** | Session Persistence & Temporal Dynamics | Flow duration, idle times, IAT variability | Long-lived sessions & temporal behavior |
| **PC3** | Protocol & Connection Semantics | Ports, TCP flags, packet length characteristics | Protocol-specific communication patterns |
| **PC4** | Flow Directionality | Forward/backward packet statistics | Balance of forward vs. backward communication |
| **PC5** | Burstiness & Packet Timing | Min IATs, small packet behavior, segment stats | Short-term burst communication patterns |

---

## 🧩 Behavioral Traffic Archetypes

K-Means was applied to the PCA-transformed feature space — the goal was to identify **behaviorally similar groups**, not to recover protocol labels. Each cluster was interpreted using **median feature values, protocol distribution, and average PC scores**.

| Cluster | Archetype | Key Characteristics |
|:---:|---|---|
| 0 | **Persistent High-Volume Sessions** | High volume · long-lived · strong protocol identity |
| 1 | **Bulk Transfer Traffic** | High throughput · large packets · relatively short sessions |
| 2 | **Micro-Transaction Traffic** | Very short duration · minimal payload · small packet counts |
| 3 | **Interactive Application Traffic** | Moderate volume · bidirectional · typical user activity |
| 4 | **Control / Acknowledgement Traffic** | Minimal payload · ACK-dominated · very small packets |
| 5 | **Persistent Data Sessions** | Long duration · continuous payload · high volume |
| 6 | **Idle Persistent Connections** | Extremely long duration · low throughput · stable timing |

<table>
<tr>
<td><img src="image-8.png" width="100%"></td>
<td><img src="image-7.png" width="100%"></td>
</tr>
<tr>
<td><img src="image-6.png" width="100%"></td>
<td><img src="image-5.png" width="100%"></td>
</tr>
<tr>
<td><img src="image-4.png" width="100%"></td>
<td><img src="image-3.png" width="100%"></td>
</tr>
<tr>
<td colspan="2"><img src="image-2.png" width="100%"></td>
</tr>
</table>

---

## ✅ Cluster Validation

### Elbow Method

![Elbow Method](image-1.png)

### Silhouette Analysis

| K | Silhouette Score |
|:---:|:---:|
| 2 | 0.292 |
| **3** | **0.336** ⭐ |
| 4 | 0.329 |
| 5 | 0.290 |
| 6 | 0.288 |
| **7** | **0.290** ✅ *(selected)* |
| 8 | 0.286 |
| 9 | 0.233 |
| 10 | 0.233 |

**K = 3** gave the highest silhouette score but produced overly broad categories that obscured distinct behavioral patterns. **K = 7** was chosen instead — it trades a marginal drop in separation score for **materially richer, more interpretable** behavioral segmentation.

---

## 💡 Key Findings

1. Network traffic naturally organizes into distinct behavioral groups
2. Protocol labels alone do **not** fully explain traffic behavior
3. Five dominant behavioral dimensions emerged: **Volume, Persistence, Protocol Semantics, Directionality, Burstiness**
4. Multiple protocols frequently appear across several behavioral clusters
5. Behavioral characterization offers a **richer representation** of network activity than application labels alone

---

## 🚀 Applications

| Domain | Use Case |
|---|---|
| **Network Monitoring** | Summarize millions of flows into a handful of behavioral categories |
| **Traffic Engineering** | Identify bulk transfers, interactive traffic, and persistent sessions for better resource allocation |
| **Security Analytics** | Use behavioral clusters as a baseline for anomaly detection and traffic profiling |
| **Encrypted Traffic Analysis** | Characterize communication behavior without inspecting packet payloads |

---

## ⚠️ Limitations

- PCA and clustering were performed on a 100,000-flow sample, not the full dataset
- Cluster labels are behavioral interpretations, not ground-truth categories
- K-Means assumes roughly spherical clusters and may miss more complex structures
- Behavioral patterns may drift over time as network usage evolves

---

## 🔮 Future Work

- [ ] Isolation Forest–based anomaly detection
- [ ] Cluster stability analysis
- [ ] Real-time traffic classification
- [ ] Automated traffic policy recommendation
- [ ] Behavioral drift detection over time

---

## 🧰 Tech Stack

<div align="center">

| Category | Tools |
|---|---|
| **Language** | ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) |
| **Data Handling** | ![Pandas](https://img.shields.io/badge/-Pandas-150458?style=flat-square&logo=pandas&logoColor=white) ![NumPy](https://img.shields.io/badge/-NumPy-013243?style=flat-square&logo=numpy&logoColor=white) |
| **Modeling** | ![scikit-learn](https://img.shields.io/badge/-scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white) |
| **Visualization** | ![Matplotlib](https://img.shields.io/badge/-Matplotlib-11557C?style=flat-square) ![Seaborn](https://img.shields.io/badge/-Seaborn-4C72B0?style=flat-square) |
| **Environment** | ![Jupyter](https://img.shields.io/badge/-Jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white) |

</div>

---

## 👤 Author

**K. Aakanksh Reddy**
ECE Undergraduate, IIT Kharagpur

**Areas of Interest:** Digital Design · Machine Learning & Data · Signal Processing · Control Systems

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/AakankshReddy)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/kadukuntla-aakanksh-reddy-a985a3387/)
