# Business Requirements Document

### Target: Integrated Shrimp Farm Management System

## 1. Project Overview

- **System overview:** A hierarchical (Farms Manager / Technical Manager) integrated management web system for Indonesian shrimp farms using IoT sensors and data analysis
- **Target company / target aquaculture ponds:** A shrimp farming company operating multiple Estates in Indonesia, with several dozen to several hundred Ponds in each Estate. Farms Managers are located in urban areas such as Jakarta, while aquaculture sites are distributed in rural areas
- **Background:** It is difficult to monitor the status of multiple Farms distributed across large areas in real time, estimate the business impact when abnormalities occur, and maintain accuracy in harvest timing and shipment planning

---

## 2. Current Issues (As-Is)

- **Current aquaculture management process:** Visual monitoring and simple measurements are performed in the field, and the status is reported to the management building / Farms Manager (digital integration is not yet established)
- **Current recording method and its problems:**
  - The status of Farms distributed across large areas cannot be monitored in real time
  - Reporting accuracy and operational practices vary between field sites, making shipment planning accuracy unstable
  - Field data (feeding, mortality, water quality, growth) is not organized in a form usable for management decisions, so decisions about harvest timing and shipment planning depend on experience
- **Hypotheses regarding past problems and their causes:**
  - Farms Managers and field staff notice signs of deteriorating water quality (such as declining DO) too late, resulting in delayed responses
  - Farms Managers cannot grasp the detailed status of each Farm, delaying management decisions during abnormalities and around harvest timing

## 3. Systemization Purpose and Goals

- **Purpose of systemization:** Organize and provide data obtained from field IoT sensors, daily inputs, and weekly sampling in the granularity and format required by each of the Farms Manager and Technical Manager (Farms Manager = analysis results, field = detailed data), thereby enabling both company-wide management decisions and rapid field-level response to abnormalities.
- **Goals to be achieved:**
  - Enable the Farms Manager to see the status of Farms across the country at a glance and quickly notice Farms where thresholds have been exceeded or abnormalities have occurred
  - Enable the Farms Manager to review, as analysis results, the recommended harvest timing and profit forecast for each Pond, the status of biological risks, and the future available shipment volume
  - Enable the Technical Manager to check detailed values from IoT devices in assigned Farms, receive threshold-exceedance alerts, and respond quickly
  - Enable AI to learn and predict changes in IoT data obtained in the field and provide advice that supports the Technical Manager's decisions
  - Establish a mechanism for the Technical Manager to report daily Pond conditions (surviving population, deaths, weather, etc.) and weekly sampling results
  - Enable Actuators such as pumps and aeration systems to be PID-controlled according to conditions such as threshold exceedance, while allowing the Technical Manager to remotely monitor, approve, and override the control

---

## 4. Scope Definition

### 4.1 In Scope

- Farms Manager integrated dashboard (status list and summary for all Farms)
- Analysis-result display for Farms Manager (three areas: revenue optimization, biological risk, and advance sales / future inventory forecast)
- Drill-down from analysis results to source data
- Technical Manager monitoring screen (detailed IoT data for Ponds within assigned Farms and threshold-exceedance alert reception)
- IoT sensor data collection at 3–5 minute intervals
- Threshold-exceedance alert functionality
- Daily report input functionality (Technical Manager)
- Weekly sampling and laboratory inspection result input, and automatic calculation of related KPIs (ABW, ADG, SR, Biomass, FCR, COGS, etc.)
- AI prediction and advice functionality based on learned changes in IoT data (for Technical Manager)
- PID control of Actuators such as pumps and aeration systems (PID control / Safety Layer), together with monitoring, approval, and override functions by the Technical Manager

### 4.2 Phasing

- **Phase 1:** Rule-based KPI calculation, revenue simulation, and future inventory forecasting (first establish "visibility and explainability")
- **Phase 2:** Introduce anomaly detection and biological risk scores (validate after sufficient historical data has accumulated)
- **Phase 3:** Further enhance prediction using machine learning (select the model approach based on data volume and accuracy validation results)

---

## 5. Assumptions and Constraints

- **Assumptions:**
  - Waterproof IoT sensors (smart probes) are installed in each Pond and automatically measure DO, pH, water temperature, turbidity, TDS, water level, etc.
  - Actuators such as pumps and aeration systems are installed in each Pond and can receive control signals from the system
  - Weekly sampling (weight measurement and shrimp counting) is conducted as part of field operations
- **Technical constraints:**
  - Communication and server architecture must support continuous data transmission at 3–5 minute intervals
- **Regulations / data sovereignty:** Confirmation is required regarding Indonesia's Personal Data Protection Law (PDP Law) and whether cross-border data transfer is permitted
- **Languages:** Indonesian is mandatory for Technical Manager interfaces; Indonesian / English bilingual support is assumed for Farms Manager interfaces
- **Development structure:** The development team consists of two people. Design, implementation, and verification will be conducted using AI agents based on the Agentic SDLC described below
- **Assumed infrastructure:** GCP (Google Cloud Platform) is assumed as the cloud platform. Sensor data is assumed to be sent to GCP through an Edge layer
- **Business rules requiring confirmation:** The definition of profit (gross profit / operating profit, etc.), market-price update rules, priority of harvest decisions, risk thresholds, the scope within which AI may make recommendations, and the treatment of supply forecasts (forecast value or saleable volume with a safety margin) will be finalized through agreement with the company

---

## 6. Stakeholders and User Definitions

| Role | Description | Information Granularity | System Permission Concept |
| --- | --- | --- | --- |
| Farms Manager | Management responsibility for centrally monitoring Farm groups nationwide from an urban location. This includes the possibility that sales and technical personnel use the same dashboard (permission differences to be confirmed) | Status of all Farms and analysis results by Pond / Farm (revenue optimization, biological risk, future inventory forecast). Raw data is generally not displayed | Company-wide dashboard viewing and drill-down viewing from analysis results |
| Technical Manager | Field-level manager responsible for Ponds within assigned Farms | Detailed IoT device values for Ponds within assigned Farms (DO, pH, water temperature, etc.), threshold-exceedance alerts, and AI prediction advice | Detailed monitoring for assigned Farms, alert confirmation, AI advice viewing, daily report / weekly sampling input, monitoring / approval / Human Override of automatic control |

---

## 7. Business Requirements

### 7.1 Data Collection and Input

- Automatically collect environmental data for each Pond from IoT sensors
- Technical Manager enters feeding amount, number of deaths, sampling results, and other required information

### 7.2 Status Monitoring and Analysis

- The system integrates collected and manually entered data
- Make it possible to review water quality, growth conditions, aquaculture KPIs, and related information
- Notify the Technical Manager when abnormalities or risks are detected

### 7.3 Field Response

- The Technical Manager checks Pond conditions and notification contents
- When necessary, performs feeding, equipment control, and other field responses
- The system supports equipment control and records control status

### 7.4 AI and Prediction Support

- Analyze and predict water quality, growth, risk, and other factors based on accumulated data
- AI presents response options to the Technical Manager
- Technical Manager reviews predictions and recommendations and decides how to respond

### 7.5 Cross-Farm Management

- Farms Manager checks the status of multiple Farms
- Reviews analysis results related to revenue, biological risk, and future shipments
- When necessary, checks detailed information and requests confirmation or gives instructions to the Technical Manager

#### Business Process Flow

![](../img/work_flow.png)

---

## 8. Main Business Rules to Be Confirmed by the Company

- Definition of profit (gross profit / operating profit / contribution margin to be optimized)
- Market-price update frequency, validity period, minimum price, and other rules
- Priorities in harvest decisions (size, profit, disease risk, contracts, logistics constraints, etc.)
- Risk thresholds separating Normal / Attention / Warning
- Scope of recommendations AI is allowed to make and areas requiring human approval
- Whether future supply volume is treated as a "forecast value" or "saleable volume after deducting a safety margin"
- Formal definitions of KPIs such as SR, FCR, and COGS (what is included and what is used as the denominator)

---

## 9. Development Structure and Approach (Agentic SDLC)

> Key points of the development structure and approach for this system, based on the system design document (ShrimpOS Agentic SDLC Basic Design)

- **Development team size:** 2 people
- **Basic policy:** Human decides "what to build and why," while AI agents handle "how to implement it." Important decisions must still be approved by a Human
- **Agent configuration:**

| Agent | Role |
| --- | --- |
| Management Agent | Overall management, task allocation, and progress management |
| Design Agent | Requirements organization, system design, API / DB / UI design |
| Data/ML Agent | IoT data design, feature engineering, ML model design, and evaluation |
| Builder Agent | Implementation of source code, infrastructure, and ML pipelines |
| Verify Agent | Independent verification of implementation, design, and ML models |

- **Basic flow:** Human → Management Agent → Design Agent / Data-ML Agent → Builder Agent → Verify Agent → Human (final approval)
- **Items requiring Human approval:**
  - Changes to system Architecture
  - Important DB / API changes
  - Adoption of an ML model for production
  - Introduction of PID control rules
  - Important changes to the production environment
- **Relationship with this Business Requirements Document:** This document is positioned as one of the inputs and deliverables of the Design Agent. Future design changes are assumed to be reflected after updating this Business Requirements Document and receiving Human approval
