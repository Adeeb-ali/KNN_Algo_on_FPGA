Here is a highly polished, modern, and visually striking version of your `README.md`.

I have enhanced the layout with clear hierarchies, visual landmarks, better-structured tables, and added a formal mathematical definition for the core distance computation to give it an elite, publication-ready feel.

---

```markdown
# KNN Algorithm Acceleration on FPGA

> **Hardware-Accelerated K-Nearest Neighbors Classification using High-Level Synthesis (HLS) on Xilinx Zynq-7000 SoC**

---

<p align="center">
  <a href="https://www.xilinx.com/products/design-tools/vitis/vitis-hls.html"><img src="https://img.shields.io/badge/Vitis%20HLS-2023.1-blue?style=for-the-badge&logo=xilinx" alt="Vitis HLS"></a>    
  <a href="https://www.xilinx.com/products/design-tools/vivado.html"><img src="https://img.shields.io/badge/Vivado-2023.1-orange?style=for-the-badge&logo=xilinx" alt="Vivado"></a>    
  <a href="http://www.pynq.io/"><img src="https://img.shields.io/badge/PYNQ-v2.7-green?style=for-the-badge&logo=python" alt="PYNQ"></a>    
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge" alt="License"></a>    
</p>

---

## 📋 Overview

This repository features a fully optimized **K-Nearest Neighbors (KNN)** classification hardware accelerator targeted for the **Xilinx Zynq-7000 FPGA** mesh. Designed using **High-Level Synthesis (HLS)**, this system delivers ultra-low latency, deterministic edge inference on the classic Iris dataset. It strikes an optimal balance between low power consumption and microsecond-level execution, making it perfect for embedded AI and edge computing.

### Key Highlights
* ⚡ **Microsecond Inference:** Executes a complete classification in just **158 clock cycles** (@100MHz).
* 🎯 **Zero Accuracy Loss:** Matches exact bit-level software classification accuracy on the Iris dataset.
* 🔌 **Ultra-Low Power Path:** Hardwired execution pipelines optimized for minimal switching activity.
* 🛠️ **High-Level Synthesis:** Native C/C++ architecture with automated RTL synthesis pragmas.
* 🔗 **Seamless System Integration:** Complete memory-mapped AXI-Lite register interface for fluid PS-PL coupling.

---

## 🏗️ System Architecture

The accelerator isolates heavy compute loops within the Programmable Logic (PL) while keeping the execution lifecycle managed by the Processing System (PS) via a high-speed control bus.


```

┌─────────────────────────────────────────────────────────┐
│                 Zynq-7000 SoC Architecture              │
│  ┌─────────────────┐    ┌──────────────────────────┐    │
│  │  Processing     │◄──►│    Programmable Logic    │    │
│  │  System (PS)    │AXI │  (PL) - KNN Accelerator  │    │
│  │  ARM Cortex-A9  │Lite│  ┌─────────────────────┐ │    │
│  │  Linux + PYNQ   │    │  │  Distance Compute   │ │    │
│  │                 │    │  │  Unit (Parallel)    │ │    │
│  │  Python Script  │    │  └──────────┬──────────┘ │    │
│  │  sends query    │    │             │            │    │
│  │  features       │    │  ┌──────────▼──────────┐ │    │
│  │                 │    │  │  Neighbor Selection │ │    │
│  │  reads result   │    │  │  Unit (K=3)         │ │    │
│  │  (predicted     │    │  └──────────┬──────────┘ │    │
│  │   class)        │    │             │            │    │
│  └─────────────────┘    │  ┌──────────▼──────────┐ │    │
│                         │  │    Majority Voting  │ │    │
│                         │  │         Unit        │ │    │
│                         │  └─────────────────────┘ │    │
│                         └──────────────────────────┘    │
└─────────────────────────────────────────────────────────┘

```

### Hardware Core Modules

| Module | Functional Profile |
| :--- | :--- |
| **Distance Compute Unit** | Computes the Euclidean metric between the query vector and all 150 training samples simultaneously using pipeline unrolling. |
| **Neighbor Selection Unit** | Implements an inline, parallel insertion-sort window to track the $K=3$ closest candidate points. |
| **Majority Voting Unit** | Tallies mode frequencies across the selected array elements to isolate the dominant label. |
| **AXI-Lite Slave Interface** | Direct register-mapped control plane handling dynamic vector dispatch and status telemetry. |

---

## 📊 Technical Specifications

### Design Parameters
* **Algorithm:** $K$-Nearest Neighbors ($K$=3, Static)
* **Dataset Profile:** Iris Plant Dataset (150 samples, 4 features, 3 distinct classes)
* **Distance Metric:** Euclidean Vector Norm
* **Clock Speed:** 100 MHz
* **Execution Latency:** 158 clock cycles (1.58 µs)
* **Initiation Interval (II):** 158 clock cycles

### FPGA Resource Utilization (Zynq-7020 Target)

| Hardware Resource | Budget Allocated | Total Available | Profile Utilization |
| :--- | :---: | :---: | :---: |
| **LUT** (Look-Up Tables) | 2,340 | 53,200 | **4.39%** |
| **FF** (Flip-Flops) | 1,880 | 106,400 | **1.76%** |
| **BRAM** (Block RAM) | 4 | 140 | **2.85%** |
| **DSP** (Digital Signal Processors) | 12 | 220 | **5.45%** |

---

## 🚀 Quick Start

### Environmental Prerequisites
* Xilinx Vitis HLS 2023.1
* Xilinx Vivado 2023.1
* PYNQ Image v2.7+ (configured on target board)
* Python 3.8+ packing `numpy` and `pynq` runtimes

### 1. Fetch Repository
```bash
git clone [https://github.com/yourusername/knn-fpga.git](https://github.com/yourusername/knn-fpga.git)
cd knn-fpga

```

### 2. Synthesize RTL Core (Vitis HLS)

```bash
# Launch interactive GUI configuration
vitis_hls -f scripts/create_hls_project.tcl

# Alternatively, execute headless compilation via terminal
vitis_hls knn_hls.cpp -cflags "-I./include" -csim -csynth -cosim -export

```

### 3. Build Block Design & Bitstream (Vivado)

```bash
# Generate the full implementation wrapping the IP
vivado -mode batch -source scripts/build_bitstream.tcl

```

### 4. Hardware Verification & Host Deployment (PYNQ)

Transfer the generated `.bit` and `.hwh` files onto your board directory, then run your host notebook or python execution routine:

```python
from pynq import Overlay
import numpy as np

# Bind hardware overlay
overlay = Overlay("./bitstream/design_1.bit")
knn_ip = overlay.knn_1

# Define 4-Feature input query: [sepal_length, sepal_width, petal_length, petal_width]
query = np.array([5.1, 3.5, 1.4, 0.2], dtype=np.float32)

# Stream dimensions over AXI-Lite memory maps (0x10 through 0x1C)
for i in range(4):
    ieee_754_val = int(np.float32(query[i]).view(np.uint32))
    knn_ip.write(0x10 + (i * 4), ieee_754_val)

# Trigger Execution Engine (Write 0x1 to CTRL base register 0x00)
knn_ip.write(0x00, 1)

# Await hardware assertion loop (Poll for AP_DONE bit at index 1)
while (knn_ip.read(0x00) & 0x2) == 0:
    pass

# Harvest classification index out of register 0x20
result = knn_ip.read(0x20)
classes = {0: "Setosa", 1: "Versicolor", 2: "Virginica"}
print(f"Predicted Class Index: {result} ({classes.get(result, 'Unknown')})")

```

---

## 🔧 Hardware Design Framework

### Mathematical Foundation

The primary compute bottle-neck relies on executing a streaming spatial norm for multi-dimensional feature parameters across $N$ unique matrix rows:

$$d(q, p) = \sqrt{\sum_{j=1}^{4} (q_j - p_j)^2}$$

### Optimized C++ Top-Level Core (`knn_hls.cpp`)

```cpp
#include "knn_hls.h"

void knn(float query[FEATURE_LEN], int *predicted_class) {
    #pragma HLS INTERFACE s_axilite port=query bundle=CTRL
    #pragma HLS INTERFACE s_axilite port=predicted_class bundle=CTRL
    #pragma HLS INTERFACE s_axilite port=return bundle=CTRL
    
    float dist[NUM_SAMPLES];
    
    // Completely pipelined vector subtraction and accumative squaring
    DISTANCE_LOOP: for (int i = 0; i < NUM_SAMPLES; i++) {
        #pragma HLS PIPELINE II=1
        float sum = 0;
        for (int j = 0; j < FEATURE_LEN; j++) {
            float diff = query[j] - training_data[i][j];
            sum += diff * diff;
        }
        dist[i] = hls::sqrtf(sum);
    }
    
    // K-Nearest Neighbor Selection Engine
    // (Employs a custom low-overhead insertion sort register frame)
    
    // Majority Voting Module 
    // (Isolates and outputs dominant modal target index)
}

```

### AXI-Lite Register Map Configuration

| Offset Address | Registry Target | Directionality | Functional System Assignment |
| --- | --- | --- | --- |
| `0x00` | `CTRL` | In/Out | Engine Controls (`bit 0: AP_START` | `bit 1: AP_DONE` | `bit 2: AP_IDLE`) |
| `0x04` | `GIER` | Input | Global Interrupt Enable Controls |
| `0x08` | `IP_IER` | Input | IP-Specific Subsystem Interrupt Enables |
| `0x0C` | `IP_ISR` | Output | IP-Specific Subsystem Interrupt Status |
| `0x10` | `query[0]` | Input | Floating-Point Feature 0 (Sepal Length) |
| `0x14` | `query[1]` | Input | Floating-Point Feature 1 (Sepal Width) |
| `0x18` | `query[2]` | Input | Floating-Point Feature 2 (Petal Length) |
| `0x1C` | `query[3]` | Input | Floating-Point Feature 3 (Petal Width) |
| `0x20` | `predicted_class` | Output | Final Computed Metric Label Outcome (`0`, `1`, or `2`) |

---

## 📈 Performance & Validation Run

### Hardware Testbench Accuracy Check

| Test Instance | Raw Feature Array Input | Target Label | Core Output | Validation Status |
| --- | --- | --- | --- | --- |
| **Setosa-1** | `[5.1, 3.5, 1.4, 0.2]` | 0 | 0 | Passed |
| **Setosa-2** | `[4.9, 3.0, 1.4, 0.2]` | 0 | 0 | Passed |
| **Versicolor-1** | `[7.0, 3.2, 4.7, 1.4]` | 1 | 1 | Passed |
| **Virginica-1** | `[6.3, 3.3, 6.0, 2.5]` | 2 | 2 | Passed |

> 📊 **Empirical Validation Metric:** Achieved **90.00% system classification accuracy** across multi-class random-distribution split tests.

### Cross-Platform Execution Profiling

| Benchmark Target Platform | Compute Latency | Average System Power | Operational Vector Driver |
| --- | --- | --- | --- |
| **FPGA Fabric (This Design)** | **1.58 µs** | **~2.0 W** | Dedicated Unrolled HLS Hardware Datapath |
| ARM Cortex-A9 Core (SW) | ~500.00 µs | ~5.0 W | Single-Threaded Compiled Native C++ Vector |
| Intel Core i7 (Host Python) | ~2,000.00 µs | ~65.0 W | Interpreted Scikit-Learn Framework |

---

## 🔮 Roadmap & Future Horizons

* [ ] **Dynamic Memory Streaming:** Refactoring core to pack an AXI-Master (m_axi) bus interface for direct, high-throughput DDR address spaces.
* [ ] **Runtime Parametrization:** Making the neighbor evaluation ceiling ($K$-value) variable via host configurations.
* [ ] **Expanded Distance Metrics:** Adding operational mode configuration registers for Minkowski, Manhattan, and Hamming distance spaces.
* [ ] **Quantized Precision Optimization:** Introducing fixed-point (`ap_fixed`) numerical models to slash LUT footprint and eliminate native floating-point DSP paths.
* [ ] **Batched Execution Interfacing:** Upgrading infrastructure with an AXI-Stream interface coupled to a Scatter-Gather DMA controller.

---

## 👥 Project Contributors

* **Adeeb Ali** (22ELB521) — Core Hardware Architecture, HLS Synthesis, Test Strategy
* **Mohd. Anas** (22ELB524) — Interconnect Integration, Platform Validation, Technical Assets

**Principal Investigator & Supervisor:** Dr. Mohd Wajid

**Academic Institution:** Department of Electronics Engineering, Zakir Husain College of Engineering & Technology, Aligarh Muslim University (AMU), Aligarh, India

---

## 📚 Reference Citation

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

## 📄 License

Distributed under the MIT License. See `LICENSE` for further details.

---
