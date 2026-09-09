# Smart Shrimp Pond Management System
## システム設計書・提案書 完全版アウトライン

**Version:** 0.1  
**作成日:** 2026-09-09  
**想定開発期間:** 約2か月  
**設計準備期間:** 3日  
**チーム:** 2名  

> **決定事項は [`docs/DECISIONS.md`](docs/DECISIONS.md) を正とする。**
> 本アウトラインと矛盾する場合はADRが優先される。
> Agentic SDLCの詳細設計は [`docs/12_agentic-sdlc.md`](docs/12_agentic-sdlc.md)。

---

# 0. この設計書の目的

本設計書は、エビ養殖（Shrimp Pond Farming）において、IoT・データ基盤・機械学習・PID制御・AI Agentを統合し、

- 高品質なエビの生産
- 生産量・生産性の向上
- 養殖環境の安定化
- 養殖者の監視・判断・操作負担の削減

を実現するシステムの構想・要求・アーキテクチャ・開発計画を定義する。

3日後の成果物は「完成したシステム」ではなく、**約2か月で実現可能なシステムの設計提案**とする。

---

# 1. Executive Summary

## 1.1 提案概要

### 提案するシステム

**Smart Shrimp Pond Management System**

養殖池をIoTによって継続的に観測し、取得したデータを蓄積・分析する。機械学習による異常検知・将来予測と、PID制御によるポンプ制御を組み合わせ、養殖環境の維持・改善を支援する。

さらにAI Agentをシステムの意思決定・オーケストレーション層として配置し、人間が常時監視・判断する必要を減らす。

## 1.2 Core Concept

> **From Monitoring to Intelligent Aquaculture**

または、

> **「見る養殖」から「予測し、考え、動く養殖」へ**

## 1.3 システムの基本ループ

```text
Observe
  ↓
Collect
  ↓
Understand
  ↓
Predict
  ↓
Decide
  ↓
Control
  ↓
Observe
  ↺
```

## 1.4 最終的な価値

```text
高品質
  ×
大量生産
  ×
養殖者の負担軽減
```

---

# 2. Background / Context

## 2.1 エビ養殖の概要

調査項目：

- 養殖池の構造
- 種苗投入
- 成長サイクル
- 給餌
- 水質管理
- 水交換
- ポンプ・曝気
- 収穫
- 病気・ストレス
- 生産性を左右する要因

## 2.2 養殖環境管理

FAO等の資料を参考に、養殖環境に関係する主要パラメータを整理する。

### 今回利用するセンサー

| Parameter | Sensor | Purpose |
|---|---|---|
| pH | pH Sensor | 水質状態 |
| Temperature | Temperature Sensor | 水温・成長環境 |
| TDS | TDS Sensor | 水中溶解物質の指標 |
| Turbidity | Turbidity Sensor | 濁度・水質状態 |
| Water Level | Ultrasonic Sensor | 水深・水量変化 |

### 今後検討するセンサー

- DO（Dissolved Oxygen）
- Salinity
- ORP
- Ammonia / Nitrite
- Weather
- Camera / Vision

※今回の実装範囲と将来拡張範囲を分離する。

## 2.3 既存技術・先行研究

調査対象：

- IoTによる養殖環境モニタリング
- IoT + MLによる異常検知
- 水質予測
- エビ成長予測
- 給餌最適化
- PID制御
- 自動養殖
- Computer Visionによるエビ状態推定
- Agentic AI / AI Agentを利用した農業・養殖

## 2.4 Existing Research vs Proposed System

以下を比較する。

| 機能 | Existing | Proposed |
|---|---|---|
| IoT Monitoring | ○ | ○ |
| Alert | ○ | ○ |
| ML Prediction | ○ | ○ |
| Dashboard | ○ | ○ |
| PID Control | △ | ○ |
| AI Agent | △ | ○ |
| Decision Orchestration | △ | ○ |
| Closed-loop Control | △ | ○ |
| Human Workload Reduction | △ | ○ |
| Agentic SDLC | - | ○ |

※実際の先行研究調査後に修正する。

---

# 3. Problem Definition

## 3.1 現在想定される課題

```text
養殖池
 ↓
センサー
 ↓
養殖者
 ↓
経験・判断
 ↓
ポンプ操作
```

課題候補：

- 常時監視が必要
- センサーデータが分散する
- 異常発見が遅れる
- 将来状態を把握できない
- ポンプ操作が属人的
- データ記録の負担
- 経験への依存
- 生産性と水質の関係が明確でない
- 過去データを十分活用できない

## 3.2 Problem Statement

以下を明文化する。

> 養殖環境は時間とともに変化するが、養殖者がすべての状態を継続的に監視・判断・制御することには限界がある。

## 3.3 Root Cause

```text
Data exists
  ↓
Data is not continuously interpreted
  ↓
Prediction is limited
  ↓
Decision depends on humans
  ↓
Control is manual / reactive
```

## 3.4 解決すべき本質

**Monitoring Problem**

ではなく、

**Decision & Control Problem**

として定義する。

---

# 4. Goal / Objective

## 4.1 Primary Goal

> 高品質なエビを安定して大量生産できる養殖環境を構築する。

## 4.2 Secondary Goal

> 養殖者が行う監視・分析・判断・操作を自動化・支援し、人の負担を削減する。

## 4.3 System Goals

1. 養殖環境のリアルタイム把握
2. データの長期蓄積
3. 異常検知
4. 将来状態予測
5. 適切な制御量の決定
6. ポンプ制御
7. 養殖者への通知
8. 養殖者によるOverride
9. 生産性分析
10. 継続的なデータ収集とモデル改善

---

# 5. KPI / Success Metrics

## 5.1 Aquaculture KPI

### Quality

- Survival Rate
- Shrimp Size
- Growth Rate
- Health / Stress indicators

### Production

- Harvest Volume
- Biomass
- Production per pond
- Production per unit time

### Efficiency

- Feed Conversion Ratio
- Water usage
- Energy usage
- Pump operation efficiency

## 5.2 Environmental KPI

- pH stability
- Temperature stability
- TDS stability
- Turbidity stability
- Water level stability

## 5.3 System KPI

- Sensor uptime
- Data completeness
- Prediction accuracy
- Anomaly detection performance
- Alert precision / recall
- Control stability
- System latency

## 5.4 Human KPI

- Monitoring time
- Manual operations
- Number of manual interventions
- Response time
- Daily management workload

---

# 6. User / Actor Definition

## 6.1 Farmer / Aquaculture Operator

できること：

- 池の状態確認
- センサー確認
- アラート確認
- 推奨アクション確認
- ポンプ状態確認
- 手動操作
- AI操作の承認
- AI操作の拒否
- データ確認
- レポート確認

## 6.2 System Administrator

- Device registration
- Sensor configuration
- Threshold configuration
- User management
- System monitoring

## 6.3 AI Agent

- Data retrieval
- Situation analysis
- ML invocation
- Risk assessment
- Recommendation
- Control request
- Monitoring
- Report generation

---

# 7. Use Cases

## UC-01 Real-time Monitoring

```text
Sensor
 ↓
Gateway
 ↓
Backend
 ↓
Dashboard
 ↓
Farmer
```

## UC-02 Anomaly Detection

```text
Sensor Data
 ↓
ML
 ↓
Anomaly
 ↓
Alert
 ↓
Farmer
```

## UC-03 Future Prediction

```text
Historical Data
+
Current Data
 ↓
ML
 ↓
Future Environment
```

## UC-04 Pump Control

```text
Target
 ↓
PID
 ↓
Pump
 ↓
Pond
```

## UC-05 AI Recommendation

```text
Current State
+
Historical Data
+
Prediction
 ↓
Agent
 ↓
Recommendation
 ↓
Farmer
```

## UC-06 Autonomous Control

```text
Current State
 ↓
Prediction
 ↓
Agent
 ↓
Control Policy
 ↓
PID
 ↓
Pump
```

## UC-07 Human Override

```text
Automatic Control
 ↓
Farmer Override
 ↓
Manual Control
```

## UC-08 Daily Report

```text
Daily Data
 ↓
Agent
 ↓
Summary
 ↓
Farmer
```

---

# 8. Functional Requirements

## 8.1 IoT

- センサー値取得
- センサーID管理
- timestamp付与
- device health monitoring
- calibration information
- communication status

## 8.2 Data Platform

- Telemetry ingestion
- Data validation
- Data storage
- Historical query
- Aggregation
- Data export

## 8.3 Backend

- REST API
- Authentication
- Authorization
- Device API
- Pond API
- Telemetry API
- Alert API
- Prediction API
- Control API

## 8.4 Frontend

- Dashboard
- Pond view
- Sensor charts
- Alert view
- Prediction view
- Pump control
- AI recommendation
- History
- Reports

## 8.5 Machine Learning

- Data preprocessing
- Anomaly detection
- Time-series prediction
- Model evaluation
- Model versioning
- Prediction logging

## 8.6 Control

- Target setting
- PID control
- Pump operation
- Safety limits
- Manual override
- Emergency stop

## 8.7 Agent

- Tool calling
- State analysis
- Prediction invocation
- Recommendation
- Action planning
- Control request
- Logging
- Human approval

---

# 9. Non-functional Requirements

## 9.1 Reliability

- Sensor failure handling
- Network failure handling
- Backend failure handling
- Pump failure detection
- Safe fallback

## 9.2 Safety

- Maximum pump output
- Minimum / maximum environmental limits
- Emergency stop
- Human override
- Autonomous control boundary

## 9.3 Security

- Authentication
- Authorization
- Device authentication
- API security
- Audit logs

## 9.4 Scalability

将来的に、

```text
1 Pond
 ↓
Multiple Ponds
 ↓
Farm
 ↓
Multiple Farms
```

へ拡張可能な構造とする。

---

# 10. Domain Model

## 10.1 Core Entities

```text
Farm
 └── Pond
      ├── Sensor
      ├── Pump
      ├── Telemetry
      ├── Alert
      ├── Prediction
      ├── ControlAction
      ├── FeedingRecord
      └── HarvestRecord
```

## 10.2 Data Relationships

```text
Pond
 ├── has Sensors
 ├── has Pumps
 ├── produces Telemetry
 ├── produces Alerts
 ├── receives ControlActions
 └── has ProductionRecords
```

## 10.3 Future Entities

- Shrimp Batch
- Feed
- Disease Event
- Weather
- Water Exchange
- Fertilizer / Nutrient
- Worker
- Maintenance Record

---

# 11. System Architecture

## 11.1 High-level Architecture

```text
                PHYSICAL WORLD
                     │
                  Sensors
                     │
                     ▼
              Edge / Gateway
                     │
                     ▼
                Data Platform
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
  Dashboard         ML          Agent
                     │             │
                     └──────┬──────┘
                            ▼
                       Decision
                            │
                      ┌─────┴─────┐
                      ▼           ▼
                    Farmer       PID
                                  │
                                  ▼
                                Pump
                                  │
                                  ▼
                                 Pond
```

## 11.2 Architecture Principles

1. Separation of concerns
2. Edge-first data collection
3. Cloud-based data storage
4. ML separated from control logic
5. Agent separated from low-level control
6. Human override
7. Fail-safe design
8. Observable system
9. Replaceable components
10. Simulation-first development

---

# 12. IoT Architecture

## 12.1 Sensors

```text
pH
Temperature
TDS
Turbidity
Ultrasonic
```

## 12.2 Edge Device

候補：

- ESP32
- Microcontroller
- Gateway

役割：

- Sensor reading
- Filtering
- Local validation
- Timestamp
- Communication
- Offline buffering

## 12.3 Communication

候補：

- MQTT
- Wi-Fi
- LoRa / LoRaWAN
- Ethernet

採用理由を比較する。

## 12.4 Telemetry Format

```json
{
  "pond_id": "POND-001",
  "device_id": "DEVICE-001",
  "timestamp": "2026-09-09T12:00:00Z",
  "ph": 7.8,
  "temperature": 28.4,
  "tds": 1850,
  "turbidity": 32.1,
  "water_level": 82.4
}
```

## 12.5 Sensor Data Quality

- Missing values
- Outliers
- Sensor drift
- Impossible values
- Stuck values
- Communication loss
- Calibration

---

# 13. Backend / Data Architecture

## 13.1 Backend Candidate

- FastAPI
- PostgreSQL
- MQTT Broker

## 13.2 Data Flow

```text
Sensor
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

## 13.3 Database

主要テーブル候補：

```text
farms
ponds
devices
sensors
telemetry
alerts
predictions
models
pumps
control_actions
feeding_records
harvest_records
users
agent_actions
```

---

# 14. Frontend Design

## 14.1 Dashboard

表示：

- Pond Status
- pH
- Temperature
- TDS
- Turbidity
- Water Level
- Pump Status
- Risk Level
- Prediction
- Recommended Action

## 14.2 Pond Detail

```text
Current
 ↓
History
 ↓
Prediction
 ↓
Alerts
 ↓
Control
```

## 14.3 AI Recommendation

例：

```text
Current Status
🟡 Warning

Reason:
Recent water quality change detected.

Prediction:
Risk may increase within 6 hours.

Recommendation:
Check / adjust pump operation.

[Approve] [Reject]
```

## 14.4 Human Override

必須要件として、

- Manual ON
- Manual OFF
- Output adjustment
- Auto mode
- Emergency stop

を設ける。

---

# 15. Machine Learning Architecture

## 15.1 MLの役割

MLは主に、

1. Anomaly Detection
2. Prediction
3. Production Estimation
4. Optimization support

を担当する。

## 15.2 Model 1: Anomaly Detection

Input:

```text
pH
Temperature
TDS
Turbidity
Water Level
```

Output:

```text
Normal
Warning
Critical
```

候補：

- Rule-based baseline
- Isolation Forest
- Autoencoder
- Statistical anomaly detection

## 15.3 Model 2: Time-series Prediction

```text
Past Sensor Data
+
Current Sensor Data
+
Operations
 ↓
Prediction
```

予測対象候補：

- pH
- Temperature
- TDS
- Turbidity
- Water Level
- Composite risk score

## 15.4 Model 3: Production Prediction

将来的に、

```text
Water Environment
+
Feeding
+
Stocking
+
Growth
+
Weather
 ↓
Expected Biomass / Harvest
```

を予測する。

## 15.5 Data Availability Problem

重要事項：

> 実データが少ない状態では、高精度な生産予測モデルを構築することは困難。

したがって、

### Phase 1

Rule-based / anomaly detection

### Phase 2

Environment prediction

### Phase 3

Growth / production prediction

と段階化する。

---

# 16. PID Control Architecture

## 16.1 PIDの役割

PIDは、

> **現在値を目標値へ安定して近づける低レベル制御**

を担当する。

## 16.2 Control Loop

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

## 16.3 MLとの役割分担

```text
ML
 ↓
Prediction
 ↓
Optimal Target / Risk
 ↓
PID
 ↓
Pump
```

原則：

> MLが直接ポンプを細かく制御するのではなく、PID等の検証済み制御系を介して実環境を操作する。

---

# 17. Agent Architecture

## 17.1 Agent Definition

今回のAgentは、

> **養殖環境に関する複数の情報源・ツールを利用し、状況を理解し、適切な次の行動を計画する意思決定オーケストレーター**

として定義する。

## 17.2 Agent Tools

```text
get_current_sensor_data()
get_historical_data()
get_pump_status()
get_prediction()
get_alerts()
get_farm_context()
set_pump_target()
create_alert()
generate_report()
```

## 17.3 Agent Workflow

```text
User / Event
     ↓
Agent
     ↓
Observe
     ↓
Analyze
     ↓
Retrieve Data
     ↓
Call ML
     ↓
Assess Risk
     ↓
Plan Action
     ↓
Safety Check
     ↓
Human Approval / Automatic
     ↓
Control
     ↓
Observe Result
```

## 17.4 Agentの自律レベル

### Level 1: Monitoring

Agent → Alert

### Level 2: Recommendation

Agent → Recommendation → Human

### Level 3: Supervised Control

Agent → Control Proposal → Human Approval → PID

### Level 4: Autonomous Control

Agent → Safety Check → PID → Pump

2か月の開発では、**Level 2～3を現実的な目標**とし、Level 4は将来構想とする。

---

# 18. Safety Architecture

自律制御ではSafety LayerをAgentより下位に置く。

```text
Agent
 ↓
Proposed Action
 ↓
Safety Layer
 ├── Range Check
 ├── Rate Limit
 ├── Runtime Limit
 ├── Emergency Stop
 └── Human Override
 ↓
PID
 ↓
Pump
```

Agentが誤った判断をしても、直接ポンプを危険な状態にできない構造を目指す。

---

# 19. Digital Twin / Simulation

## 19.1 Purpose

実池での実験には時間・コスト・リスクがあるため、シミュレーション環境を用意する。

## 19.2 Virtual Pond

```text
Virtual Pond
 ├── pH
 ├── Temperature
 ├── TDS
 ├── Turbidity
 └── Water Level
```

## 19.3 Simulation

```text
Virtual Pond
 ↓
Sensor Simulation
 ↓
ML
 ↓
Agent
 ↓
PID
 ↓
Virtual Pump
 ↓
Virtual Pond
```

## 19.4 Test Scenarios

- Normal
- pH increase
- pH decrease
- Temperature spike
- Turbidity increase
- Water level decrease
- Sensor failure
- Network failure
- Pump failure
- Multiple simultaneous anomalies

---

# 20. Agentic SDLC

`DECIDED` — ADR-004 / 詳細設計：[`docs/12_agentic-sdlc.md`](docs/12_agentic-sdlc.md)

## 20.0 中核アイデア

運用系と開発系に**同一のアーキテクチャ原則**を適用する。

```text
【運用】 Farmer → Agent → Safety Layer      → PID → Pump  → Pond
【開発】 Human  → Agent → Verification Gate → CI  → Merge → Codebase
```

Agentic SDLCを採用する理由は生産性ではなく、**プロダクトの設計思想との一貫性**である。
§36のチームルール（Agentに直接ハードウェアを触らせない／Safety Layerを必ず通す／Human Override）が、
そのまま開発プロセスの規律として再利用される。

## 20.1 Development Philosophy

今回の開発では、AIを単なるコード生成器ではなく、開発プロセスの各工程を支援するAgentとして利用する。

## 20.2 Basic Loop

```text
GUIDE
 ↓
GENERATE
 ↓
VERIFY
 ↓
SOLVE
 ↓
REVIEW
 ↓
ITERATE
```

## 20.3 Human Role

Human：

- Goal definition
- Domain decision
- Architecture approval
- Safety decision
- Requirement approval
- Final review

AI Agent：

- Research support
- Documentation
- Code generation
- Test generation
- Debugging
- Data analysis
- Refactoring
- Verification support

---

# 21. Development Agent Structure

## 21.1 Architecture Agent

担当：

- Requirements
- Architecture
- ADR
- Interface definition

## 21.2 IoT Agent

担当：

- Firmware
- Sensor integration
- MQTT
- Edge processing

## 21.3 Backend Agent

担当：

- API
- Database
- Authentication
- Data ingestion

## 21.4 Frontend Agent

担当：

- UI
- Dashboard
- Visualization

## 21.5 ML Agent

担当：

- Data preprocessing
- Feature engineering
- Model training
- Evaluation

## 21.6 Control Agent

担当：

- PID
- Simulation
- Control safety

## 21.7 QA Agent

担当：

- Unit test
- Integration test
- Simulation test
- Failure test
- Regression test

---

# 22. Team of Two: Collaboration Model

## 22.1 基本原則

2人で作業するため、担当を完全分離するのではなく、

> **領域担当 + 共有設計**

とする。

## 22.2 推奨担当

### Member A: System / IoT / Control

主担当：

- Domain research
- System architecture
- IoT
- MQTT
- Backend
- PID / Control

### Member B: Intelligence / Application

主担当：

- ML
- Agent
- Frontend
- Data analysis
- Evaluation

ただし、重要な設計判断は必ず共有する。

## 22.3 Shared Areas

両者で必ずレビュー：

- Requirements
- Architecture
- Data Model
- API
- ML assumptions
- Agent permissions
- Safety
- Final presentation

---

# 23. Shared Repository Structure

```text
smart-shrimp-farm/
│
├── README.md
│
├── docs/
│   ├── 00_project-overview.md
│   ├── 01_domain.md
│   ├── 02_requirements.md
│   ├── 03_use-cases.md
│   ├── 04_architecture.md
│   ├── 05_iot.md
│   ├── 06_backend.md
│   ├── 07_frontend.md
│   ├── 08_ml.md
│   ├── 09_control.md
│   ├── 10_agent.md
│   ├── 11_safety.md
│   ├── 12_agentic-sdlc.md
│   ├── 13_evaluation.md
│   ├── 14_roadmap.md
│   └── decisions/
│
├── frontend/
├── backend/
├── iot/
├── ml/
├── simulation/
└── tests/
```

---

# 24. Design Decision Management

設計判断はGitHub Issue / ADR等に残す。

## ADR Template

```text
# ADR-001: MQTTを採用する

## Context
なぜ通信方式を決める必要があるか

## Options
- MQTT
- HTTP
- LoRaWAN

## Decision
MQTTを採用

## Reason
...

## Consequences
...

## Open Questions
...
```

これにより、2人の認識差を減らす。

---

# 25. Requirements Traceability

各要件がどの機能につながっているかを管理する。

```text
Business Goal
 ↓
Requirement
 ↓
Use Case
 ↓
System Component
 ↓
Implementation
 ↓
Test
 ↓
KPI
```

例：

```text
Workload Reduction
 ↓
Automatic Monitoring
 ↓
Real-time Dashboard
 ↓
IoT + Backend
 ↓
Integration Test
 ↓
Monitoring Time
```

---

# 26. Development Roadmap: 2 Months

## Week 1: Research & Architecture

- Domain research
- Existing research
- Requirements
- Architecture
- Data model
- API design
- Agent design
- Safety design

## Week 2: IoT Foundation

- Sensor interface
- ESP32
- MQTT
- Telemetry
- Data validation

## Week 3: Backend / Database

- PostgreSQL
- FastAPI
- Data ingestion
- API
- Authentication

## Week 4: Frontend

- Dashboard
- Pond detail
- Sensor charts
- Alerts
- Pump status

## Week 5: Anomaly Detection Service

`DECIDED` — ADR-003

- Data pipeline
- 予測・検知層の**インターフェース確定**（入出力スキーマ）
- Phase 1実装：ルールベース／統計（レンジ・移動平均±3σ・変化率・stuck値・欠損）
- MLモデルへの差し替え可能性の検証

※学習モデル構築はPhase 2。余剰工数はWeek 7 / Week 8とバッファへ再配分する。

## Week 6: Control

- PID
- Pump interface
- Simulation
- Safety limits

## Week 7: Agent

- Agent tools
- Decision workflow
- Recommendation
- Human approval
- Control integration

## Week 8: Integration / Evaluation

- End-to-end integration
- Simulation
- Failure testing
- UX improvement
- Documentation
- Presentation

---

# 27. MVP Definition

2か月で最低限完成させる範囲：

```text
Sensors / Simulator
       ↓
MQTT
       ↓
Backend
       ↓
Database
       ↓
Dashboard
       ↓
Anomaly Detection
       ↓
Prediction
       ↓
Agent Recommendation
       ↓
PID Simulation
```

### 検証環境

実際の養殖池は用いず、**センサー実機＋給排水ポンプによるベンチスケール環境**で検証する。
水位制御に関しては、以下のクローズドループが実機で完結する。

```text
実センサー → MQTT → Backend → DB → Agent → Safety → PID(Edge) → 実ポンプ
     ↑                                                              ↓
     └──────────────── 実際に水位が変化する ──────────────────┘
```

> 動かない大きな構想ではなく、**小さくても実機で1周するループ**を成果物とする。
> スケール軸（1池→複数池）とインテリジェンス軸（ルール→ML）は、その上に拡張する。

実際の養殖池へのポンプ制御適用は、環境・安全性・設備状況に応じて段階的に導入する。

---

# 28. Future Roadmap

## Phase 1

Monitoring

## Phase 2

Prediction

## Phase 3

Decision Support

## Phase 4

Supervised Automation

## Phase 5

Autonomous Aquaculture

## Phase 6

Multi-Pond Optimization

## Phase 7

Farm-level Optimization

---

# 29. Risks

## Technical

- センサー精度
- センサードリフト
- 通信障害
- データ不足
- ML精度不足
- PID tuning
- Agent hallucination
- Hardware failure

## Operational

- 実池への導入リスク
- 誤制御
- 養殖者の信頼
- メンテナンス

## Data

- 学習データ不足
- ラベル不足
- データ品質
- Pondごとの差

## Scope

- **観測範囲の限界**：本システムはDO（溶存酸素）・アンモニア等、
  斃死に直結しうる主要因子を今回の観測対象に含まない。Phase 2で追加する（§2.2）
- 実際の養殖池を持たないため、検証はベンチスケール実機＋シミュレーションで行う

---

# 30. Mitigation

```text
AI
 ↓
Safety Layer
 ↓
PID
 ↓
Pump
```

- Human Override
- Simulation-first
- Rule-based fallback
- Sensor validation
- Logging
- Model versioning
- Gradual autonomy
- Manual mode

---

# 31. Evaluation Plan

## 31.1 Technical Evaluation

- Sensor accuracy
- Data latency
- API latency
- System uptime
- ML performance
- Alert accuracy
- Control stability

## 31.2 Operational Evaluation

- Monitoring time reduction
- Manual operation reduction
- Response time
- Number of missed anomalies

## 31.3 Aquaculture Evaluation

可能な範囲で、

- Survival Rate
- Growth Rate
- Harvest Volume
- Feed efficiency
- Environmental stability

を比較する。

## 31.4 Agentic SDLC Evaluation

開発プロセス自体も評価対象とする（詳細：`docs/12_agentic-sdlc.md` §12.7）。

- Gate初回通過率
- Human修正率
- SOLVEエスカレーション率
- Issue→PR リードタイム
- ADR整合違反の検出数
- Humanレビュー時間

> 運用系KPIが「養殖者の監視時間削減」であるのと対応し、
> 開発系KPIは「開発者のレビュー時間削減」となる。同じ主張を2層で検証する。

## 31.5 Baseline Comparison

```text
Traditional
vs
IoT Monitoring
vs
IoT + ML
vs
IoT + ML + Agent + Control
```

---

# 32. Demonstration Scenario

最終デモでは、以下のシナリオを想定する。

## Scenario

1. 池は正常状態
2. センサー値が変化
3. システムが異常兆候を検知
4. MLが将来リスクを予測
5. Agentが状況を分析
6. 推奨アクションを生成
7. Humanが承認
8. PIDがポンプを制御
9. 池の状態が変化
10. センサーが再観測
11. Agentが結果を確認

```text
Sensor
 ↓
ML
 ↓
Agent
 ↓
Human Approval
 ↓
PID
 ↓
Pump
 ↓
Pond
 ↓
Sensor
```

---

# 33. Presentation Structure

3日後の提案発表は以下を推奨する。

## Slide 1

Title

**Smart Shrimp Pond Management System**

## Slide 2

Background

## Slide 3

Shrimp Farming Problem

## Slide 4

Current Workflow

## Slide 5

Our Goal

## Slide 6

Proposed Concept

## Slide 7

IoT Sensors

## Slide 8

System Architecture

## Slide 9

Data Flow

## Slide 10

Machine Learning

## Slide 11

PID Control

## Slide 12

AI Agent

## Slide 13

Closed-loop Architecture

## Slide 14

Human + AI

## Slide 15

Agentic SDLC

## Slide 16

Development Agents

## Slide 17

2-Month Roadmap

## Slide 18

MVP

## Slide 19

Evaluation

## Slide 20

Future Vision

---

# 34. Open Questions

3日間で必ず明確化・確認する項目。

## Domain

- エビの種類
- 池のサイズ
- 水量
- 養殖密度
- 養殖期間
- 給餌方法
- 水交換方法

## Hardware

- センサーの具体的型番
- センサー精度
- ESP32等のEdge Device
- Pump仕様
- Pumpを何の目的で制御するか
- 通信方式
- 電源

## Control

`DECIDED` — ADR-001（本カテゴリはクローズ）

| 項目 | 決定 |
|---|---|
| pH / 水温 / TDS / Turbidity を制御するか | **No** — 観測・判断材料のみ |
| 水位を制御するか | **Yes** — 唯一のPID制御対象 |
| 操作変数 | 給排水ポンプ |
| PID Input / Output / Setpoint | 水位[cm] / ポンプduty[-100〜+100%] / 目標水位[cm] |

残Open Question：ポンプ駆動方式（ON/OFF or 流量可変）— ADR-002

## ML

`DECIDED` — ADR-003（本カテゴリはクローズ）

| 項目 | 決定 |
|---|---|
| 利用可能な実データ量 | 実質なし前提。学習モデルはPhase 2 |
| データ取得頻度 | 制御用（水位）1Hz／記録用（水質）1分 — ADR-002 |
| ラベルの有無 | なし → 教師なし・ルールベース |
| 過去の養殖記録 / 収穫 / 給餌 | なし → Domain Modelには残しPhase 2 |

## Agent

- Agentにどこまで権限を与えるか
- Human approvalが必要な操作
- 自動制御可能な操作
- Safety constraints
- Agentのログ保存

## Business

- 生産量をどのように評価するか
- 品質をどのように定義するか
- 現場導入条件
- システム導入コスト
- ROI

---

# 35. 3日間の作業計画

## Day 1 — Understand

### Team A

- Shrimp farming research
- FAO analysis
- Existing research
- IoT requirements
- Control requirements

### Team B

- Existing IoT + ML systems
- Agentic AI
- Agentic SDLC
- ML feasibility
- Data requirements

### Joint

- Problem definition
- Goal
- Requirements
- Open Questions

### Day 1 Deliverables

```text
domain.md
requirements.md
research.md
problem-definition.md
```

---

# Day 2 — Design

### Team A

- IoT architecture
- Backend architecture
- Control architecture
- Safety

### Team B

- ML architecture
- Agent architecture
- Frontend
- Data model

### Joint

- Overall architecture
- Data flow
- Interface definition
- Agent permissions

### Day 2 Deliverables

```text
architecture.md
iot.md
backend.md
ml.md
agent.md
control.md
safety.md
```

---

# Day 3 — Proposal

### Team A

- Development roadmap
- Technical feasibility
- Risk

### Team B

- Evaluation
- Agentic SDLC
- Presentation

### Joint

- Final architecture
- MVP
- 2-month roadmap
- Presentation
- Q&A preparation

### Day 3 Deliverables

```text
proposal.md
roadmap.md
evaluation.md
presentation
```

---

# 36. Team Collaboration Rules

## Rule 1

**設計変更は共有する。**

## Rule 2

**重要な設計判断はADRに記録する。**

## Rule 3

**コードより先にInterfaceを決める。**

## Rule 4

**MLとAgentとPIDの責任範囲を混ぜない。**

## Rule 5

**Agentに直接Hardwareを操作させない。**

## Rule 6

**Safety Layerを必ず通す。**

## Rule 7

**実データがない部分はSimulationで検証する。**

## Rule 8

**「できること」と「仮説」を明確に分ける。**

---

# 37. Core Architecture Principle

このプロジェクトでは、以下の責任分離を原則とする。

```text
Sensor
  ↓
"Observe"

Data Platform
  ↓
"Remember"

ML
  ↓
"Predict"

Agent
  ↓
"Decide"

PID
  ↓
"Control"

Pump
  ↓
"Act"

Farmer
  ↓
"Supervise"
```

これをシステム設計上の最重要原則とする。

---

# 38. Final Vision

最終的に目指すシステム：

```text
                    FARMER
                       │
                 Supervision
                       │
                       ▼
              ┌────────────────┐
              │   FARM AGENT   │
              └───────┬────────┘
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
          Observe   Predict   Decide
             │        │        │
             ▼        ▼        ▼
          IoT       ML       Policy
             │        │        │
             └────────┼────────┘
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
                 SHRIMP POND
                      │
                      ▼
                   SHRIMP
                      │
                      └───────────────┐
                                      │
                                      ▼
                                   Sensors
                                      │
                                      └──────→ Agent
```

## 最終的な思想

> **人間が養殖池を常に監視するのではなく、システムが養殖池を継続的に観測・理解・予測し、人間には重要な判断だけを求める。**

---

# 39. 3日後に「まだ決めなくてよいこと」

以下は提案段階で無理に確定しない。

- 最終MLモデル
- 最終LLM
- Agent framework
- センサーの最終型番
- クラウドサービス
- PIDの最終パラメータ
- Production predictionの精度
- 完全自律制御

これらは、

**Requirements → Prototype → Evaluation**

の順に決定する。

---

# 40. 3日後に「必ず決めること」

最低限、以下はチームとして合意する。

```text
[Business]
□ 何を改善するのか
□ 成功とは何か

[Domain]
□ 池で何が起きているか
□ 何を観測するか

[System]
□ どんなデータフローか
□ 各コンポーネントの役割

[ML]
□ 何を予測するか
□ データは何が必要か

[Control]
□ 何を制御するか
□ PIDは何を担当するか

[Agent]
□ Agentは何を判断するか
□ 何を自動化するか
□ 何を人間が承認するか

[Development]
□ Agentic SDLCをどう使うか
□ 2か月で何を完成させるか

[Evaluation]
□ 何をもって成功とするか
```

---

# 41. Design Status

このドキュメントは**Living Document**として扱う。

設計段階では以下のステータスを使用する。

- `DRAFT` — 仮説
- `DISCUSSION` — チーム議論中
- `DECIDED` — チーム合意済み
- `IMPLEMENTED` — 実装済み
- `VALIDATED` — 実験・評価済み
- `DEPRECATED` — 廃止

各設計項目にステータスを付与し、2人の認識を同期する。

---

# 42. Immediate Next Actions

## 今日

1. このアウトラインをチーム共有
2. GitHub Repository作成
3. `/docs`作成
4. Open QuestionsをIssue化
5. 役割分担
6. Research開始

## 明日まで

1. Domain Model
2. Existing Research Map
3. Requirements
4. System Architecture Draft

## 3日後

1. Architecture確定
2. Agentic SDLC確定
3. MVP確定
4. 2-month Roadmap確定
5. Presentation完成

---

# Appendix A: Key Terms

| Term | Definition |
|---|---|
| IoT | 現実世界の状態をセンサーで取得する仕組み |
| Telemetry | センサーから収集される時系列データ |
| ML | データから状態・未来を推定する仕組み |
| Anomaly Detection | 通常状態からの逸脱を検出 |
| Prediction | 将来状態を推定 |
| PID | 目標値へ安定的に制御する手法 |
| Agent | ツールを利用しながら状況を理解し行動を計画するシステム |
| Decision Support | 人間の意思決定を支援する仕組み |
| Closed-loop Control | 制御結果を再び観測し、制御へフィードバックする構造 |
| Digital Twin | 現実環境をデジタル上に再現するモデル |
| Agentic SDLC | AI Agentをソフトウェア開発ライフサイクルに組み込む開発方法 |

---

# Appendix B: One-line Architecture

```text
IoT → Data → ML → Agent → Safety → PID → Pump → Pond → IoT
```

# Appendix C: One-line Development Process

```text
Human Goal → Agentic SDLC → Design → Generate → Verify → Review → Iterate
```
