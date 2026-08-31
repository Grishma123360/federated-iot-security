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
