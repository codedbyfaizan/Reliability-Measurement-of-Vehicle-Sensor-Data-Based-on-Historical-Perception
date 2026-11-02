# 01 – Problem Statement

## Project Title
**Reliability Measurement of Vehicle Sensor Data Based on Historical Perception**

## 1. Background
Autonomous and driver-assistance systems make safety-critical decisions from sensor data. If a sensor gives wrong, noisy, delayed or spoofed data, the vehicle may brake late, drift out of lane or misjudge obstacles. Most current systems only detect *hard failures* (sensor offline); they do not quantify *how trustworthy* a working sensor's data is at a given moment.

## 2. Problem Definition
There is no lightweight, real-time method that assigns a **reliability score** to each sensor reading by comparing it with what the vehicle (or fleet) has **historically perceived** in the same context (location, time of day, weather, speed).

## 3. Motivation
- Sensors degrade gradually (fog, rain, glare, lens dirt, calibration drift).
- GPS spoofing and sensor attacks are rising.
- Sensor fusion works better when each input is weighted by trust.
- Safety standards (ISO 26262, ISO 21448 SOTIF) require awareness of sensing limitations.

## 4. Objectives
1. Collect and store historical perception data from vehicle sensors.
2. Build a baseline model of "normal" sensor behaviour per context.
3. Compute a reliability score (0–1) for each live sensor reading.
4. Detect anomalies and degraded sensors early.
5. Provide a dashboard and alerts to the driver/engineer.
6. Evaluate accuracy against labelled faulty/normal data.

## 5. Scope
**In scope**
- Sensors: camera, LiDAR, radar, GPS, IMU, wheel-speed (extensible).
- Offline replay of datasets and near-real-time streaming simulation.
- Score computation, anomaly detection, visualization, reporting.

**Out of scope**
- Actual vehicle control / actuation.
- Hardware sensor design.
- Full sensor-fusion/perception stack.

## 6. Proposed Solution (Summary)
A pipeline that: (1) ingests sensor streams, (2) pre-processes and time-synchronizes them, (3) retrieves historical perception for the same context, (4) computes consistency metrics (historical, temporal, cross-sensor, signal quality), (5) fuses them into a reliability score, and (6) displays/logs results and raises alerts when the score drops below a threshold.

## 7. Expected Outcomes
- Per-sensor reliability score and health status.
- Early warning of sensor degradation.
- Historical reliability reports and trends.
- Measured detection performance (precision, recall, F1, false-alarm rate).

## 8. Stakeholders
| Stakeholder | Interest |
|-------------|----------|
| Driver / Operator | Know when to trust the system |
| ADAS / AV engineers | Debug and tune sensors |
| Fleet manager | Maintenance planning |
| Safety auditors | Evidence for compliance |

## 9. Constraints & Assumptions
- Datasets are public or simulated; real vehicle access is not required.
- Timestamps are available and roughly synchronized.
- Historical data contains mostly healthy-sensor samples.
- Processing latency target: < 100 ms per frame batch.

## 10. Success Criteria
- Detection F1-score ≥ 0.85 on injected faults.
- False alarm rate ≤ 5%.
- Score computation latency ≤ 100 ms.
