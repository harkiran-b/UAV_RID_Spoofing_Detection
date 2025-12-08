# UAV Remote ID Spoofing Detection via RF Fingerprinting

A proof-of-concept system demonstrating that Radio Frequency Fingerprints (RFF) can be used to detect spoofed UAV Remote Identification (RID) messages. This project simulates UAV transmissions with realistic hardware imperfections and trains machine learning models to distinguish legitimate drones from attackers.

---

## Overview

With FAA regulations now requiring drones to broadcast Remote ID messages, a new attack surface has emerged: **RID spoofing**. An attacker can transmit fake RID messages claiming to be a legitimate drone while flying maliciously or remaining on the ground.

This project demonstrates that even when an attacker perfectly spoofs the *content* of RID messages, the **hardware characteristics of their transmitter** create a unique RF fingerprint that machine learning can detect.

### Key Concept

Every transmitter has manufacturing imperfections that create a unique "fingerprint" in its RF emissions:
- Carrier Frequency Offset (CFO)
- Phase Noise
- I/Q Gain & Phase Imbalance
- DC Offset
- Power Amplifier Compression

Different hardware classes (e.g., DJI drones vs. HackRF SDRs vs. cheap clone drones) have *distinct fingerprint distributions* that ML models can learn to classify.

---

## Repository Structure

```
├── uav_rid_gen2.py          # Data generation & simulation
├── uav_rid_ml_rff2.py       # ML model training & evaluation
├── detecting_spoofed_RID_using_RFF.pdf         # Research paper
└── README.md
```

---

## Quick Start

### Prerequisites

```bash
pip install numpy pandas matplotlib h5py scikit-learn tensorflow imbalanced-learn
```

### 1. Generate Dataset (Fast Test)

Open `uav_rid_gen2.py` in Google Colab and run all cells.

**To adjust generation speed:**

At the bottom of the file, find the `generate_dataset_csv()` call:

```python
# Generate dataset
df = generate_dataset_csv(num_scenarios=50)  # Change this number
```

| Scenarios | Approx. Time | Use Case |
|-----------|--------------|----------|
| 10-50 | Seconds | Quick testing |
| 100-500 | Minutes | Development |
| 1000+ | ~10-30 min | Full training (recommended) |

**To customize drone composition:**

In the `generate_dataset_csv()` function, modify:

```python
uav_config_random = {
    'num_citizens': np.random.randint(1, 3),        # Legitimate drones
    'num_uav_attackers': np.random.randint(0, 3),   # Malicious drones in flight
    'num_ground_attackers': np.random.randint(0, 2) # Ground-based SDR attackers
}
```

### 2. Visualize a Scenario

Before generating the full dataset, you can visualize individual scenarios. Run the visualization section to see:
- UAV positions and flight paths on a map
- Attacker spoofing relationships (dotted lines)
- Received signal amplitudes (distance-dependent)

### 3. Train ML Models

After generating `uav_dataset.csv`, open `uav_rid_ml_rff2.py` and run all cells.

This trains three classifiers:
- **1D CNN** - Deep learning on raw I/Q samples
- **Random Forest** - Traditional ML with extracted features
- **Logistic Regression** - Baseline comparison

---

## Important Notes

### ⚠️ Training Time & Pre-trained Models

**The ML training file (`uav_rid_ml_rff2.py`) expects a CSV dataset file.**

- Training the 1D CNN on a large dataset (~1000 scenarios) takes approximately **1 hour**
- The generated CSV files are too large to include in this repository (hundreds of MB)
- **You must generate your own dataset** using `uav_rid_gen2.py` before running the ML training

**Recommended workflow:**
1. Generate a small dataset (50-100 scenarios) for quick testing
2. For full results replication, generate 1000+ scenarios and train overnight

### Dataset File

The training script expects the dataset at:
```python
df = pd.read_csv("uav_dataset.csv")
```

Make sure to either:
- Run `uav_rid_gen2.py` first (it saves to `uav_dataset.csv` automatically)
- Or adjust the path if you saved elsewhere

---

## Expected Results

With sufficient training data (1000+ scenarios), typical results:

| Model | Accuracy | F1 Score |
|-------|----------|----------|
| 1D-CNN | ~85-95% | ~85-95% |
| Random Forest | ~70-85% | ~70-85% |
| Logistic Regression | ~60-75% | ~60-75% |

*Results vary based on dataset size and class distribution.*

---

## Customization

### Adjusting Scenario Parameters

In `uav_rid_gen2.py`, key configuration blocks:

```python
# Environment bounds
ENV_CONFIG = {
    'lat_min': 35.0, 'lat_max': 35.02,
    'lon_min': -80.0, 'lon_max': -79.98,
}

# Signal parameters
SIGNAL_CONFIG = {
    'snr_db': 20.0,           # Signal-to-noise ratio
    'duration_ms': 10.0,      # Transmission duration
    'sample_rate_hz': 20e6,   # Sample rate
}
```

### Adding New Hardware Classes**
*Under Review*

Extend `gen_hardware_profile()` with new manufacturer specs:

```python
company_specs = {
    0: {...},  # DJI
    1: {...},  # HackRF
    2: {...},  # Generic
    3: {...},  # Your new hardware class
}
```
