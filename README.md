# ❤️ SAX Edge Monitor
### False-Alarm-Resistant Edge Patient Monitor Using Symbolic Aggregate Approximation (SAX) on PPG Inter-Beat Intervals

<p align="center">
  <b>Embedded Systems • Biomedical Signal Processing • FreeRTOS • Time-Series Analysis</b>
</p>

<p align="center">
  <i>From pulse signals to symbolic rhythm intelligence.</i>
</p>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Proposed Solution](#-proposed-solution)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Hardware Components](#-hardware-components)
- [PPG Signal Acquisition](#-ppg-signal-acquisition)
- [Inter-Beat Interval Calculation](#-inter-beat-interval-ibi-calculation)
- [Beat Validation and Artifact Rejection](#-beat-validation-and-artifact-rejection)
- [Symbolic Aggregate approXimation](#-symbolic-aggregate-approximation-sax)
- [Risk Classification](#-risk-classification)
- [Real-Time Embedded Implementation](#-real-time-embedded-implementation)
- [Challenges and Engineering Solutions](#-challenges-and-engineering-solutions)
- [Results and Demonstration](#-results-and-demonstration)
- [Comparison with Existing Approaches](#-comparison-with-existing-approaches)
- [Technology Stack](#-technology-stack)
- [Future Scope](#-future-scope)
- [Repository Structure](#-repository-structure)
- [Team](#-team)

---

## 🌐 Overview

**SAX Edge Monitor** is a low-cost embedded patient-monitoring prototype designed to assess the reliability and rhythm regularity of Photoplethysmography (PPG) measurements using Symbolic Aggregate approXimation (SAX).

The project addresses a common challenge in continuous patient monitoring: frequent false or non-actionable alarms caused by motion artifacts, sensor displacement, and unreliable signal measurements.

Repeated false alarms can contribute to alarm fatigue, potentially reducing the attention given to genuine alerts.

The proposed system combines PPG signal processing, inter-beat interval (IBI) analysis, beat validation, statistical variation analysis, and symbolic time-series representation to provide a deterministic edge-based monitoring approach.

The prototype is implemented on an **STM32F429I-DISC1 Discovery Board** using **FreeRTOS** and a **MAX30102 optical sensor**.

Unlike approaches that depend on cloud connectivity or trained machine-learning models, the current implementation performs its signal processing and monitoring operations locally on the embedded platform.

### Core Objective

To explore a deterministic, resource-conscious embedded monitoring system that evaluates beat reliability and rhythm variation before presenting a monitoring status.

---

## ⚠️ Problem Statement

Continuous bedside monitoring systems can generate a large number of alarms that may not always represent clinically meaningful events.

Common sources of unreliable readings include:

- Motion artifacts during patient movement.
- Optical sensor displacement.
- Incorrect beat detection.
- Noise and fluctuations in the acquired PPG waveform.
- Irregularities introduced by signal acquisition and processing delays.

A system that reacts to every detected waveform peak without validating its reliability may generate misleading heart-rate estimates and unnecessary alarms.

### Why This Matters

Repeated non-actionable alerts can lead to alarm fatigue among healthcare professionals.

Therefore, there is a need to explore monitoring approaches that assess the quality and consistency of acquired measurements before generating status indications.

---

## 💡 Proposed Solution

SAX Edge Monitor introduces a multi-stage PPG processing pipeline that validates detected beats and analyzes the variation of successive Inter-Beat Intervals.

The system follows these major stages:

1. **Signal Acquisition:** Collects optical PPG readings using the MAX30102 sensor.
2. **Signal Preprocessing:** Removes the DC component and prepares the waveform for peak detection.
3. **Peak Detection:** Uses adaptive thresholding and interpolated peak timing.
4. **Beat Validation:** Applies plausibility checks, confirmation rules, and beat-to-beat jump validation.
5. **IBI Calculation:** Calculates the time interval between accepted successive beats.
6. **Window-Based Analysis:** Groups accepted IBIs into short windows.
7. **SAX Encoding:** Converts normalized IBI sequences into symbolic representations.
8. **Status Generation:** Uses configured statistical thresholds to classify the current monitoring condition as SAFE or WARNING.
9. **Visualization:** Displays waveform and monitoring metrics on the embedded touchscreen.

The system is designed to improve the reliability of the measurements used for monitoring rather than treating every detected pulse peak as a valid beat.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| Real-Time PPG Acquisition | Acquires optical pulse data using MAX30102 |
| Adaptive Peak Detection | Detects candidate beats using adaptive thresholding |
| Interpolated Peak Timing | Improves beat timestamp resolution |
| Beat Validation | Rejects implausible or unconfirmed beat candidates |
| IBI Analysis | Calculates successive inter-beat intervals |
| SAX Encoding | Converts normalized time-series windows into symbolic strings |
| Statistical Monitoring | Uses standard deviation to assess interval variation |
| Edge Processing | Performs processing locally on STM32 |
| FreeRTOS Integration | Uses an RTOS-based embedded software environment |
| Touchscreen Dashboard | Displays waveform, BPM, SAX output and monitoring status |

---

## 🏗️ System Architecture

The system follows a sequential signal acquisition, processing, analysis, and visualization architecture.

### Processing Pipeline

| Stage | Module | Function |
|---|---|---|
| 01 | MAX30102 Sensor | Acquires optical PPG readings |
| 02 | STM32F429 | Reads and processes sensor data |
| 03 | Signal Preprocessing | Removes DC component and prepares waveform |
| 04 | Peak Detection | Identifies candidate pulse peaks |
| 05 | IBI Calculation | Calculates intervals between accepted beats |
| 06 | Windowing and Normalization | Groups and normalizes IBI values |
| 07 | SAX Encoding | Generates symbolic rhythm representation |
| 08 | Decision Logic | Determines SAFE/WARNING status |
| 09 | Touchscreen Interface | Displays monitoring information |

### Overall Data Flow

```text
       MAX30102 Optical Sensor
                 |
                 v
          PPG Acquisition
                 |
                 v
        DC Component Removal
                 |
                 v
       Adaptive Peak Detection
                 |
                 v
        Beat Validation Logic
                 |
                 v
         IBI Calculation
                 |
                 v
       Window-Based Processing
                 |
                 v
         Z-Score Normalization
                 |
                 v
           SAX Encoding
                 |
                 v
       Statistical Evaluation
                 |
                 v
        SAFE / WARNING Status
                 |
                 v
        Touchscreen Dashboard
```

---

## ❤️ PPG Signal Acquisition

The MAX30102 is an optical sensor used to acquire photoplethysmography signals.

PPG measures variations in blood volume through changes in light absorption and reflection at the sensor.

The prototype uses the MAX30102 to acquire optical readings at a configured sampling frequency of **100 Hz**.

### Signal Processing

The acquired PPG signal contains both a slowly varying DC component and a pulsatile AC component.

The firmware removes the DC component to emphasize the pulsatile waveform used for beat detection.

### Adaptive Thresholding

An adaptive threshold is used to identify candidate pulse peaks.

This approach helps the peak detector respond to changing signal amplitudes rather than depending entirely on one fixed threshold.

### Interpolated Peak Timing

Peak timestamps are refined through interpolation to estimate beat timing between adjacent samples.

This is important because IBI calculations depend directly on accurate beat timestamps.

---

## ⏱️ Inter-Beat Interval (IBI) Calculation

An Inter-Beat Interval represents the time difference between two successive accepted beats.

If two consecutive beat timestamps are \(t_i\) and \(t_{i+1}\), then:

\[
IBI_i = t_{i+1}-t_i
\]

When timestamps are expressed in milliseconds, the corresponding BPM estimate can be calculated as:

\[
BPM = \frac{60000}{IBI}
\]

IBI values provide a time-series representation of the intervals between successive detected beats.

Instead of analysing only an individual BPM value, the system examines groups of successive IBIs to understand their variation over time.

---

## 🛡️ Beat Validation and Artifact Rejection

A central component of SAX Edge Monitor is its beat-validation mechanism.

A detected waveform peak is initially treated as a candidate beat. It is not automatically accepted.

The prototype incorporates multiple validation mechanisms.

### 1. Plausibility Range Gate

Candidate intervals are checked against a configured acceptable range.

Intervals outside the permitted range are rejected.

### 2. Two-Beat Confirmation

A new rhythm is not accepted solely because of one isolated candidate interval.

Two agreeing intervals are used to establish a more reliable rhythm.

### 3. Beat-to-Beat Jump Check

Sudden changes between successive intervals are evaluated to identify potentially unreliable beat detections.

### 4. Adaptive Refractory Lockout

A refractory period prevents the same pulse waveform from producing multiple detections.

The lockout is adapted using the previous accepted interval to help reject secondary waveform features such as the dicrotic bump.

### Objective

These checks aim to reduce false beat detections caused by noise, motion artifacts, and waveform irregularities before the intervals are passed to the analysis stage.

---

## 🔤 Symbolic Aggregate approXimation (SAX)

Symbolic Aggregate approXimation is a method for converting numerical time-series data into a compact symbolic representation.

In this project, SAX is applied to normalized windows of accepted Inter-Beat Intervals.

Instead of representing rhythm variation only through numerical values, the system generates a symbolic string that captures the relative pattern of the interval sequence.

### Step 1: Windowing

Accepted IBI values are collected into small groups.

The prototype uses short windows of successive intervals for analysis.

### Step 2: Z-Score Normalization

The IBI values are normalized to remove their mean and scale while preserving their relative variation pattern.

For an observation \(x\):

\[
z = \frac{x-\mu}{\sigma}
\]

Where:

- \(x\) = original IBI value
- \(\mu\) = mean of the window
- \(\sigma\) = standard deviation of the window
- \(z\) = normalized value

### Step 3: Symbolic Mapping

The normalized values are mapped to discrete symbolic regions using a Gaussian-based alphabet.

Each region corresponds to a character.

The resulting sequence forms the SAX string.

### Step 4: Pattern Representation

The symbolic sequence provides a compact representation of the relative shape of the IBI window.

It allows rhythm patterns to be represented using a small number of characters instead of retaining the entire numerical sequence.

### Example

A symbolic sequence such as:

```text
adbe
```

represents the relative pattern of a normalized interval window.

The SAX string is displayed alongside the numerical monitoring metrics.

**Important:** In the current implementation, SAX is used as a pattern representation. It does not directly determine the SAFE/WARNING status.

---

## 📊 Risk Classification

The current prototype uses statistical variation analysis to determine the monitoring status.

### Standard Deviation

The standard deviation measures how much the accepted IBI values vary within a window.

\[
\sigma =
\sqrt{\frac{1}{N}\sum_{i=1}^{N}(x_i-\mu)^2}
\]

Where:

- \(N\) = number of intervals in the window
- \(x_i\) = individual IBI value
- \(\mu\) = mean IBI
- \(\sigma\) = standard deviation

### Status Generation

The calculated standard deviation is compared against configured thresholds.

| Status | Interpretation |
|---|---|
| SAFE | Variation is within the configured range |
| WARNING | Variation exceeds the configured threshold |

The current decision mechanism is based on statistical variation and configured thresholds.

SAX provides additional symbolic information about the rhythm pattern, while the integration of SAX-derived features into the final decision logic remains a future development direction.

**This monitoring classification is an experimental engineering output, not a clinical diagnosis or validated medical alarm.**

---

## ⚙️ Real-Time Embedded Implementation

The system is implemented on the STM32F429I-DISC1 Discovery Board.

### Microcontroller Platform

The STM32F429 uses an ARM Cortex-M4 processor and provides the embedded processing resources required for sensor acquisition, signal processing, and display operations.

### FreeRTOS

FreeRTOS is used as the real-time operating system environment for the embedded implementation.

It provides the framework for organizing application operations into real-time tasks.

### Display Interface

The Discovery Board's integrated touchscreen is used to present the monitoring results.

The displayed metrics include:

- Live PPG waveform.
- Estimated BPM.
- Current SAX string.
- Standard deviation.
- Rejected beat count.
- SAFE/WARNING status.

### Local Processing

The current prototype performs its monitoring computations locally on the embedded board without requiring cloud connectivity or a trained machine-learning model.

---

## 🧰 Challenges and Engineering Solutions

The development process involved several important signal-processing and embedded-system debugging challenges.

| Challenge | Problem Observed | Engineering Solution |
|---|---|---|
| Peak Detection Lockup | Noise ripple generated hundreds of false peaks | Introduced hysteresis and refractory lockout |
| Dicrotic Bump Detection | Secondary pulse-wave features were detected as separate beats | Adapted the lockout period using the previous interval |
| FIFO Configuration | Incorrect MAX30102 FIFO register address | Corrected the register configuration |
| Sampling Frequency Mismatch | Sensor operated at 25 samples/s instead of the assumed 100 | Configured 400 samples/s with 4× on-chip averaging |
| Incorrect Data Reads | Six-byte samples were read in three-byte portions | Implemented pointer-checked burst reads |
| Timing Contamination | Display and serial operations introduced software-loop timing jitter | Used sample-count-based timestamps with interpolation |
| Incorrect Rhythm Resynchronization | Outlier-based resynchronization locked onto bad data | Introduced two-interval seed confirmation and plausibility checks |
| Display Integration | Text overflow and flickering occurred during updates | Addressed display formatting and redraw behavior |
| Embedded Toolchain | Float printing was disabled by default | Enabled floating-point formatting support |

These debugging improvements were important in making the IBI measurements more reliable and reducing false beat detections.

---

## 🖥️ Results and Demonstration

The prototype was demonstrated using the STM32F429I-DISC1 Discovery Board and MAX30102 sensor.

### Hardware Demonstration

The hardware setup includes:

- STM32F429I-DISC1 Discovery Board.
- MAX30102 optical sensor.
- Sensor wiring and interfacing.
- Integrated touchscreen display.

### Monitoring Dashboard

The touchscreen presents the acquired signal and calculated monitoring metrics.

### Serial Monitor Output

The serial output displays successive beat intervals, BPM estimates, SAX strings, standard deviation, status, and rejected beat counts.

### Observed Example

An example demonstration screen in the project report shows:

| Metric | Recorded Example |
|---|---:|
| BPM | 80.1 |
| Standard Deviation | 23.1 ms |
| Rejected Beats | 10 |
| Status | SAFE |

These are illustrative readings from the recorded prototype demonstration.

---

## 📚 Comparison with Existing Approaches

The project report compares the proposed SAX Edge Monitor with existing PPG quality methods, embedded SAX systems, commercial wearables, and ICU monitors.

| Feature | Proposed SAX Edge Monitor |
|---|---|
| Primary Purpose | Beat validation and symbolic rhythm analysis |
| Input Signal | PPG using MAX30102 |
| Machine Learning | Not required |
| Beat Validation | Refractory lockout and IBI checks |
| Symbolic Analysis | Gaussian-based SAX |
| Connectivity | Standalone edge processing |

The distinguishing direction is the combination of beat validation, IBI variation analysis, and symbolic rhythm representation on a standalone embedded platform.

---

## 🛠️ Technology Stack

| Category | Technology |
|---|---|
| Microcontroller | STM32F429 |
| Development Board | STM32F429I-DISC1 |
| Optical Sensor | MAX30102 |
| Programming Language | Embedded C |
| IDE | STM32CubeIDE |
| Real-Time Operating System | FreeRTOS |
| Sensor Communication | I²C |
| Signal Processing | Digital PPG processing |
| Time-Series Analysis | Symbolic Aggregate approXimation |
| Normalization | Z-score normalization |
| Display | Integrated STM32 Discovery touchscreen |
| Toolchain | ARM GNU GCC |

---

## 🚀 Future Scope

The project provides a foundation for further development towards a personalized wearable monitoring system.

Potential improvements include:

### 1. Personalized Baseline Learning

Collecting individual baseline readings and comparing new measurements against a patient's usual range of variation.

### 2. SAX-Based Decision Integration

Incorporating symbolic rhythm features such as letter distribution, pattern transitions, and direction reversals into the final decision logic.

### 3. Improved Signal Quality Assessment

Further strengthening the detection of motion artifacts, sensor displacement, and unreliable measurements.

### 4. Extended Recording Validation

Testing the system using longer recordings and different signal conditions.

### 5. Wearable Ring Integration

Developing a compact ring-based implementation integrating the optical sensor, embedded processing, and power supply.

### 6. Personalized Monitoring

Exploring patient-specific thresholds and historical variation patterns to support more individualized supplementary monitoring.

---

## 📁 Repository Structure

```text
SAX-Edge-Monitor/
│
├── README.md
│
├── Core/
│   ├── Inc/
│   └── Src/
│
├── Drivers/
│
├── Middlewares/
│
├── Documentation/
│   ├── Mini-Project-Report.pdf
│   └── Architecture/
│
├── Assets/
│   ├── hardware-setup.jpg
│   ├── monitoring-dashboard.jpg
│   └── serial-output.png
│
├── .project
├── .cproject
└── .ioc
```

*The STM32 project directory should retain its actual CubeIDE-generated structure. The structure above is an organizational guide; include only files and directories that exist in the project.*

---

## 📌 Project Status

**Status: Working Embedded Prototype Demonstrated**

The current implementation demonstrates PPG acquisition, beat detection and validation, IBI calculation, SAX encoding, statistical variation analysis, and SAFE/WARNING status visualization on an STM32F429I-DISC1 platform.

Personalized baseline learning, SAX-based decision integration, and wearable ring implementation remain future development directions.

The system is an academic engineering prototype intended for research and experimentation. It is not a certified medical device and must not be used for clinical diagnosis or treatment decisions.

---

## 👥 Team

**Project Team**

- Dhakshayini Usha R
- Mishree Kalaria

**Institution:** Symbiosis Institute of Technology, Pune

**Department:** Electronics and Telecommunication Engineering

**Course:** Microcontroller and Embedded C Programming (MECp)

---

<p align="center">
  <b>SAX EDGE MONITOR</b>
  <br/>
  <i>Reliable Beats. Symbolic Patterns. Intelligent Edge Monitoring.</i>
</p>
