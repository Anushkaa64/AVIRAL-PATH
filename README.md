# 🚗 Aviral Path — AI-Based Vehicle Positioning under GPS-Denied Conditions

> **Autonomous Navigation Telemetry & IMU Dead Reckoning System**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Click%20Here-emerald?style=for-the-badge&logo=vercel)](YOUR_DEPLOYED_URL_HERE)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python)]()
[![TensorFlow](https://img.shields.io/badge/TensorFlow-Keras-orange?style=for-the-badge&logo=tensorflow)]()
[![React](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-6.x-purple?style=for-the-badge&logo=vite)](https://vitejs.dev/)

---

## 🌟 Live Interactive Dashboard for Judges

🌐 **Live Hosted Link:**  
👉 **[YOUR_DEPLOYED_URL_HERE](YOUR_DEPLOYED_URL_HERE)** *(Paste your Vercel URL here!)*

*The live dashboard allows judges to visually inspect live dead reckoning trajectory estimations, trigger GNSS blackout / tunnel simulations, load custom IO-VNBD dataset CSV files, and view real-time sensor telemetry.*

---

## 📌 Table of Contents

- [Live Interactive Dashboard](#-live-interactive-dashboard-for-judges)
- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Why This Matters](#-why-this-matters)
- [Dataset — IO-VNBD](#-dataset--io-vnbd)
- [Approach & Methodology](#-approach--methodology)
- [Model Architecture](#-model-architecture)
- [Results](#-results)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Tech Stack](#-tech-stack)
- [Team](#-team)

---

## 🔍 Overview

**Aviral Path** is an AI system designed to solve one of the core challenges in autonomous and connected vehicle navigation: **estimating a vehicle's real-time position when GPS signal is unavailable or unreliable** — for example, in tunnels, dense urban canyons, underground parking structures, or during deliberate GPS jamming/spoofing.

We approach this as an **inertial navigation (dead reckoning)** problem, powered by a deep learning model trained on the **IO-VNBD (Inertial and Odometry Vehicle Navigation Benchmark Dataset)**. Instead of relying on classical physics-only dead reckoning, we use an **LSTM (Long Short-Term Memory) neural network** to learn vehicle dynamics directly from raw sensor data — accelerometer, gyroscope, and wheel-speed signals — and predict the vehicle's velocity, which is then integrated into a full 2D trajectory.

---

## 🎯 Problem Statement

> *"Teams are required to include the preliminary AI models and the results of the position plot inferenced from the subset of IO-VNBD dataset as part of their proposals submitted for evaluation."*

Our objective:
1. Build a preliminary AI model trained on a subset of the IO-VNBD dataset.
2. Use that model's output to reconstruct the vehicle's estimated position over time.
3. Visually and quantitatively compare this AI-estimated path against the GPS ground-truth path.
4. Demonstrate the model generalizes to **unseen driving data**.

---

## 📊 Dataset — IO-VNBD

**IO-VNBD (Inertial and Odometry Vehicle Navigation Benchmark Dataset)** is a real-world driving dataset combining:

- GPS-derived ground truth (latitude, longitude, height, heading, velocity)
- Inertial Measurement Unit (IMU) data (longitudinal/lateral acceleration, yaw rate)
- Vehicle CAN-bus data (wheel speeds, steering angle, gear, throttle, brake)

---

## 📁 Repository Structure

```
intelligent-dead-reckoning/
├── dashboard/               # Interactive React + Vite Web Dashboard
│   ├── frontend/            # React Leaflet & Telemetry Components
│   └── backend/             # Python FastAPI Dead Reckoning Engine
├── IO-VNBD/                 # Dataset benchmarks
├── IO_VNBD_ML_Master.ipynb  # Main ML model training & evaluation notebook
└── README.md
```

---

## 🛠️ Quick Start Guide (Local Development)

### Run the Web Dashboard
```bash
cd dashboard/frontend
npm install
npm run dev
```
Open `http://localhost:5173` to launch the interactive dashboard locally.

---

## 👥 Team — Aviral Path

| Name | Role |
|---|---|
| Aditya Raj | Student at Bharati Vidyapeeth's College of Engineering |
| Anushka | Full-Stack Developer |
| Jaanvi Batra | Presentation Specialist |
| Raju Gupta | Core Contributor |
| Harsh Garg | Core Contributor |
| Mann Goswami | Core Contributor |

---

<p align="center">Built with ⚙️ and 🧠 by <b>Team Aviral Path</b></p>
