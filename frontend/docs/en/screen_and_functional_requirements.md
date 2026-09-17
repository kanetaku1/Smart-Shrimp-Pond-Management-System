# Screen & Functional Requirements List

## 1. Functional Requirements List

### 1.1 Functions for Farms Manager

| No.  | Function                                          | Description                                                                                                                                                                                                        | Priority (Must/Should/Could) |
| ---- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------- |
| F-01 | Company-wide Integrated Dashboard (Top Screen)    | Displays a status summary for all Farms, including the number of Ponds, status breakdown, recommended shipment volume for the current week, etc.                                                                   | Must                         |
| F-02 | Revenue Optimization Analysis Display             | Displays the recommended harvest date, maximum profit forecast, and comparison simulations at multiple points in time (current / one week later / two weeks later, etc.) for each Pond                             | Should                       |
| F-03 | Biological Risk Analysis Display                  | Displays the health score, risk level (Normal / Attention / Warning), main factors, and recommended actions for each Pond                                                                                          | Should                       |
| F-04 | Advance Sales / Future Inventory Forecast Display | Displays the expected future shipment period, supply volume by size, and supply confidence in a calendar format or similar                                                                                         | Should                       |
| F-05 | Drill-down Function                               | Allows users to move from analysis results (scores, recommendations, etc.) to aggregated KPIs, analytical evidence, and trend summaries for confirmation. Raw sensor values are not displayed to the Farms Manager | Should                       |
| F-06 | Alert List Display                                | Displays a list of alerts including severity, Pond, occurrence time, and recommended confirmation items                                                                                                            | Must                         |

### 1.2 Functions for Technical Manager

| No.  | Function                                              | Description                                                                                                                                           | Priority (Must/Should/Could) |
| ---- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| F-07 | Real-time Pond Monitoring Screen for Assigned Farms   | Displays detailed DO, pH, water temperature, turbidity, TDS, and water-level values for each Pond, updated every 3–5 minutes                          | Must                         |
| F-08 | Threshold-exceedance Alert Reception                  | Receives alert notifications when a threshold is exceeded within an assigned Farm                                                                     | Must                         |
| F-09 | Threshold Configuration                               | Sets and adjusts thresholds for each water-quality item                                                                                               | Should                       |
| F-10 | AI Prediction Advice                                  | Learns from changes in IoT data for assigned Farms and provides forecasts of future trends and response advice                                        | Should                       |
| F-11 | Daily Report Input                                    | Inputs and saves the daily condition of each Pond, including feed amount, number of deaths collected, remaining feed, weather, etc.                   | Must                         |
| F-12 | Automatic Mortality Rate Calculation                  | Automatically calculates the mortality rate from the estimated surviving population, number of deaths on the day, etc., and reflects it in the report | Should                       |
| F-13 | Weekly Sampling Input                                 | Inputs the number of sampled shrimp, total weight, etc. for each Pond                                                                                 | Must                         |
| F-14 | Automatic Aquaculture KPI Calculation                 | Automatically calculates ABW, ADG, SR, Biomass, FCR, COGS, etc. from daily and weekly data                                                            | Must                         |
| F-15 | Automatic Actuator Control                            | Automatically controls pumps, aeration, and other equipment through PID control and the Safety Layer based on conditions such as threshold exceedance | Should                       |
| F-16 | Automatic Control Monitoring / Approval / Override UI | Allows monitoring of automatic-control operation, approval, and manual intervention (Human Override)                                                  | Must                         |

### 1.3 Common Functions

| No.  | Function                            | Description                                                                             | Priority (Must/Should/Could) |
| ---- | ----------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------- |
| F-17 | User & Permission Management (RBAC) | Controls access permissions by role (Farms Manager / Technical Manager / Administrator) | Must                         |
| F-18 | Data Export                         | Exports data in CSV / Excel and other formats for analysis                              | Must                         |

---

## 2. Data Requirements

- **Collected data items (IoT):** DO (dissolved oxygen), pH, water temperature, turbidity, TDS, water level
- **Collected data items (daily input):** feed amount, number of deaths collected, remaining feed condition, daily work report, etc.
- **Collected data items (weekly input):** number of sampled shrimp, total weight
- **Data acquisition method:** IoT data is collected automatically; daily and weekly data are entered by the Technical Manager
- **Data granularity / frequency:** IoT data every 3–5 minutes, daily data once per day, weekly data approximately once per week
- **Differences in data visibility:**
  - Technical Manager: Can directly view detailed IoT, daily, and weekly data for all Ponds within assigned Farms
  - Farms Manager: Cannot view raw sensor values or detailed field input values; can view only analysis results calculated from KPIs and Alerts (scores, recommendations, forecasts), aggregated trends, and analytical evidence
- **Data format / naming rules:** Designed in a format suitable for analysis, using naming rules that allow identification by Farm and Pond
- **Data for AI prediction advice:** Time-series IoT sensor data (changes in DO, pH, water temperature, turbidity, TDS, and water level) is used as training data

### 2.1 Daily Report Input Items (Initial Draft)

> This is an initial draft and is subject to confirmation through interviews with the field based on actual operations.

| Item                            | Description (Example)                                                             | Required             |
| ------------------------------- | --------------------------------------------------------------------------------- | -------------------- |
| Report date                     | 2026/09/14, etc.                                                                  | Required             |
| Pond                            | Pond 01, etc.                                                                     | Required             |
| Reporter                        | Person who submitted the report                                                   | Required             |
| Culture day                     | Number of days since stocking                                                     | Required             |
| Weather                         | Sunny / Cloudy / Rain, etc.                                                       | Required             |
| Estimated surviving population  | Estimated current number of surviving shrimp                                      | Required             |
| Number of deaths on the day     | Number of shrimp that died on the day                                             | Required             |
| Mortality rate                  | Automatically calculated from estimated surviving population, deaths, etc. (F-12) | Required (Automatic) |
| Number of dead shrimp collected | Number of dead shrimp collected                                                   | Required             |
| Notes                           | Free-text entry                                                                   | Optional             |
| (Other items)                   | (To be considered through future interviews)                                      | -                    |

### 2.2 Weekly Sampling Input Items (Initial Draft)

> This is also an initial draft and is subject to confirmation after the company's operational rules are finalized.

| Item                     | Description (Example)                        | Required |
| ------------------------ | -------------------------------------------- | -------- |
| Pond                     | Pond 01, etc.                                | Required |
| Sampling date and time   |                                              | Required |
| Number of sampled shrimp | Number of shrimp sampled                     | Required |
| Total weight             | Total weight of sampled shrimp               | Required |
| (Other items)            | (To be considered through future interviews) | -        |

---

## 3. External Interface Requirements

- **IoT sensors:** Waterproof smart probes installed in the field for measuring DO, pH, water temperature, turbidity, TDS, and water level
- **Actuators:** Equipment such as pumps and aeration systems subject to automatic control through PID control and the Safety Layer
- **Communication network:** LoRaWAN / 4G / 5G
- **Data platform:** Data is assumed to be collected and stored through the flow Sensor → Edge → GCP (Google Cloud Platform), in accordance with the system design document
- Integration with external data sources such as weather APIs

---

## 4. Non-functional Requirements

| Category                                | Example Requirement                                                                                                                                                                                                                                                                                                                                                                                           |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Performance                             | Response performance capable of continuously receiving and processing data at 3–5 minute intervals                                                                                                                                                                                                                                                                                                            |
| Availability                            | Handling of missing data during communication outages (LoRaWAN / 4G / 5G failures)                                                                                                                                                                                                                                                                                                                            |
| Security                                | Role-based access control (RBAC), permission control preventing Farms Manager from viewing detailed values, encrypted communication                                                                                                                                                                                                                                                                           |
| Safety (Automatic Control)              | A Safety Layer that fails safely even if automatic control malfunctions. Human Override by the Technical Manager must always take priority                                                                                                                                                                                                                                                                    |
| Usability (Farms Manager Display Rules) | Statuses are consistently displayed using colors (Normal / Attention / Warning); a one-sentence conclusion is displayed at the top; recommended actions and their rationale (main factors) are shown together; forecast and actual values are clearly distinguished; the latest data timestamp is always displayed; critical actions are marked as "Requires Confirmation" and are not automatically executed |
| Scalability                             | Architecture capable of supporting an increase in the number of Estates and Ponds                                                                                                                                                                                                                                                                                                                             |
| Operating Environment                   | Farms Manager: PC (including multi-display environments) / Technical Manager: PC, tablet, etc.                                                                                                                                                                                                                                                                                                                |
