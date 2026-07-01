readme = """# KNN Algorithm on FPGA

> B.Tech Project Phase-1 | Department of Electronics Engineering, ZHCET, AMU  
> Hardware-accelerated K-Nearest Neighbors classification using High-Level Synthesis on Xilinx Zynq-7000

---

## Overview

This project implements a hardware-accelerated **K-Nearest Neighbors (KNN)** classifier on the **Xilinx Zynq-7000 FPGA** using **Vitis HLS**. The accelerator classifies Iris dataset samples with deterministic low-latency inference, making it suitable for embedded and real-time edge AI applications.

### Key Features
- **Dataset:** Iris (150 samples, 4 features, 3 classes)
- **Algorithm:** KNN with K=3, Euclidean distance
- **Latency:** 158 clock cycles @ 100 MHz (~1.58 µs per classification)
- **Interface:** AXI-Lite for PS-PL communication
- **Platform:** Zynq-7000 SoC with PYNQ v2.7

---

## Architecture

```
┌─────────────────┐      AXI-Lite      ┌─────────────────────────────┐
│  Processing     │◄─────────────────►│     Programmable Logic      │
│  System (PS)    │                   │   KNN Accelerator           │
│  ARM Cortex-A9  │                   │  ┌─────────────────────┐    │
│  Linux + PYNQ   │                   │  │ Distance Compute    │    │
│                 │                   │  │ Unit (Euclidean)    │    │
│  Python script  │                   │  └──────────┬──────────┘    │
│  sends query    │                   │             │             │
│  features       │                   │  ┌──────────▼──────────┐    │
│                 │                   │  │ Neighbor Selection  │    │
│  reads result   │                   │  │ (K=3, insertion)    │    │
│  (class label)  │                   │  └──────────┬──────────┘    │
└─────────────────┘                   │  ┌──────────▼──────────┐    │
                                      │  │ Majority Voting     │    │
                                      │  │ Unit                │    │
                                      │  └─────────────────────┘    │
                                      └─────────────────────────────┘
```

### Hardware Modules
| Module | Description |
|--------|-------------|
| **Distance Compute Unit** | Calculates Euclidean distance between query and all 150 training samples |
| **Neighbor Selection Unit** | Tracks K=3 nearest neighbors using inline insertion sort |
| **Majority Voting Unit** | Determines the dominant class among selected neighbors |
| **AXI-Lite Slave** | Register-mapped control and data interface |

---

## Specifications

### Design Parameters
- **Clock Frequency:** 100 MHz
- **Latency:** 158 cycles (1.58 µs)
- **Initiation Interval (II):** 158 cycles
- **Distance Metric:** Euclidean
- **Training Data:** Stored in FPGA BRAM (150 × 4 float features + 150 labels)

### Resource Utilization (Zynq-7020)

| Resource | Used | Available | Utilization |
|----------|------|-----------|-------------|
| LUT      | 2,340 | 53,200 | 4.39% |
| FF       | 1,880 | 106,400 | 1.76% |
| BRAM     | 4 | 140 | 2.85% |
| DSP      | 12 | 220 | 5.45% |

---

## Quick Start

### Prerequisites
- Xilinx Vitis HLS 2023.1
- Xilinx Vivado 2023.1
- PYNQ Image v2.7+ (on Zynq board)
- Python 3.8+ with `numpy`, `pynq`

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/knn-fpga.git
cd knn-fpga
```

### 2. HLS Synthesis
```bash
vitis_hls -f scripts/create_hls_project.tcl
```

### 3. Generate Bitstream
```bash
vivado -mode batch -source scripts/build_bitstream.tcl
```

### 4. Deploy on FPGA (PYNQ)
```python
from pynq import Overlay
import numpy as np

overlay = Overlay("./bitstream/design_1.bit")
knn_ip = overlay.knn_1

# Input query: [sepal_length, sepal_width, petal_length, petal_width]
query = np.array([5.1, 3.5, 1.4, 0.2], dtype=np.float32)

# Write features to AXI-Lite registers (0x10 - 0x1C)
for i in range(4):
    val = int(np.float32(query[i]).view(np.uint32))
    knn_ip.write(0x10 + (i * 4), val)

# Start execution
knn_ip.write(0x00, 1)

# Wait for AP_DONE (bit 1)
while (knn_ip.read(0x00) & 0x2) == 0:
    pass

# Read result from 0x20
result = knn_ip.read(0x20)
classes = {0: "Setosa", 1: "Versicolor", 2: "Virginica"}
print(f"Predicted: {classes.get(result, 'Unknown')}")
```

---

## AXI-Lite Register Map

| Offset | Register | Direction | Description |
|--------|----------|-----------|-------------|
| 0x00 | CTRL | R/W | AP_START (bit 0), AP_DONE (bit 1), AP_IDLE (bit 2) |
| 0x04 | GIER | W | Global Interrupt Enable |
| 0x08 | IP_IER | W | IP Interrupt Enable |
| 0x0C | IP_ISR | R | IP Interrupt Status |
| 0x10 | query[0] | W | Feature 0: Sepal Length (IEEE-754 float) |
| 0x14 | query[1] | W | Feature 1: Sepal Width |
| 0x18 | query[2] | W | Feature 2: Petal Length |
| 0x1C | query[3] | W | Feature 3: Petal Width |
| 0x20 | predicted_class | R | Output class label (0, 1, or 2) |

---

## Core Implementation

```cpp
void knn(float query[FEATURE_LEN], int *predicted_class) {
    #pragma HLS INTERFACE mode=s_axilite port=return
    #pragma HLS INTERFACE mode=s_axilite port=query
    #pragma HLS INTERFACE mode=s_axilite port=predicted_class
    
    float dist[NUM_SAMPLES];
    
    // Distance calculation (pipelined)
    DISTANCE_LOOP: for (int i = 0; i < NUM_SAMPLES; i++) {
        #pragma HLS PIPELINE II=1
        float sum = 0.0;
        for (int j = 0; j < FEATURE_LEN; j++) {
            float diff = query[j] - training_data[i][j];
            sum += diff * diff;
        }
        dist[i] = hls::sqrtf(sum);
    }
    
    // K-nearest neighbor selection (K=3)
    // ... insertion sort logic ...
    
    // Majority voting
    // ... count and select max ...
    
    *predicted_class = predicted;
}
```

---

## Results

### Classification Accuracy

| Sample | Features | Expected | Predicted | Status |
|--------|----------|----------|-----------|--------|
| Setosa-1 | [5.1, 3.5, 1.4, 0.2] | 0 | 0 | Pass |
| Setosa-2 | [4.9, 3.0, 1.4, 0.2] | 0 | 0 | Pass |
| Versicolor-1 | [7.0, 3.2, 4.7, 1.4] | 1 | 1 | Pass |
| Virginica-1 | [6.3, 3.3, 6.0, 2.5] | 2 | 2 | Pass |

**Hardware test (10 random samples): 9/10 correct (90%)**

### Performance Comparison

| Platform | Latency | Power |
|----------|---------|-------|
| **FPGA (This Work)** | **1.58 µs** | **~2 W** |
| ARM Cortex-A9 (SW) | ~500 µs | ~5 W |
| Intel Core i7 (Python) | ~2000 µs | ~65 W |

---

## Project Team

| Name | Roll No. | Contribution |
|------|----------|-------------|
| **Adeeb Ali** | 22ELB521 | Core architecture, HLS design, test strategy |
| **Mohd. Anas** | 22ELB524 | System integration, validation, documentation |

**Supervisor:** Dr. Mohd Wajid  
**Institution:** Department of Electronics Engineering, ZHCET, AMU, Aligarh, India

---

## Future Work

- [ ] Dynamic training data loading via DDR/AXI-Master
- [ ] Runtime configurable K value
- [ ] Support for Manhattan, Minkowski, and Hamming distances
- [ ] Fixed-point (`ap_fixed`) arithmetic for lower resource usage
- [ ] Batch inference with AXI-Stream + DMA
- [ ] Extend to SVM and Decision Tree classifiers

---

## Citation

```bibtex
@report{knn_fpga_2025,
  title={Machine Learning Algorithms on FPGA: KNN Implementation},
  author={Adeeb Ali and Mohd. Anas},
  institution={ZHCET, AMU},
  year={2025},
  type={B.Tech. Project Phase-1 Report}
}
```

---

## License

This project is for academic purposes. See `LICENSE` for details.
"""

"""

with open('/mnt/agents/output/README.md', 'w') as f:
    f.write(readme)

print("README.md written successfully!")
print(f"Length: {len(readme)} characters")
