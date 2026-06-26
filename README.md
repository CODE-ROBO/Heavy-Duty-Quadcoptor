<h1 align="center">High-Thrust Heavy-Duty Autonomous UAV Platform 🛸⚡</h1>
<h4 align="center">Heavy-Payload Aerodynamics, High-Current Power Electronics, & PID Flight Dynamics</h4>

<p align="center">
  <br>
  <img src="https://img.shields.io/badge/ArduPilot-00599C?style=for-the-badge&logo=ardupilot&logoColor=white" alt="ArduPilot"/>
  <img src="https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white" alt="C++"/>
  <img src="https://img.shields.io/badge/MATLAB-FFD700?style=for-the-badge&logo=mathworks&logoColor=black" alt="MATLAB"/>
  <img src="https://img.shields.io/badge/SolidWorks-00FFFF?style=for-the-badge&logo=solidworks&logoColor=black" alt="SolidWorks"/>
  <img src="https://img.shields.io/badge/Altium-8B0000?style=for-the-badge&logo=altium&logoColor=white" alt="Altium"/>
</p>

<p align="center">
  <img src="https://via.placeholder.com/800x400/0a0a0a/00FFFF?text=[INSERT+UAV+FLIGHT+GIF+OR+CAD+RENDER+HERE]" alt="Heavy-Duty UAV Render" width="100%"/>
</p>

---

<details open>
  <summary><b>📑 DIRECTORY TERMINAL (TABLE OF CONTENTS)</b></summary>
  <ol>
    <li><a href="#overview">Executive System Overview</a></li>
    <li><a href="#datasheet">System Datasheet & Engineering Targets</a></li>
    <li><a href="#logic">Flight Dynamics & PID Control Logic</a></li>
    <li><a href="#hardware">Hardware Fabrication & Bill of Materials</a></li>
    <li><a href="#pipeline">IMU Filtering & Actuation Pipeline</a></li>
    <li><a href="#architecture">Repository Architecture & CI/CD</a></li>
    <li><a href="#validation">Empirical Validation Matrix (Development Log)</a></li>
    <li><a href="#deployment">Deployment & Reproducibility</a></li>
    <li><a href="#team">R&D Framework & Development Scope</a></li>
    <li><a href="#academic">Academic Trajectory</a></li>
    <li><a href="#citation">Academic Citation</a></li>
  </ol>
</details>

---

### <a id="overview"></a>🌐 EXECUTIVE SYSTEM OVERVIEW

<div align="justify">
This repository documents the structural engineering, power architecture, and flight stability tuning for a custom-built, heavy-duty quadcopter platform. Designed as a high-payload research vehicle, the core engineering challenge was optimizing the structural rigidity and power-to-weight margins to handle severe aerodynamic transients without frame resonance or control loop oscillation.

Rejecting off-the-shelf commercial kits, this vehicle was engineered from the ground up to analyze high-current power distribution, deterministic sensor fusion, and dynamic physical payloads. The resulting platform delivers high-thrust autonomous lift capabilities backed by a highly tuned, low-latency flight control loop.
</div>

---

### <a id="datasheet"></a>📋 SYSTEM DATASHEET & ENGINEERING TARGETS

<div align="justify">
The architecture prioritizes electrical throughput and structural damping to ensure sensor fidelity is maintained during high-amp thrust spikes.
</div>

| Subsystem | Specification | Engineering Objective |
| :--- | :--- | :--- |
| **Control Architecture** | 32-Bit ARM Cortex Flight Controller | Execute deterministic IMU sampling and motor mixing at high frequencies. |
| **Propulsion Array** | High-Torque BLDC Motors | Generate dynamic vertical lift with a Thrust-to-Weight Ratio (TWR) > 2.5. |
| **Power Distribution** | Low-Resistance High-Current Bus | Manage severe amp draws without localized thermal throttling or ESC voltage sag. |
| **Flight Dynamics** | Proportional-Integral-Derivative (PID) | Achieve crisp roll/pitch/yaw tracking with zero payload-induced oscillation. |
| **Sensor Fusion** | 6-Axis IMU (Gyroscope + Accelerometer) | Isolate aerodynamic motion from high-frequency mechanical vibration. |
| **Project Status** | **FLIGHT VALIDATED** | Fully operational, tuned, and cleared for autonomous payload operations. |

---

### <a id="logic"></a>🧠 FLIGHT DYNAMICS & PID CONTROL LOGIC

<div align="justify">
Stable flight relies on calculating the difference between the pilot's requested angle (setpoint) and the aircraft's actual physical angle measured by the onboard gyroscope. This difference represents the error $e(t)$. To correct this error, the flight controller executes a high-speed PID control loop for each individual axis (Roll, Pitch, and Yaw):
</div>

$$u(t) = K_p e(t) + K_i \int e(t)dt + K_d \frac{de(t)}{dt}$$

<div align="justify">
Where $u(t)$ is the corrective motor command. To ensure the vehicle can physically recover from these dynamic errors without entering a stall state, the propulsion system was mathematically modeled to maintain a safe Thrust-to-Weight Ratio ($TWR$):
</div>

$$TWR = \frac{\sum T_{max}}{W \cdot g}$$

<div align="justify">
A $TWR$ exceeding 2.0 ensures that the flight controller has sufficient motor headroom (throttle authority) to execute aggressive PID corrections ($u(t)$) even while carrying maximum payload mass.
</div>

---

### <a id="hardware"></a>⚙️ HARDWARE FABRICATION & BILL OF MATERIALS

<div align="justify">
The physical frame and power electronics were designed to safely isolate high-current switching noise away from delicate logic-level sensors.
</div>

* 🛡️ **Structural Rigidity:** Symmetrical Carbon Fiber (CF) multi-rotor frame designed with custom vibration-damping standoffs to mechanically decouple the IMU from high-frequency motor resonance.
* ⚡ **Power Distribution System (PDS):** Routed a low-latency bus system using heavy-gauge silicone wiring and high-temp soldering to handle burst currents during aggressive vertical maneuvers.
* 🌪️ **Propulsion Dynamics:** Perfectly balanced, high-pitch aerodynamic propellers mated to heavy-duty stators to prevent mid-air desyncs under rapid acceleration.

**Primary Hardware Bill of Materials (BOM):**
| Component | Material / Specification | Subsystem |
| :--- | :--- | :--- |
| **Master Controller** | F7/H7 Class Flight Controller | Logic Engine & Sensor Fusion |
| **Electronic Speed Controllers** | 4-in-1 60A BLHeli_32 ESC | 3-Phase BLDC Commutation |
| **Actuators** | Heavy-Duty Brushless DC (BLDC) Motors | Physical Thrust Generation |
| **Power Source** | 6S High-Discharge LiPo Battery | Energy Storage & Output |
| **Chassis** | High-Tensile Carbon Fiber (Quad X) | Structural Containment |

---

### <a id="pipeline"></a>📡 IMU FILTERING & ACTUATION PIPELINE

<div align="justify">
The embedded firmware executes a continuous loop that must sample, filter, and output motor signals faster than the physical aerodynamics can destabilize the chassis.
</div>

1. **Hardware Ingestion:** The 6-axis IMU samples raw angular velocity and acceleration at an 8kHz refresh rate.
2. **DSP Filtering:** Raw gyroscope data is passed through dynamic Notch Filters and Low-Pass Filters (LPF) to mathematically erase motor vibration noise before it enters the control loop.
3. **PID Evaluation:** The filtered data is compared against the target setpoint, and the $K_p$, $K_i$, and $K_d$ multipliers calculate the required rotational correction.
4. **Motor Mixing & Protocol:** The PID output is translated by a custom motor mixer matrix and sent to the ESCs via DSHOT600 digital protocol, updating the RPM of all four motors simultaneously.

---

### <a id="architecture"></a>🗄️ REPOSITORY ARCHITECTURE & CI/CD

<div align="justify">
<i>Structured for absolute transparency, bridging high-current hardware layouts with highly tuned firmware profiles.</i>
</div>

```text
📁 High-Thrust-UAV-Platform/
│
├── 📁 .github/workflows/     # CI/CD: Automated linting for custom Python telemetry scripts
├── 📁 firmware/              # Operational Flight Code & Parameters
│   ├── flight_controller_dump.txt  # Core configuration, port mapping, and PID profiles
│   └── custom_mixer.c              # Custom motor geometry matrix
│
├── 📁 hardware/              # Physical Build Assets
│   ├── power_distribution.pdf      # High-amp wiring layout and ground loops
│   ├── component_mass_budget.xlsx  # TWR calculations and payload limits
│   └── CAD_models/                 # Custom payload/battery mounts (STEP/STL)
│
├── 📁 docs/                  # System documentation
│   ├── blackbox_tuning_logs/       # Gyro spectral density graphs and step-response data
│   └── SOP_preflight.pdf           # Strict pre-flight validation checklists
│
└── README.md                 # Main system dossier
