# 🚗 Aviral Path — AI-ML based Intelligent Dead Reckoning system for seamless navigation

> An LSTM-based inertial navigation system (INS) that estimates a vehicle's position using only onboard motion sensors — no GPS required — built on the **IO-VNBD** dataset.

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)]()
[![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-orange)]()
[![Status](https://img.shields.io/badge/status-preliminary-yellow)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Why This Matters](#-why-this-matters)
- [Dataset — IO-VNBD](#-dataset--io-vnbd)
- [Approach & Methodology](#-approach--methodology)
- [Model Architecture](#-model-architecture)
- [Results](#-results)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Usage](#-usage)
- [Tech Stack](#-tech-stack)
- [Limitations](#-limitations)
- [Roadmap](#-roadmap--future-work)
- [Team](#-team)
- [Acknowledgments](#-acknowledgments)
- [License](#-license)

---

## 🔍 Overview

**Aviral Path** is a preliminary AI system designed to solve one of the core challenges in autonomous and connected vehicle navigation: **estimating a vehicle's real-time position when GPS signal is unavailable or unreliable** — for example, in tunnels, dense urban canyons, underground parking structures, or during deliberate GPS jamming/spoofing.

Navigation apps freeze or jump when GPS drops — in tunnels, multi-level car parks, urban canyons — and most Indian vehicles have only the driver's smartphone, not a factory inertial system wired to the wheels. The ask is an AI dead-reckoning system that uses the phone's own accelerometer and gyroscope to keep tracking position through a GPS blackout, despite the noise of a phone on a dashboard.

We approach this as an **inertial navigation (dead reckoning)** problem, powered by a deep learning model trained on the **IO-VNBD (Inertial and Odometry Vehicle Navigation Benchmark Dataset)**. Instead of relying on classical, error-prone physics-only dead reckoning, we use an **LSTM (Long Short-Term Memory) neural network** to learn vehicle dynamics directly from raw sensor data — accelerometer, gyroscope, and wheel-speed signals — and predict the vehicle's velocity, which is then integrated into a full 2D trajectory.

This repository contains our preliminary model, training pipeline, and a position-plot evaluation comparing our AI-estimated trajectory against real GPS ground truth.

---

## 🎯 Problem Statement

> *"Teams are required to include the preliminary AI models and the results of the position plot inferenced from the subset of IO-VNBD dataset as part of their proposals submitted for evaluation. During the screening process more datasets will be provided for further evaluation of the AI models."*

Our objective for this stage is to:
1. Build a preliminary AI model trained on a subset of the IO-VNBD dataset.
2. Use that model's output to reconstruct the vehicle's estimated position over time.
3. Visually and quantitatively compare this AI-estimated path against the GPS ground-truth path.
4. Demonstrate the model generalizes to **unseen driving data** (a different driver/trip than it was trained on).

---

## 💡 Why This Matters

GPS is the backbone of modern vehicle navigation — but it is not always available or trustworthy:

- **Signal blockage**: tunnels, underground structures, dense urban high-rises ("urban canyon" effect)
- **Jamming & spoofing**: increasingly common security threats to GPS-dependent systems
- **Multipath errors**: reflected signals in cities that corrupt position accuracy
- **Rural/remote gaps**: inconsistent satellite coverage in certain terrains

Autonomous vehicles, fleet tracking systems, and advanced driver-assistance systems (ADAS) all need a **fallback positioning method** for these scenarios. This is where **AI-assisted inertial navigation** comes in — using the vehicle's own onboard sensors (which are always available, regardless of GPS status) to estimate position using learned motion patterns rather than raw physics integration alone, which suffers from severe drift.

---

## 📊 Dataset — IO-VNBD

**IO-VNBD (Inertial and Odometry Vehicle Navigation Benchmark Dataset)** is a real-world driving dataset combining:

- GPS-derived ground truth (latitude, longitude, height, heading, velocity)
- Inertial Measurement Unit (IMU) data (longitudinal/lateral acceleration, yaw rate)
- Vehicle CAN-bus data (wheel speeds, steering angle, gear, throttle, brake, engine RPM, etc.)

Recorded across multiple drivers and trips, making it well-suited for testing how well a model **generalizes across different driving styles and routes** — a key requirement for any real-world INS system.

### Key columns used in this project

| Column | Purpose |
|---|---|
| ` Velocity (km/hr)` | Primary regression target |
| ` Vertical velocity (km/hr)` | Secondary regression target |
| ` Yaw Rate (deg/sec)` | Heading change rate (used in trajectory reconstruction) |
| ` Heading (degrees)` | Initial heading reference (compass convention) |
| ` Latitude (degrees)` / ` Longitude (degrees)` | Ground-truth position (for evaluation only, never fed to the model) |
| ` Indicated Longitudinal/Lateral Acceleration (g)` | IMU input features |
| ` Steering Angle (degrees)` | Additional motion context |

> ⚠️ Note: GPS-derived columns (lat/lon) are used **only to generate the ground-truth trajectory for evaluation** — they are never used as model input features, since the model must work in GPS-denied conditions.

---

## 🧪 Approach & Methodology

Our pipeline has two major stages:

### Stage 1 — Sequence-to-Value Regression (LSTM)
We frame velocity estimation as a **time-series forecasting problem**: given a sliding window of the last *N* timesteps of sensor readings, predict the vehicle's velocity and vertical velocity at the next timestep.

1. **Feature engineering** — selected motion-relevant sensor columns as input features.
2. **Windowing** — sliding window of historical timesteps used as input sequences.
3. **Train/test split** — chronological split (`shuffle=False`) to preserve time-series integrity — no data leakage from future to past.
4. **Model training** — LSTM network trained to minimize regression loss (MAE/MSE) on velocity targets.
5. **Generalization test** — the trained model is evaluated on a **completely different driver's data** to test real-world robustness, not just memorization of one driving session.

### Stage 2 — Trajectory Reconstruction (Dead Reckoning)
A raw velocity prediction alone isn't a position. To generate the required **position plot**, we convert the model's velocity output into a 2D trajectory using dead-reckoning kinematics:

1. Convert GPS lat/lon ground truth into a local flat X-Y coordinate system (meters), anchored at the trip's starting point.
2. Integrate the sensor's yaw rate over time (starting from the true initial heading) to reconstruct the vehicle's heading angle at every timestep.
3. Combine heading + predicted velocity to compute incremental (dx, dy) displacement at each timestep.
4. Cumulatively sum these displacements to produce the full estimated trajectory.
5. Plot the AI-estimated trajectory against the GPS ground-truth trajectory for visual and quantitative comparison.

This mirrors how real-world INS systems work: **velocity + heading → position**, without ever touching GPS during inference.

---

## 🧠 Model Architecture

```
Input: sliding window of [window_size] timesteps × [n_features] sensor readings
        │
        ▼
   LSTM layer(s)  — learns temporal dependencies in motion data
        │
        ▼
   Dense layer(s) — regression head
        │
        ▼
Output: [Velocity (km/hr), Vertical Velocity (km/hr)] at next timestep
```

- **Framework**: TensorFlow / Keras
- **Loss function**: Mean Squared Error (MSE)
- **Evaluation metric**: Mean Absolute Error (MAE)
- **Optimizer**: Adam
- **Sequence input**: sliding window over time-ordered sensor readings
- **Targets**: `Velocity (km/hr)`, `Vertical velocity (km/hr)`

> Full architecture, hyperparameters, and layer sizes are defined in [`notebooks/IO_VNBD_ML_Master.ipynb`](./notebooks/IO_VNBD_ML_Master.ipynb).

---

## 📈 Results

### Model Performance
| Evaluation | Metric | Value |
|---|---|---|
| Same-driver test set | MAE | `[insert your Test MAE here]` |
| Cross-driver generalization (Driver B) | MAE | `[insert your generalization MAE here]` |

### Position Plot — AI-Estimated Trajectory vs GPS Ground Truth

![Position Plot](./assets/position_plot.png)

*Blue: actual GPS ground-truth path. Orange (dashed): AI-estimated path reconstructed purely from predicted velocity + inertial yaw rate — no GPS used during inference.*

**Final positional drift:** `688.7 meters` over `1035 seconds` (~17 minutes) of driving.

The model's estimated trajectory closely tracks the true path for the initial segment of the drive before gradually diverging — a well-understood characteristic of inertial dead reckoning, where small per-timestep sensor and heading errors accumulate over time. Notably, the AI-driven trajectory closely matches a trajectory generated from *true* (non-predicted) sensor values, indicating the model's velocity predictions themselves are not the primary source of error — the drift stems from the fundamental limitation of uncorrected inertial integration, which motivates our planned future work (see [Roadmap](#-roadmap--future-work)).

---

## 📁 Repository Structure

```
aviral-path/
├── notebooks/
│   └── IO_VNBD_ML_Master.ipynb     # Main training + evaluation + position-plot notebook
├── data/
│   └── [dataset files or download instructions]
├── assets/
│   └── position_plot.png           # Final trajectory comparison plot
├── models/
│   └── velocity_kinematics_lstm.h5 # Saved trained model
├── requirements.txt
├── README.md
└── LICENSE
```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- pip
- (Optional) Google Colab — this project was developed and tested in Colab

### Installation

```bash
git clone https://github.com/rajaditya0806/[repo-name].git
cd [repo-name]
pip install -r requirements.txt
```

### requirements.txt (example)
```
tensorflow>=2.15
pandas
numpy
matplotlib
scikit-learn
```

---

## ▶️ Usage

1. Open `notebooks/IO_VNBD_ML_Master.ipynb` in Jupyter or Google Colab.
2. Place the IO-VNBD dataset CSV(s) in the `data/` folder (or update the file path in the notebook).
3. Run all cells top to bottom:
   - Data loading & preprocessing
   - Model training
   - Evaluation on same-driver test set
   - Cross-driver generalization test
   - Dead-reckoning trajectory reconstruction
   - Final position plot generation
4. The final plot will be saved to `assets/position_plot.png`, and console output will print the model's MAE and final positional drift.

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python |
| Deep Learning | TensorFlow, Keras |
| Data Handling | Pandas, NumPy |
| Visualization | Matplotlib |
| Development Environment | Google Colab / Jupyter Notebook |
| Version Control | Git, GitHub |

---

## ⚠️ Limitations

This is a **preliminary** model, and we're transparent about its current constraints:

- Trained and validated on a limited subset of the IO-VNBD dataset; broader validation is planned once additional datasets are provided during the screening round.
- Uses raw dead-reckoning integration without drift correction, so positional error accumulates over longer durations — a known, expected characteristic of uncorrected INS.
- Currently evaluated on two drivers; wider driver/road-type diversity is needed to fully validate generalization.
- Heading is currently only initialized from the ground-truth compass heading at the start of the test window (a standard INS benchmarking approach, but a real deployed system would need a heading initialization strategy that doesn't depend on GPS at all).

---

## 🗺️ Roadmap / Future Work

- [ ] **Drift correction** — introduce a learned or Kalman-filter-based correction layer to counteract accumulated positional drift over time.
- [ ] **Expanded generalization testing** — validate across the additional datasets provided in the screening round, covering more drivers, vehicles, and road conditions.
- [ ] **Multi-output trajectory prediction** — explore directly predicting displacement/heading-change per timestep instead of deriving position solely from velocity integration.
- [ ] **Sensor fusion** — incorporate wheel-speed and steering-angle signals more directly into the position-estimation pipeline, not just velocity/yaw.
- [ ] **Real-time inference pipeline** — package the model for lightweight, real-time on-device inference.
- [ ] **Uncertainty quantification** — estimate confidence/error bounds on the predicted trajectory, useful for downstream safety-critical decision-making.

### Planned System Extensions

1. **P2P SOS Mesh Network Integration** — enabling automated, blackout-resilient accident and incident alerting in subterranean/underground infrastructure, where no centralized network connectivity is available.
2. **Mainstream Map App API Integration** — using the **OpenStreetMap API** to enable seamless transitions between indoor (GPS-denied, AI-estimated) and outdoor (GPS-available) positioning within a single, familiar map interface.
3. **Cross-Platform Sensor Fusion Support** — extending the positioning pipeline to ingest data from wearables, responder body-cams, and multi-agent industrial IoT devices, broadening the system beyond single-vehicle use cases.
---
## 🌐 Beyond the Problem Statement

While the core deliverable focuses on GPS-denied position estimation, this technology extends into several high-impact real-world applications:

- **Enhanced safety for first responders** operating in complex indoor environments (e.g. multi-story buildings, disaster sites) where GPS is unreliable or entirely unavailable.
- **Optimized logistics and autonomous vehicle operation** in warehouses and tunnels — environments where continuous, GPS-independent positioning is essential for automation.
- **Integrated SOS Alert System** — leveraging our AI-based positioning to pinpoint and relay an accurate location during critical incidents, even when GPS signal is lost or corrupted.
- 

## 👥 Team — Aviral Path

| Name | Role | GitHub |
|---|---|---|
| Aditya Raj | Machine Learning Developer Team Member | [@rajaditya0806](https://github.com/rajaditya0806) |
| Anushka | Full-Stack Developer | `[GitHub link]` |
| Jaanvi Batra | PPT-Specialist | `[GitHub link]` |
| Raju Gupta | Front-End Lead | @rajugupta40110-hue(https://github.com/rajugupta40110-hue) |
| Harsh Garg | `-` | `github` |
| Mann Goswami | `-` | `github` |

---

## 🙏 Acknowledgments

- **IO-VNBD dataset** creators and maintainers, for providing a benchmark dataset specifically built for GPS-denied vehicle navigation research.
- The broader inertial navigation and sensor fusion research community, whose prior work on dead reckoning and drift correction informed our methodology.

---




<p align="center">Built with ⚙️ and 🧠 by <b>Team Aviral Path</b></p>
