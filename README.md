# Federated Learning IoT Security 
# Technical Report: Privacy-Preserving Collaborative Intrusion Detection via Federated Averaging (FedAvg)

**Author:** Embedded AI Security Engineering Lab  
**Domain Focus:** Decentralized IoT Protocol Security & Edge AI Privacy

---

## 1. Problem Statement
Centralized anomaly detection pipelines require distributed IoT nodes to continuously stream their raw environmental telemetry logs to a single master cloud server. In mission-critical smart city deployments, this design introduces severe bottlenecks: excessive network bandwidth usage, high data transmission latency, and critical privacy vulnerabilities via eavesdropping.

This project implements a decentralized **Federated Learning (FL)** framework. By deploying a manual **Federated Averaging (FedAvg)** orchestration routine, 4 simulated local client IoT edge nodes collaboratively train a shared 1D-CNN intrusion detector. Each node trains exclusively on its private local data partition and shares only structural model weight parameters—never raw data—with the central orchestrator, safeguarding data privacy at the edge.

---

## 2. Dataset Partitioning & The Non-IID Challenge
The dataset utilizes the windowed sliding environmental telemetry matrices (`synthetic_L64.csv`) generated across a 64-timestep footprint. To replicate authentic edge conditions, the data is partitioned into a strict **Non-IID (Non-Independent and Identically Distributed) distribution** across 4 client device profiles:
* **IoT Node 0 (Skewed):** Saturated with Normal operation + Freeze attacks (0% Replay data).
* **IoT Node 1 (Skewed):** Saturated with Normal operation + Replay attacks (0% Freeze data).
* **IoT Node 2 (Concentrated):** Saturated predominantly with pure Freeze attack streams.
* **IoT Node 3 (Concentrated):** Saturated predominantly with pure Replay attack streams.

---

## 3. Methodology & Architecture
* **Modular Decentralized Engineering:** Built a self-contained client execution function `train_local_model(X_local, y_local, global_weights)` that instantiates a local 1D-CNN replica, synchronizes its weights with the global server, runs 3 local optimization epochs, and returns the modified weight matrix.
* **Weighted Parameter Aggregation (FedAvg):** Built a server-side aggregation function `federated_average(client_weights_list, client_sizes)`. Instead of a simple mean, it calculates a weighted average of client updates scaled directly by each node's private dataset size coefficient:
$$\theta_{t+1} = \sum_{k=1}^{K} \frac{n_k}{n} \theta_t^k$$
* **Orchestration Network Loop:** Programmed a complete 10-round global communication cycle to manage model weight synchronization, edge client training optimization, server-side parameter blending, and global master evaluation.

---

## 4. Performance & Trade-Off Benchmarks

### Federated vs. Centralized vs. Isolated Trade-Off Matrix

| Workspace Architecture Variant | Data Privacy Layer | Evaluation Accuracy | Bandwidth & Privacy Overhead |
| :--- | :--- | :--- | :--- |
| **Centralized Model (Baseline)** | None (Exposes Raw Inputs) | 100.00% | High Network Congestion |
| **Federated Global Model (FedAvg)** | **Strictly Private (Weights Only)** | **~68.50% (Converged)** | **Ultra-Low Data Footprint** |
| **Average Isolated Local-Only Node** | Strictly Private (No Sharing) | ~55.00% (Fails on unobserved anomalies) | Zero Network Communication |

### Core Mitacs Globalink Interview Insights
* **The Non-IID Penalty:** The global federated master model stabilizes at an accuracy boundary (~68.50%) lower than the 100% centralized model. This represents the classic, highly documented "Non-IID Convergence Penalty" in Federated Learning research. It occurs because client gradients pull the model parameters in radically opposing directions during edge optimization loops due to their conflicting attack data profiles.
* **Collaboration Parity:** Despite the Non-IID penalty, the collaborative model massively outperforms isolated local configurations. Because Node 0 or Node 1 have zero exposure to specific attacks in isolation, their local models fail completely when encountering unobserved signatures. The Federated master successfully balances these blind spots safely without ever leaking private data arrays over the network.
## 🔒 Extended Module: Differential Privacy Integration (DP-FedAvg)

### 1. Conceptual Grounding & Threat Model
While Federated Learning natively prevents the leakage of raw sensor datasets by restricting network transmissions to model updates alone, structural weights are still vulnerable to **Inversion Attacks**. A sophisticated adversary can intercept decentralized model updates ($\theta_t^k$) and use reverse-engineering techniques to reconstruct sensitive localized air quality telemetry metrics.

To counter this vulnerability, we integrated **Differential Privacy (DP)** into our localized training pipeline [Tue, Sep 1, 2026]. The mathematical core of DP ensures that the output distribution of our global aggregation remains practically unchanged regardless of whether any single data point or node profile is included in the training set. This relationship is governed by the privacy parameter epsilon ($\epsilon$): a smaller $\epsilon$ guarantees stronger privacy but introduces more noise, typically creating a visible trade-off in overall model performance [Tue, Sep 1, 2026].

---

### 2. Implementation Methodology (DP-SGD-Lite)
We deployed an on-device **DP-SGD-Lite** noise-injection engine directly into our parallel client callback functions before weight arrays leave the edge nodes. This localized security pipeline consists of two strict algorithmic steps:

1. **L2-Norm Weight Clipping:** To prevent a malicious node or extreme time-series outlier sample from dominating the server's parameter space, client model updates are passed through a clipping threshold ($\delta=1.0$). If the Euclidean (L2) norm of a weight layer crosses this boundary, the elements are scaled down proportionally:
$$\bar{\theta} = \theta \cdot \min\left(1, \frac{\delta}{||\theta||_2}\right)$$
2. **Gaussian Noise Injection:** Once bounded, calibrated zero-centered Gaussian statistical noise ($\mathcal{N}(0, \sigma^2)$) is mathematically added to the parameter arrays, effectively obfuscating individual data fingerprints before they are routed back to the central server via the weighted `federated_average` loop [Tue, Sep 1, 2026].

---

### 3. Privacy-vs.-Utility Trade-Off Results
Our Day 3 multi-model experimental sweep explicitly maps out the classic **Privacy-Utility Trade-Off Curve** that defines privacy-preserving edge architectures:

* **No Noise Baseline (Maximum Utility):** Stabilizes at peak accuracy, providing a solid target reference but leaving the network updates exposed to parameter profiling.
* **Low Noise ($\sigma=0.005$):** Achieves strong classification performance with minimal accuracy loss, establishing an optimal operational balance for smart city sensor grids.
* **High Noise ($\sigma=0.08$ - Maximum Privacy):** The injected high-frequency variance destabilizes the server's parameter path, causing the global model's evaluation accuracy to decay significantly. This visual performance drop clearly maps the limit of how much statistical noise can be added before the edge detection network becomes functionally unusable [Tue, Sep 1, 2026].

---

### 4. Technical Scope Limitations & Research Maturity
We note the following scope boundaries honestly to maintain complete transparency:
* **Algorithmic Simplification:** This pipeline implements a localized DP approximation (DP-SGD-Lite) designed to evaluate privacy-utility trends directly within custom prototyping loops. It does not track formal $(\epsilon, \delta)$-differential privacy bounds or manage an active privacy budget accountant across the communication rounds.
* **Production Scaling:** A formal production-grade implementation would require native integration with authoritative enterprise privacy accounting libraries, such as **TensorFlow Privacy** or **PyTorch Opacus**, to dynamically calculate tight mathematical privacy guarantees [Tue, Sep 1, 2026].
