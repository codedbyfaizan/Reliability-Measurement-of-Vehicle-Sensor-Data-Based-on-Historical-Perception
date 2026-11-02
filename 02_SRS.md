# 02 – Software Requirements Specification (SRS)

**Project:** Reliability Measurement of Vehicle Sensor Data Based on Historical Perception
**Version:** 1.0 **Standard:** IEEE 830 (adapted)

---

## 1. Introduction
### 1.1 Purpose
This document specifies the functional and non-functional requirements of a system that evaluates the reliability of vehicle sensor data using historical perception.

### 1.2 Scope
The system ingests multi-sensor data, compares it with historical baselines, computes reliability scores, detects anomalies, and presents results via a dashboard and reports.

### 1.3 Definitions
| Term | Meaning |
|------|---------|
| Historical Perception | Stored past sensor observations under a given context |
| Reliability Score | Value 0–1 indicating trust in a sensor reading |
| Context | Location, time of day, weather, speed |
| ADAS | Advanced Driver Assistance Systems |
| SRS | Software Requirements Specification |

### 1.4 References
IEEE 830-1998, ISO 26262, ISO 21448 (SOTIF), KITTI / nuScenes dataset docs.

---

## 2. Overall Description
### 2.1 Product Perspective
A standalone analytic module that can sit beside a vehicle's perception stack or run offline on recorded data.

### 2.2 Product Functions
1. Sensor data ingestion
2. Pre-processing & synchronization
3. Historical perception storage and retrieval
4. Reliability metric computation
5. Score fusion & classification
6. Anomaly detection & alerting
7. Visualization & reporting

### 2.3 User Classes
Driver/Operator, Engineer/Analyst, Administrator.

### 2.4 Operating Environment
Windows/Linux, Python 3.10+, 8 GB RAM minimum, modern browser for dashboard.

### 2.5 Assumptions & Dependencies
Timestamped sensor data; availability of datasets; historical DB populated before scoring.

---

## 3. Functional Requirements
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1 | System shall ingest data from camera, LiDAR, radar, GPS, IMU, wheel-speed in CSV/ROS-bag/JSON format | High |
| FR-2 | System shall remove noise, handle missing values and time-synchronize sensor streams | High |
| FR-3 | System shall store processed observations with context in a historical database | High |
| FR-4 | System shall retrieve the most similar historical context for a live reading | High |
| FR-5 | System shall compute Historical Consistency (deviation from baseline) | High |
| FR-6 | System shall compute Temporal Consistency (change vs previous frames, Kalman residual) | High |
| FR-7 | System shall compute Cross-Sensor Agreement (e.g., GPS vs IMU vs wheel odometry) | High |
| FR-8 | System shall compute Signal Quality (SNR, dropout rate, point density) | Medium |
| FR-9 | System shall fuse metrics into a reliability score between 0 and 1 using configurable weights | High |
| FR-10 | System shall classify each sensor as High (≥0.8), Medium (0.5–0.8) or Low (<0.5) reliability | High |
| FR-11 | System shall raise an alert when a score falls below a configurable threshold | High |
| FR-12 | System shall display live scores, trends and sensor status on a dashboard | High |
| FR-13 | System shall allow fault injection (noise, bias, dropout, spoofing) for testing | Medium |
| FR-14 | System shall export reports (CSV/PDF) | Medium |
| FR-15 | System shall allow users to configure thresholds and weights | Medium |
| FR-16 | System shall update the historical baseline with verified high-reliability data | Low |
| FR-17 | System shall provide user login with role-based access | Low |

---

## 4. Non-Functional Requirements
| ID | Category | Requirement |
|----|----------|-------------|
| NFR-1 | Performance | Score computation ≤ 100 ms per batch |
| NFR-2 | Accuracy | Anomaly detection F1 ≥ 0.85; false alarm ≤ 5% |
| NFR-3 | Scalability | Handle ≥ 10 sensors and 1M+ historical records |
| NFR-4 | Reliability | System availability ≥ 99% during runs; graceful handling of missing sensors |
| NFR-5 | Usability | Dashboard usable with ≤ 15 min of training |
| NFR-6 | Security | Role-based access; data stored without personal identifiers |
| NFR-7 | Maintainability | Modular code, ≥ 70% unit test coverage, documented APIs |
| NFR-8 | Portability | Runs on Windows, Linux, macOS |
| NFR-9 | Extensibility | New sensor types added via plug-in interface |

---

## 5. External Interface Requirements
### 5.1 User Interface
Web dashboard (see `04_UI_Design.md`).

### 5.2 Software Interfaces
- Dataset readers: KITTI, nuScenes, CSV, ROS bag
- Database: SQLite/PostgreSQL
- Libraries: NumPy, Pandas, SciPy, scikit-learn

### 5.3 Hardware Interfaces
None mandatory (optional live CAN/sensor feed).

### 5.4 Data Format (sample)
```json
{
  "timestamp": "2026-11-02T10:15:30.120Z",
  "sensor_id": "LIDAR_01",
  "type": "lidar",
  "context": {"lat": 17.385, "lon": 78.486, "weather": "clear", "speed_kmph": 42},
  "values": {"points": 54210, "mean_range_m": 18.4}
}
```

---

## 6. Use Cases (Summary)
| ID | Use Case | Actor |
|----|----------|-------|
| UC-1 | Upload / stream sensor data | Engineer |
| UC-2 | View real-time reliability | Driver, Engineer |
| UC-3 | Configure thresholds/weights | Admin |
| UC-4 | Receive alert | Driver |
| UC-5 | Generate report | Engineer |
| UC-6 | Inject fault for testing | Engineer |
| UC-7 | Manage historical database | Admin |

---

## 7. Data Requirements
Entities: Sensor, Observation, Context, HistoricalBaseline, ReliabilityScore, Alert, User (see ER diagram in `03_System_Design_Diagrams.md`).

## 8. Constraints
Python-based implementation, open datasets only, academic timeline.

## 9. Acceptance Criteria
- All High-priority FRs implemented and demonstrated.
- NFR-1 and NFR-2 verified by test results.
- Final report and documentation delivered.

## 10. Future Enhancements
Deep-learning-based reliability estimation, V2X fleet-shared history, edge-device deployment, ROS 2 integration.
