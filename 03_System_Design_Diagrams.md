# 03 – System Design & Diagrams

All diagrams use [Mermaid](https://mermaid.js.org/) and render automatically on GitHub.

---

## 1. System Architecture
```mermaid
flowchart LR
    subgraph Sensors
        A1[Camera]
        A2[LiDAR]
        A3[Radar]
        A4[GPS]
        A5[IMU]
        A6[Wheel Speed]
    end

    A1 & A2 & A3 & A4 & A5 & A6 --> B[Data Ingestion Layer]
    B --> C[Pre-processing & Time Sync]
    C --> D[(Historical Perception DB)]
    C --> E[Metric Engine]
    D --> E
    subgraph Metric Engine
        E1[Historical Consistency]
        E2[Temporal Consistency]
        E3[Cross-Sensor Agreement]
        E4[Signal Quality]
    end
    E --> F[Reliability Score Fusion]
    F --> G[Classifier: High / Medium / Low]
    G --> H[Alert Manager]
    G --> I[Dashboard & Reports]
    G -->|verified high reliability| D
```

---

## 2. Data Flow Diagram – Level 0 (Context)
```mermaid
flowchart LR
    S[Vehicle Sensors] -->|raw data| SYS((Reliability Measurement System))
    U[User / Engineer] -->|config, queries| SYS
    SYS -->|scores, alerts, reports| U
    SYS <-->|read / write| DB[(Historical DB)]
```

## 3. Data Flow Diagram – Level 1
```mermaid
flowchart TD
    S[Sensors] -->|raw stream| P1[1.0 Ingest & Validate]
    P1 -->|clean data| P2[2.0 Preprocess & Sync]
    P2 -->|synced frames| P3[3.0 Retrieve Historical Context]
    DB[(Historical DB)] -->|baseline| P3
    P3 -->|frame + baseline| P4[4.0 Compute Metrics]
    P4 -->|metrics| P5[5.0 Fuse Score & Classify]
    P5 -->|score| P6[6.0 Alert & Visualize]
    P6 -->|alerts / charts| U[User]
    P5 -->|verified data| P7[7.0 Update Baseline]
    P7 --> DB
```

---

## 4. Use Case Diagram
```mermaid
flowchart LR
    Driver((Driver))
    Engineer((Engineer))
    Admin((Admin))

    subgraph System
        UC1([View live reliability])
        UC2([Receive alert])
        UC3([Upload / stream data])
        UC4([Generate report])
        UC5([Inject test fault])
        UC6([Configure thresholds])
        UC7([Manage historical DB])
    end

    Driver --> UC1
    Driver --> UC2
    Engineer --> UC1
    Engineer --> UC3
    Engineer --> UC4
    Engineer --> UC5
    Admin --> UC6
    Admin --> UC7
```

---

## 5. Sequence Diagram – Scoring a Sensor Reading
```mermaid
sequenceDiagram
    participant S as Sensor
    participant I as Ingestion
    participant P as Preprocessor
    participant H as Historical DB
    participant M as Metric Engine
    participant F as Score Fusion
    participant A as Alert Manager
    participant D as Dashboard

    S->>I: Raw reading
    I->>P: Validated data
    P->>H: Query context (location, weather, speed)
    H-->>P: Historical baseline
    P->>M: Frame + baseline
    M->>M: Historical / Temporal / Cross-sensor / Quality
    M->>F: Metric values
    F->>F: Weighted fusion → score
    F->>D: Score + level
    alt score < threshold
        F->>A: Trigger alert
        A->>D: Show warning
    end
    F->>H: Store if score high
```

---

## 6. Activity / Flowchart – Main Algorithm
```mermaid
flowchart TD
    Start([Start]) --> R[Read sensor frame]
    R --> V{Valid data?}
    V -- No --> X[Mark sensor as faulty / missing] --> L
    V -- Yes --> PP[Clean, normalize, sync timestamps]
    PP --> Q[Fetch historical baseline for context]
    Q --> M1[Compute historical deviation z-score]
    M1 --> M2[Compute temporal residual]
    M2 --> M3[Compute cross-sensor agreement]
    M3 --> M4[Compute signal quality]
    M4 --> SC[Reliability = weighted sum]
    SC --> T{Score >= 0.8?}
    T -- Yes --> HI[High] --> UPD[Update baseline]
    T -- No --> T2{Score >= 0.5?}
    T2 -- Yes --> MED[Medium]
    T2 -- No --> LOW[Low → Alert]
    UPD --> L[Log & display]
    MED --> L
    LOW --> L
    L --> E{More data?}
    E -- Yes --> R
    E -- No --> End([End])
```

---

## 7. Class Diagram
```mermaid
classDiagram
    class Sensor {
        +String sensorId
        +String type
        +read() Observation
    }
    class Observation {
        +DateTime timestamp
        +Dict values
        +Context context
    }
    class Context {
        +float lat
        +float lon
        +String weather
        +float speed
    }
    class HistoricalDB {
        +store(Observation)
        +getBaseline(Context) Baseline
    }
    class Baseline {
        +float mean
        +float std
        +int sampleCount
    }
    class MetricEngine {
        +historicalConsistency() float
        +temporalConsistency() float
        +crossSensorAgreement() float
        +signalQuality() float
    }
    class ReliabilityScorer {
        +Dict weights
        +computeScore() float
        +classify() String
    }
    class AlertManager {
        +float threshold
        +trigger(Score)
    }
    class Dashboard {
        +showScores()
        +showTrends()
    }

    Sensor --> Observation
    Observation --> Context
    HistoricalDB --> Baseline
    MetricEngine --> HistoricalDB
    MetricEngine --> Observation
    ReliabilityScorer --> MetricEngine
    ReliabilityScorer --> AlertManager
    ReliabilityScorer --> Dashboard
```

---

## 8. ER Diagram (Database)
```mermaid
erDiagram
    SENSOR ||--o{ OBSERVATION : produces
    CONTEXT ||--o{ OBSERVATION : "has"
    CONTEXT ||--o| BASELINE : defines
    OBSERVATION ||--|| RELIABILITY_SCORE : evaluated_as
    RELIABILITY_SCORE ||--o{ ALERT : triggers
    USER ||--o{ ALERT : acknowledges

    SENSOR {
        int sensor_id PK
        string type
        string location_on_vehicle
        string status
    }
    CONTEXT {
        int context_id PK
        float lat
        float lon
        string weather
        float speed_kmph
        string time_of_day
    }
    OBSERVATION {
        int obs_id PK
        int sensor_id FK
        int context_id FK
        datetime timestamp
        json values
    }
    BASELINE {
        int baseline_id PK
        int context_id FK
        float mean
        float std
        int sample_count
    }
    RELIABILITY_SCORE {
        int score_id PK
        int obs_id FK
        float historical
        float temporal
        float cross_sensor
        float quality
        float final_score
        string level
    }
    ALERT {
        int alert_id PK
        int score_id FK
        datetime raised_at
        string severity
        boolean acknowledged
    }
    USER {
        int user_id PK
        string name
        string role
    }
```

---

## 9. Component / Module Layout
```mermaid
flowchart TB
    subgraph src
        ingest[ingestion/]
        pre[preprocessing/]
        hist[history/]
        metrics[metrics/]
        score[scoring/]
        alerts[alerts/]
        ui[dashboard/]
        tests[tests/]
    end
    ingest --> pre --> metrics --> score --> alerts
    hist --> metrics
    score --> ui
```

---

## 10. Reliability Score Formulae
| Metric | Formula |
|--------|---------|
| Historical Consistency (H) | `H = exp(-\|x - μ\| / (k·σ))` where μ, σ come from the historical baseline |
| Temporal Consistency (T) | `T = exp(-r² / σ_r²)` where r is the Kalman/predicted residual |
| Cross-Sensor Agreement (C) | `C = 1 - normalized_difference(sensor_A, sensor_B)` |
| Signal Quality (Q) | `Q = w_snr·SNR_norm + w_d·(1 - dropout_rate)` |
| **Final Score (R)** | `R = w1·H + w2·T + w3·C + w4·Q` with `Σw = 1` (default 0.35, 0.25, 0.25, 0.15) |

| Level | Range |
|-------|-------|
| High | R ≥ 0.80 |
| Medium | 0.50 ≤ R < 0.80 |
| Low | R < 0.50 |
