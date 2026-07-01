# 🚀 K-Nearest Neighbors (KNN) Hardware Accelerator on FPGA

A hardware-accelerated implementation of the **K-Nearest Neighbors (KNN)** classification algorithm using **Xilinx Vitis HLS**, **Vivado**, and the **PYNQ-Z2 (Zynq-7000 FPGA)**.

The project demonstrates how a classical Machine Learning algorithm can be mapped into FPGA hardware to achieve low-latency and energy-efficient inference for embedded systems.

---

## 📌 Overview

Traditional implementations of KNN execute on CPUs or GPUs, requiring repeated distance calculations over the complete training dataset.

This project moves the computationally intensive portion of KNN into FPGA programmable logic, allowing:

- Parallel distance computation
- Low inference latency
- Hardware acceleration
- Energy-efficient execution
- Real-time embedded inference

The accelerator communicates with the ARM Processing System through an AXI-Lite interface.

---

## ✨ Features

- Hardware implementation of KNN
- Euclidean distance computation
- K = 3 nearest neighbor search
- Majority voting classifier
- AXI-Lite interface
- Vitis HLS implementation
- Vivado integration
- PYNQ Python API support
- C++ simulation testbench
- FPGA deployment support

---

# Hardware Architecture

```
                   +--------------------------+
                   |      ARM Processor       |
                   |     (Processing System)  |
                   +------------+-------------+
                                |
                          AXI-Lite Bus
                                |
        +-----------------------+----------------------+
        |                                              |
        |          KNN Hardware Accelerator            |
        |                                              |
        | +----------------------+                     |
        | | Distance Calculator  |                     |
        | +----------------------+                     |
        |            |                                 |
        | +----------------------+                     |
        | | K Nearest Selection  |                     |
        | +----------------------+                     |
        |            |                                 |
        | +----------------------+                     |
        | | Majority Voting      |                     |
        | +----------------------+                     |
        |                                              |
        +-----------------------+----------------------+
```

---

# Algorithm Flow

```
Input Query
      │
      ▼
Load Training Dataset
      │
      ▼
Compute Euclidean Distance
      │
      ▼
Find K Nearest Neighbors
      │
      ▼
Majority Voting
      │
      ▼
Predicted Class
```

---

# Technologies Used

| Tool | Purpose |
|------|----------|
| C++ | HLS Implementation |
| Vitis HLS | Hardware Synthesis |
| Vivado | FPGA Design |
| PYNQ | FPGA Runtime |
| Python | FPGA Testing |
| AXI-Lite | PS-PL Communication |
| Zynq-7000 | FPGA Platform |

---

# Dataset

The accelerator is evaluated using the classic **Iris Dataset**.

- 150 Samples
- 4 Features
- 3 Classes

Classes:

- Iris Setosa
- Iris Versicolor
- Iris Virginica

---

# Project Structure

```
KNN-on-FPGA/
│
├── hls/
│   ├── knn_hls.cpp
│   ├── knn_hls.h
│   └── knn_tb.cpp
│
├── vivado/
│   ├── design_1.bd
│   ├── bitstream/
│   └── reports/
│
├── python/
│   └── test_fpga.py
│
├── images/
│
├── docs/
│
└── README.md
```

---

# Build Flow

## 1. Clone Repository

```bash
git clone https://github.com/yourusername/KNN-on-FPGA.git

cd KNN-on-FPGA
```

---

## 2. Open Vitis HLS

Create a project.

Add

- knn_hls.cpp
- knn_hls.h
- knn_tb.cpp

Run

- C Simulation
- C Synthesis
- C/RTL Co-Simulation

Export RTL.

---

## 3. Open Vivado

- Import exported IP
- Create Block Design
- Connect AXI Interface
- Generate Bitstream

---

## 4. Deploy to PYNQ

Copy

```
design_1.bit
design_1.hwh
```

to the board.

Run

```python
from pynq import Overlay

overlay = Overlay("design_1.bit")
```

---

# Testing

Example Query

```text
Input:

5.1
3.5
1.4
0.2

Prediction:

Setosa
```

---

# Optimization Techniques

The design uses several HLS optimizations:

- Loop Pipelining
- AXI-Lite Interface Pragmas
- Static Training Dataset
- Parallel Distance Computation
- Efficient Neighbor Selection

---

# Performance Goals

- Low inference latency
- Hardware parallelism
- Reduced CPU workload
- Energy-efficient execution
- Embedded deployment

---

# Future Improvements

- Dynamic training dataset loading
- Configurable K value
- Manhattan distance support
- Fixed-point arithmetic
- AXI DMA interface
- BRAM optimization
- Larger datasets
- Approximate KNN
- Streaming inference
- Support for additional ML algorithms

---

# Learning Outcomes

This project demonstrates:

- FPGA-based Machine Learning
- High-Level Synthesis (HLS)
- Hardware/Software Co-Design
- Embedded AI
- AXI Protocol
- FPGA Memory Architecture
- Hardware Optimization
- PYNQ Development

---

# References

- Xilinx Vitis HLS Documentation
- Xilinx Vivado Design Suite
- PYNQ Documentation
- Iris Machine Learning Dataset

---

# Author

**Adeeb Ali**

B.Tech Electronics Engineering

Aligarh Muslim University

---

## ⭐ If you found this project useful, consider giving it a Star.
