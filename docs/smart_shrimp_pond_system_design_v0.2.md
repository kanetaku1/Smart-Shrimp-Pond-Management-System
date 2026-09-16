# Smart Shrimp Pond Management System

## システム設計書・提案書

## 0. Document Purpose

本設計書は、エビ養殖において、IoT・環境データ・養殖管理データ・機械学習・AI Agent・制御システムを統合し、養殖池環境の継続的なモニタリング、異常・リスクの早期検知、養殖者への通知・意思決定支援、将来的な自動制御を実現するシステムの構想・要求・アーキテクチャ・開発計画を定義する。

最終目的は単なるIoTモニタリングではなく、

> **養殖池の状態を継続的に観測し、池内環境・外部環境・餌・養殖管理情報を統合して、エビにとって望ましくない状態を事前に予測し、養殖者が早期に適切なアクションを取れる状態を作ること**

である。

---

# 1. Executive Summary

## 1.1 提案システム

**Smart Shrimp Pond Management System**

養殖池に設置したIoTセンサーから水質・水環境データを取得し、天候・大気環境などの外部環境データ、および餌・給餌などの養殖管理データと統合する。

蓄積データを機械学習に利用し、異常検知・将来予測・リスク推定を行う。初期段階では推奨閾値による通知を実装し、学習データが蓄積されるにつれて、現在の異常だけでなく将来の環境悪化を予測した「Predictive Alert」へ発展させる。

AI Agentは、センサーデータ、履歴、外部環境、餌、ML予測などを統合して状況を解釈し、養殖者への推奨アクションを生成する。

将来的には、Agentの判断をSafety LayerとPID Controllerを介してポンプ等のアクチュエータへ接続し、閉ループ型の養殖環境制御を目指す。

## 1.2 Core Concept

> **From Monitoring to Predictive Aquaculture**

日本語では、

> **「状態を見る養殖」から「未来を予測して動く養殖」へ**

## 1.3 基本ループ

```text
Observe
  ↓
Collect
  ↓
Integrate
  ↓
Understand
  ↓
Predict
  ↓
Notify / Recommend
  ↓
Act
  ↓
Observe
  ↺
```

---

# 2. Background / Domain Context

## 2.1 エビ養殖の環境管理

調査対象：

- 養殖池の構造
- 水質管理
- 水交換
- 給餌
- エビの成長
- 生存率
- 疾病・ストレス
- 水温
- pH
- DO
- TDS / 塩分
- 濁度
- 水深
- 天候
- 大気環境

## 2.2 FAO等の資料

FAOの「Shrimp Farming: Pond Design, Operation and Management」等をドメイン知識の基礎資料として利用する。

https://www.fao.org/4/AC210E/AC210E00.htm

確認項目：

- 池の設計
- 水管理
- 水質管理
- 給餌
- 池の運用
- 生産管理

## 2.3 既存研究

調査対象：

- IoTによる養殖環境モニタリング
- IoT + MLによる異常検知
- 水質予測
- エビ成長予測
- 給餌最適化
- 自動養殖
- PID制御
- AI Agentを利用した農業・養殖
- Predictive Aquaculture

---

# 3. Problem Definition

## 3.1 現状の想定

```text
                    養殖池
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        水質          天候        餌・給餌
          │            │            │
          └────────────┼────────────┘
                       ▼
                  養殖者の判断
                       │
                       ▼
                  ポンプ等操作
```

## 3.2 課題

### Monitoring

- 継続的な監視が必要
- 複数のセンサー値を同時に把握する必要がある
- センサー値だけでは状況の意味を判断しにくい

### Data

- 水質データだけでは将来予測が難しい可能性がある
- 天候などの外部環境を考慮する必要がある
- 餌の種類・成分・給餌量なども考慮する必要がある
- 給餌・水交換・ポンプ操作等の管理履歴も重要
- 過去データを継続的に蓄積する必要がある

### Decision

- 閾値を超えてから対応する事後対応になりやすい
- 複数変数の関係を人間だけで把握するのは困難
- 経験・知識への依存

### Operation

- ポンプ等の操作を人間が行う必要がある
- 常時監視が必要
- 異常時の対応負担が大きい

---

# 4. Goal / Objective

## 4.1 Primary Goal

> **エビにとって望ましい養殖環境を安定的に維持し、高品質なエビを大量・安定的に生産する。**

## 4.2 Secondary Goal

> **養殖者が行う監視・データ分析・判断・操作の負担を減らす。**

## 4.3 System Objectives

1. 養殖池環境をリアルタイムに把握する
2. 水質・水環境データを長期蓄積する
3. 天候・大気環境を取得する
4. 餌・給餌情報を記録する
5. 養殖管理情報を記録する
6. 複数データソースを統合する
7. 推奨閾値で異常を検知する
8. 異常時に管理者へ通知する
9. MLで将来の環境リスクを予測する
10. 環境悪化前に通知できる状態を目指す
11. AI Agentで状況を統合し推奨アクションを提示する
12. 将来的にポンプ等の自動制御へ接続する

---

# 5. Key Concept: Threshold Alert → Predictive Alert

## 5.1 Stage 1: Threshold-based Alert

```text
Sensor
 ↓
Current Value
 ↓
Recommended Threshold
 ↓
Threshold Exceeded
 ↓
Notification
```

例：

```text
DO
 ↓
Recommended Range
 ↓
Below Threshold
 ↓
Low DO Alert
```

MLが十分に学習されていない初期段階でも実装可能。

## 5.2 Stage 2: Predictive Alert

```text
Current Data
+
Historical Data
+
Weather
+
Air Environment
+
Feed
+
Operation
 ↓
ML
 ↓
Future Risk
 ↓
Prediction
 ↓
Early Notification
```

例：

```text
Current DO: Normal

Temperature ↑
Turbidity ↑
DO ↓ trend
Weather condition changed

        ↓

ML:
High risk of undesirable DO condition
within the next 3 hours.

        ↓

Early Notification
        ↓
Recommended Action
```

## 5.3 Target State

> **「異常が発生したから知らせる」のではなく、「異常が発生する可能性が高まった段階で知らせる」**

---

# 6. Data Architecture

本システムでは、単一のセンサーデータではなく、複数カテゴリのデータを統合する。

```text
                     Data Platform
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       ▼                  ▼                  ▼
 Pond Environment    External Environment  Farm Operation
       │                  │                  │
       ▼                  ▼                  ▼
   IoT Sensors          Weather             Feed
                        Air Quality          Feeding
                                             Stocking
                                             Pump
                                             Water Exchange
                                             Harvest
```

## 6.1 Data Categories

### A. Pond IoT Data

- pH
- Temperature
- TDS
- Turbidity
- Water Level
- DO

### B. External Environment

- Air Temperature
- Humidity
- Rainfall
- Wind
- Atmospheric Pressure
- Solar Radiation
- Weather Forecast
- PM2.5
- PM10
- NO2
- SO2
- CO
- O3
- AQI

### C. Farm Operation

- Feed type
- Feed composition
- Feed amount
- Feeding time
- Feeding frequency
- Stocking date
- Stocking density
- Shrimp species
- Estimated biomass
- Water exchange
- Pump operation
- Fertilizer / nutrient input
- Treatment
- Maintenance
- Harvest

---

# 7. IoT Sensor Architecture

## 7.1 Required Sensors

1. pH Sensor
2. Temperature Sensor
3. TDS Sensor
4. Turbidity Sensor
5. Ultrasonic Sensor
6. DO Sensor

## 7.2 Sensor Purpose

| Parameter   | Sensor             | Purpose            |
| ----------- | ------------------ | ------------------ |
| pH          | pH Sensor          | 水質・酸塩基状態   |
| Temperature | Temperature Sensor | 水温・環境状態     |
| TDS         | TDS Sensor         | 水中溶解物質の指標 |
| Turbidity   | Turbidity Sensor   | 水の濁り・状態変化 |
| Water Level | Ultrasonic Sensor  | 水深・水量変化     |
| DO          | DO Sensor          | 溶存酸素・酸素環境 |

## 7.3 DO

DO（Dissolved Oxygen）を重要な水質パラメータとして扱う。

```text
Temperature
+
pH
+
TDS
+
Turbidity
+
Water Level
+
DO
```

を統合することで、単一指標では捉えにくい環境変化を分析できる可能性がある。

## 7.4 将来追加候補

- Salinity
- ORP
- Ammonia
- Nitrite
- Camera
- Weather Station
- Feed Monitoring
- Biomass Estimation

---

# 8. IoT Edge Architecture

```text
Sensors
   ↓
ESP32 / Edge Device
   ↓
Filtering
   ↓
Validation
   ↓
Timestamp
   ↓
MQTT / Communication
   ↓
Backend
```

Edgeの役割：

- Sensor reading
- Local filtering
- Basic validation
- Timestamp
- Communication
- Offline buffering
- Device health monitoring

通信方式候補：

- MQTT
- Wi-Fi
- LoRa / LoRaWAN
- Ethernet

---

# 9. Data Platform

## 9.1 Data Flow

```text
IoT
 ↓
MQTT
 ↓
Ingestion Service
 ↓
Validation
 ↓
Database
 ↓
API
 ↓
Frontend / ML / Agent
```

## 9.2 Database Entities

```text
Farm
 └── Pond
      ├── Sensor
      ├── Pump
      ├── Telemetry
      ├── EnvironmentalData
      ├── FeedRecord
      ├── OperationRecord
      ├── Alert
      ├── Prediction
      ├── ControlAction
      └── ProductionRecord
```

---

# 10. Telemetry Data

例：

```json
{
  "pond_id": "POND-001",
  "device_id": "DEVICE-001",
  "timestamp": "2026-09-10T12:00:00Z",
  "ph": 7.8,
  "temperature": 28.4,
  "tds": 1850,
  "turbidity": 32.1,
  "water_level": 82.4,
  "do": 5.6
}
```

## 10.1 Data Quality

- Missing values
- Outliers
- Sensor drift
- Stuck values
- Impossible values
- Timestamp errors
- Calibration
- Communication loss

```text
Raw Data
 ↓
Validation
 ↓
Cleaning
 ↓
Quality Monitoring
 ↓
Dataset
```

---

# 11. External Environment Data

## 11.1 Weather

候補：

- Temperature
- Humidity
- Rainfall
- Wind speed
- Wind direction
- Solar radiation
- Atmospheric pressure
- Weather forecast

## 11.2 Air Quality

候補：

- PM2.5
- PM10
- NO2
- SO2
- CO
- O3
- AQI

注意：

> 大気汚染度がエビ養殖環境へどの程度影響するかは、先行研究・実測データを確認した上で特徴量として採用する。

---

# 12. Feed / Farm Operation Data

## 12.1 Feed

候補：

- Feed type
- Brand / Product
- Composition
- Protein content
- Lipid content
- Feed size
- Feeding amount
- Feeding frequency
- Feeding time

## 12.2 Farm Operation

- Stocking
- Water exchange
- Pump operation
- Nutrient input
- Treatment
- Maintenance
- Harvest

## 12.3 ML Dataset

```text
Pond Environment
+
External Environment
+
Feed
+
Farm Operation
+
Historical Production
        ↓
      Dataset
```

---

# 13. Recommended Threshold Architecture

## 13.1 Threshold

閾値はコードへ固定せず、設定可能なデータとして管理する。

```text
Recommended Range
        ↓
Threshold Configuration
        ↓
Alert Engine
```

## 13.2 Levels

- Normal
- Watch
- Warning
- Critical

## 13.3 Context-aware Threshold

将来的には、

```text
Species
+
Shrimp Age
+
Growth Stage
+
Season
+
Pond Condition
+
Historical Data
```

に応じた閾値を検討する。

※具体的な閾値はFAO・研究論文・専門家・現場条件を確認して決定する。

---

# 14. Alert System

## 14.1 Current-state Alert

```text
Sensor Data
 ↓
Threshold Check
 ↓
Normal / Warning / Critical
 ↓
Notification
```

## 14.2 Predictive Alert

```text
Sensor + Environment + Feed + History
 ↓
ML
 ↓
Future Risk
 ↓
Notification
```

## 14.3 Notification Content

- 何が起きているか
- 問題のパラメータ
- 現在値
- 推奨範囲
- トレンド
- 将来予測
- リスク
- 推奨アクション
- 発生時刻

---

# 15. Frontend / Dashboard

## 15.1 Main Dashboard

管理者が一目で池の状態を把握できる。

```text
┌──────────────────────────────┐
│       SHRIMP POND             │
├──────────────────────────────┤
│ Overall Status     Watch      │
├──────────────────────────────┤
│ pH          7.8               │
│ Temperature 28.4°C            │
│ TDS         1850 ppm          │
│ Turbidity   32 NTU            │
│ Water Level 82 cm              │
│ DO          5.6 mg/L          │
├──────────────────────────────┤
│ Future Risk       68%         │
├──────────────────────────────┤
│ Recommended Action             │
│ Check water condition          │
└──────────────────────────────┘
```

## 15.2 Views

- Current Status
- Historical Data
- Prediction
- Alerts
- Pump Status
- AI Recommendation
- Alert History
- Feed / Operation History

---

# 16. Machine Learning Architecture

## 16.1 MLの役割

1. Anomaly Detection
2. Environmental Prediction
3. Future Risk Prediction
4. Growth / Production Prediction
5. Optimization Support

---

# 17. ML Dataset

## 17.1 Input

```text
Pond IoT
├── pH
├── Temperature
├── TDS
├── Turbidity
├── Water Level
└── DO

External Environment
├── Weather
├── Rainfall
├── Humidity
├── Air Temperature
└── Air Quality

Farm Operation
├── Feed
├── Feeding Amount
├── Feeding Time
├── Stocking
├── Water Exchange
└── Pump Operation
```

## 17.2 Target Variables

### Phase 1

- Environmental anomaly
- Risk classification

### Phase 2

- Future sensor values
- Future environmental condition

### Phase 3

- Shrimp growth
- Biomass
- Survival
- Harvest volume

---

# 18. ML Learning Strategy

## 18.1 Data Scarcity

実運用開始時には十分な学習データがない可能性が高い。

したがって、

> **「最初から高精度な予測AIを作る」のではなく、「データを蓄積しながらモデルを改善する」**

という設計とする。

## 18.2 Stage 0

```text
Rules
+
Recommended Threshold
```

## 18.3 Stage 1

```text
Historical Data
 ↓
Anomaly Detection
```

## 18.4 Stage 2

```text
Historical Time Series
+
Environment
+
Feed
+
Operation
 ↓
Future Prediction
```

## 18.5 Stage 3

```text
Prediction
+
Production Outcome
 ↓
Growth / Yield Model
```

---

# 19. Predictive Alert Architecture

```text
                 Current State
                      │
                      ▼
             Feature Engineering
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
     IoT           Weather          Feed
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                     ML
                      │
                      ▼
              Future Prediction
                      │
                      ▼
                Risk Assessment
                      │
             ┌────────┴────────┐
             ▼                 ▼
          Low Risk          High Risk
                                │
                                ▼
                           Notification
                                │
                                ▼
                        Recommended Action
```

重要な評価指標：

> **Lead Time：実際の環境悪化よりどれだけ早く通知できたか**

---

# 20. AI Agent Architecture

## 20.1 Agent Definition

> **センサーデータ、外部環境データ、養殖管理データ、ML予測、システム状態を統合し、養殖者が次に取るべき行動を支援する意思決定オーケストレーター**

AgentそのものがMLモデルではない。

## 20.2 Agent Tools

```text
get_current_sensor_data()
get_historical_data()
get_weather_data()
get_air_quality_data()
get_feed_records()
get_operation_records()
get_thresholds()
get_alerts()
get_predictions()
get_pump_status()
create_alert()
generate_recommendation()
request_control()
generate_daily_report()
```

## 20.3 Workflow

```text
Event / User Request
        ↓
      Agent
        ↓
     Observe
        ↓
  Retrieve Data
        ↓
    Analyze State
        ↓
    Call ML Model
        ↓
   Assess Future Risk
        ↓
    Plan Response
        ↓
    Safety Check
        ↓
┌───────┴────────┐
▼                ▼
Notify Farmer   Control Request
                   │
                   ▼
              Safety Layer
                   │
                   ▼
                  PID
                   │
                   ▼
                 Pump
                   │
                   ▼
                 Pond
                   │
                   ▼
                Sensors
                   ↺
```

---

# 21. Agent Autonomy Levels

## Level 1 — Monitoring

```text
Sensor
 ↓
Agent
 ↓
Status Summary
```

## Level 2 — Alert

```text
Sensor / ML
 ↓
Agent
 ↓
Notification
```

## Level 3 — Recommendation

```text
Sensor + ML + Context
 ↓
Agent
 ↓
Recommended Action
 ↓
Farmer
```

## Level 4 — Supervised Control

```text
Agent
 ↓
Control Proposal
 ↓
Human Approval
 ↓
Safety
 ↓
PID
 ↓
Pump
```

## Level 5 — Autonomous Control

```text
Agent
 ↓
Safety
 ↓
PID
 ↓
Pump
```

2か月の開発では**Level 2～3を主要目標**とし、Level 4を可能な範囲で実証する。Level 5は将来構想とする。

---

# 22. PID Control Architecture

## 22.1 PIDの役割

PIDは、

> **決定された目標値に対して、ポンプ等のアクチュエータを安定的に制御する低レベル制御**

を担当する。

## 22.2 Control Loop

```text
Target
 ↓
Error
 ↓
PID Controller
 ↓
Pump Output
 ↓
Pond
 ↓
Sensor
 ↓
Feedback
 ↺
```

## 22.3 Responsibility Separation

```text
ML
 ↓
Predict
 ↓
Agent
 ↓
Decide / Recommend
 ↓
Safety Layer
 ↓
PID
 ↓
Control
 ↓
Pump
```

原則：

> **Agent / LLMが直接ポンプの低レベル制御を行わない。**

---

# 23. Safety Architecture

```text
Agent
 ↓
Action Proposal
 ↓
Safety Layer
 ├── Range Check
 ├── Rate Limit
 ├── Runtime Limit
 ├── Sensor Validation
 ├── Pump Status Check
 ├── Emergency Stop
 └── Human Override
 ↓
PID
 ↓
Pump
```

自動制御を実環境へ導入する場合、Safety Layerを必須とする。

## Fail-safe

以下の場合は自動制御を停止または安全側へ移行する。

- Sensor failure
- Missing data
- Communication failure
- Abnormal sensor values
- Pump failure
- ML confidence too low
- Agent uncertainty
- Safety threshold violation

---

# 24. Human + AI Architecture

AIが人間を完全に置き換えることを目的としない。

```text
AI
├── Monitor
├── Analyze
├── Predict
├── Alert
└── Recommend

Human
├── Domain Knowledge
├── Final Decision
├── Override
└── Exceptional Response
```

初期段階では、

> **AIは「判断を支援する存在」**

として導入し、十分な検証が進んだ領域から自動化する。

---

# 25. Digital Twin / Simulation

## 25.1 Purpose

実池での制御実験にはリスクがあるため、仮想養殖池を利用して検証する。

## 25.2 Virtual Pond

```text
Virtual Pond
├── pH
├── Temperature
├── TDS
├── Turbidity
├── Water Level
└── DO
```

## 25.3 Closed-loop Simulation

```text
Virtual Pond
 ↓
Sensor Simulation
 ↓
Data Platform
 ↓
ML
 ↓
Agent
 ↓
Safety
 ↓
PID
 ↓
Virtual Pump
 ↓
Virtual Pond
 ↺
```

## 25.4 Test Scenarios

- Normal condition
- DO decrease
- Temperature increase
- pH change
- Turbidity increase
- Water level decrease
- Rainfall event
- Multiple simultaneous anomalies
- Sensor failure
- Network failure
- Pump failure

---

# 26. Backend Architecture

候補：

- FastAPI
- PostgreSQL
- MQTT Broker
- Python
- Docker

最終技術選定はArchitecture Phaseで決定する。

```text
IoT
 ↓
MQTT
 ↓
Ingestion Service
 ↓
Validation
 ↓
Database
 ↓
API
 ↓
Frontend
ML
Agent
```

---

# 27. Frontend Architecture

Main Views：

```text
Dashboard
 ├── Current Status
 ├── Sensor Monitoring
 ├── Prediction
 ├── Alerts
 ├── Pump Status
 ├── AI Recommendation
 ├── Feed / Operation
 └── History
```

管理者が、

- 閾値
- センサー
- 池
- ポンプ
- 通知
- AI設定

を管理できる構造を検討する。

---

# 28. Notification Architecture

通知先候補：

- Web Dashboard
- Browser Notification
- Email
- LINE
- SMS
- Mobile Push

MVPでは実装コストを考慮して1つ以上を選択する。

通知には、

- Current Value
- Recommended Range
- Trend
- Future Risk
- Recommended Action
- Timestamp

を含める。

---

# 29. Functional Requirements

## IoT

- センサー値取得
- Timestamp
- Device ID
- Pond ID
- Sensor health
- Communication status
- Calibration information

## Data

- Telemetry ingestion
- Environmental data ingestion
- Feed record
- Operation record
- Historical storage
- Data validation

## Monitoring

- Real-time dashboard
- Historical charts
- Status indicators

## Threshold

- Threshold management
- Warning / Critical
- Context-aware configuration

## Alert

- Threshold alert
- Predictive alert
- Notification
- Alert history
- Response logging

## ML

- Dataset creation
- Preprocessing
- Anomaly detection
- Prediction
- Evaluation
- Model versioning

## Agent

- Tool calling
- Context retrieval
- Situation analysis
- Recommendation
- Report generation
- Action proposal

## Control

- PID
- Pump operation
- Safety limit
- Manual override
- Emergency stop

---

# 30. Non-functional Requirements

## Reliability

- Sensor failure handling
- Network failure handling
- Backend failure handling
- Pump failure detection

## Safety

- Safe fallback
- Human override
- Emergency stop
- Autonomous control limits

## Security

- Authentication
- Authorization
- Device authentication
- API security
- Audit log

## Scalability

```text
1 Pond
 ↓
Multiple Ponds
 ↓
Farm
 ↓
Multiple Farms
```

へ拡張可能な構造を目指す。

---

# 31. Evaluation

## 31.1 Technical

- Sensor accuracy
- Data latency
- API latency
- System uptime
- ML performance
- Alert accuracy
- Control stability

## 31.2 ML

### Anomaly Detection

- Precision
- Recall
- F1
- False Positive Rate
- False Negative Rate

### Prediction

- MAE
- RMSE
- R²
- Forecast horizon

### Predictive Alert

- Lead Time
- Early Detection Rate
- False Alarm Rate

## 31.3 Aquaculture

可能な範囲で、

- Survival Rate
- Growth Rate
- Shrimp Size
- Biomass
- Harvest Volume
- Feed Conversion Ratio
- Water usage
- Energy usage

を評価する。

## 31.4 Human

- Monitoring Time
- Manual Operations
- Alert Response Time
- Number of Manual Interventions
- Decision Time

---

# 32. Baseline Comparison

```text
A. Traditional Management
B. IoT Monitoring
C. IoT + Threshold Alert
D. IoT + ML
E. IoT + ML + Agent
F. IoT + ML + Agent + Control
```

比較する観点：

1. 異常検知
2. 予測精度
3. Lead Time
4. 誤通知
5. 環境安定性
6. 作業負担
7. 生産性

---

# 33. Development: Agentic SDLC

システム開発自体にもAgentic SDLCを適用する。

```text
Human Goal
    ↓
Guide
    ↓
Generate
    ↓
Verify
    ↓
Solve
    ↓
Review
    ↓
Iterate
```

## Human

- Goal definition
- Domain decisions
- Requirement approval
- Architecture approval
- Safety decisions
- Final review

## AI Agent

- Research support
- Documentation
- Code generation
- Test generation
- Debugging
- Data analysis
- Refactoring
- Verification support

---

# 34. Development Agents

## Management Agent

- Project requirements
- Task decomposition
- Progress management
- Documentation coordination

## Design Agent

- System architecture
- API
- Data model
- UI design
- ADR

## Data / ML Agent

- Dataset
- Feature engineering
- Model training
- Evaluation
- Data quality

## Builder Agent

- IoT
- Backend
- Frontend
- Integration
- Infrastructure

## Verify Agent

- Unit tests
- Integration tests
- Simulation
- Failure tests
- Regression tests
- Requirement verification

---

# 35. Development Workflow

```text
Human Requirement
       ↓
Management Agent
       ↓
Design Agent
       ↓
Architecture
       ↓
Data / ML Agent
       ↓
Builder Agent
       ↓
Verify Agent
       ↓
Human Review
       ↓
Iteration
```

重要な設計・安全判断はHumanが承認する。

---

# 36. Design Principles

## Principle 1 — Observe before Automate

まず正確に観測する。

## Principle 2 — Data before ML

十分なデータ品質を確保してからMLを高度化する。

## Principle 3 — Prediction before Autonomous Control

予測精度を検証してから自動制御へ進む。

## Principle 4 — Agent does not directly control Hardware

AgentとHardwareをSafety Layerで分離する。

## Principle 5 — Human remains the Supervisor

初期段階では人間が最終判断を行う。

## Principle 6 — Simulation before Real-world Control

実池への制御前にSimulationで検証する。

## Principle 7 — Measure Lead Time

予測精度だけでなく、異常発生前にどれだけ早く通知できたかを評価する。

---

# 37. Final System Architecture

```text
                       Farms Manager
                            │
                            ▼
                  ┌─────────────────┐
                  │   Dashboard     │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │   Farm Agent    │
                  └────────┬────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Current      Historical    Context
            Data          Data        Data
              │            │            │
              └────────────┼────────────┘
                           ▼
                          ML
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Anomaly       Prediction    Risk
          Detection                    Score
                           │
                           ▼
                     Recommendation
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 Farmer        Safety
                                  │
                                  ▼
                                 PID
                                  │
                                  ▼
                                Pump
                                  │
                                  ▼
                             SHRIMP POND
                                  │
             ┌────────────────────┼──────────────────┐
             ▼                    ▼                  ▼
          pH Sensor          Temperature          TDS
             │                    │                  │
             ▼                    ▼                  ▼
        Turbidity           Ultrasonic              DO
             │                    │                  │
             └────────────────────┼──────────────────┘
                                  │
                                  ▼
                            Data Platform
                                  ▲
                                  │
                 ┌────────────────┴────────────────┐
                 │                                 │
                 ▼                                 ▼
          Weather / Air                      Feed / Operation
           Environment                          Records
```

---

# 38. Final Vision

```text
             SHRIMP FARM
                  │
           Continuous Data
                  │
                  ▼
           Data Platform
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
     Pond       Weather     Feed
      IoT       / Air       / Farm
                 Environment Operation
       └──────────┼──────────┘
                  ▼
                 ML
                  │
           Future Prediction
                  │
                  ▼
                Agent
                  │
         Decision / Recommendation
                  │
          ┌───────┴────────┐
          ▼                ▼
       Farmer           Control
                           │
                         Safety
                           │
                          PID
                           │
                          Pump
                           │
                           ▼
                         Pond
                           │
                           └────────↺
```

## Final Message

> **「養殖者が池を監視する」のではなく、システムが池を継続的に観測・理解・予測し、養殖者には重要な判断と対応だけを求める。**

最初は閾値ベースの監視から始め、データが蓄積されるにつれてMLによる予測へ進み、最終的にはAI Agentと制御システムによる半自律・自律的な養殖環境管理を目指す。

---
