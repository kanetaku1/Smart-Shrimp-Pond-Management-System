# UI/UX Design Document

## 1. Screen Requirements (List Level)

| Screen ID | Screen Name | Description | Primary Users |
| --- | --- | --- | --- |
| S-01 | Login Screen | | All users |
| S-02 | Company-wide Integrated Dashboard (Top Screen) | Status summary for all Farms, summaries of the three areas—revenue, risk, and future inventory—and Alert list | Farms Manager |
| S-03 | Revenue Simulation Screen | Recommended harvest date, profit forecast, and comparison of multiple points in time by Pond | Farms Manager |
| S-04 | Risk / Alert Screen | Health score, risk level, factors, and recommended actions by Pond | Farms Manager |
| S-05 | Future Inventory Calendar Screen | Forecast of available shipment volume by week and size | Farms Manager |
| S-06 | Pond Detail Screen (Drill-down) | Confirmation of source data and KPIs supporting analysis results | Farms Manager |
| S-07 | Assigned-Farm Pond Monitoring Screen | Real-time display of detailed IoT values by Pond | Technical Manager |
| S-08 | Alert Confirmation Screen | Confirmation of alert details and detailed values when thresholds are exceeded | Technical Manager |
| S-09 | Threshold Configuration Screen | Threshold configuration for each water-quality item | System Administrator / Technical Manager |
| S-10 | AI Prediction Advice Screen | Display of IoT data trend forecasts and response advice | Technical Manager |
| S-11 | Daily Report Input Screen | Input of daily reports for each Pond, including surviving population, deaths, weather, etc. | Technical Manager |
| S-12 | Weekly Report Input Screen | Input of Pond sampling results | Technical Manager |
| S-13 | Daily / Weekly Report History Screen | Viewing and searching past daily and weekly reports | Technical Manager |
| S-14 | Automatic Control Monitoring / Approval Screen | Monitoring of Actuator operation, approval of automatic control, and Human Override operations | Technical Manager |
| S-15 | User / Permission Management Screen (RBAC) | | System Administrator |

---

## 2. UI/UX Design Purpose and Principles

### 2.1 Design Purpose

This document defines screen, interaction, and display rules for the Smart Shrimp Pond Management System and serves as a standard for preventing inconsistent implementation decisions in subsequent frontend design.

The UI of this system prioritizes enabling users to quickly discover abnormalities and matters requiring decisions, confirm the rationale, and take appropriate action, rather than simply reading large amounts of data.

### 2.2 Basic Principles

1. **Show the conclusion first:** Display the current status, items requiring action, and latest data timestamp at the top of the screen.
2. **Vary information volume by Role:** Display aggregated information necessary for management decisions to Farms Manager and detailed information necessary for field response to Technical Manager.
3. **Separate forecasts from actuals:** Clearly distinguish forecast, actual, and estimated values using labels and display formats.
4. **Do not communicate status through color alone:** Use labels, icons, shapes, and text in addition to color.
5. **Require human confirmation for critical operations:** AI presents Recommendations but is not the actor responsible for Decisions or Actuator operations.
6. **Do not hide freshness or missing data:** Always display data acquisition time, update delay, communication outages, and sensor missing data.
7. **Prevent input errors and allow cancellation:** Clearly indicate required fields, units, ranges, save results, and audit records.

## 3. Brand and Logo Policy

### 3.1 Brand Definition

| Item | Definition |
| --- | --- |
| Official name | Smart Shrimp Pond Management System |
| Display short name | Smart Shrimp Pond |
| Brand personality | Supportive of field operations, trustworthy, enables fast decisions, clear data |
| Primary environments | Farms Manager PC, Technical Manager PC / tablet |
| Brand colors | Water Blue, Mangrove Green |

The brand expression should be a calm business system suitable for both aquaculture operations and management decisions, rather than having a decorative or game-like atmosphere.

### 3.2 Logo Concept

The logo should combine a symbol and wordmark that abstract a **shrimp silhouette, a water-surface line, and a sensor-data waveform**. Thin lines and overly complex details should be avoided so that the logo remains recognizable at small sizes.

- Symbol only: Use for app icons, favicons, and narrow sidebars
- Symbol + wordmark: Use on the login screen, header, and exported reports
- Monochrome version: Use for printing, complex backgrounds, and situations requiring sufficient contrast
- Prohibited: Changing aspect ratio, excessive shadows / gradients, placement on low-contrast backgrounds, changing letter spacing
- Safe area: Leave at least 25% of the symbol's height around the outer edge of the symbol
- Asset format: SVG as the primary format; PNG or ICO for favicon use

The logo image will be produced after the brand materials are finalized. In the frontend, the display area should be fixed so that replacing the logo does not break the layout.

## 4. Design Tokens

### 4.1 Colors

The following are initial candidates and will be finalized after brand confirmation and contrast verification. Brand colors and semantic status colors must be kept separate.

| Usage | Color Name | Candidate Value | Usage Rule |
| --- | --- | --- | --- |
| Primary | Water Blue | `#087EA4` | Primary buttons, selected states, links, in-progress displays |
| Primary Dark | Deep Water | `#075985` | Header, dark buttons, backgrounds for white text |
| Secondary | Mangrove Green | `#2F855A` | Secondary buttons, auxiliary representations of growth, supply, and actuals |
| Background | Mist | `#F4F8F7` | Overall application background |
| Surface | White | `#FFFFFF` | Panels, tables, and form backgrounds |
| Text | Ink | `#18323B` | Body text, primary numbers, headings |
| Muted Text | Slate | `#5B7078` | Supplementary information, units, update times |
| Border | Tide Line | `#D6E3E5` | Borders and separators |
| Normal | Status Green | `#237A57` | Normal, stable, complete |
| Attention | Status Amber | `#A15C00` | Attention, confirmation recommended |
| Warning | Status Orange | `#B54708` | Warning, action required |
| Critical | Status Red | `#B42318` | Critical, immediate action required |
| Info | Status Blue | `#1769AA` | Information, reference, system notification |

- Status colors must always be accompanied by a status label and icon.
- Text-to-background contrast must meet WCAG 2.2 AA-level requirements.
- Avoid relying only on red-versus-green comparisons; distinguish states through shapes, labels, patterns, and position as well.
- Do not reuse the Primary color to represent warning or normal status.

### 4.2 Typography and Numbers

- Japanese: `Noto Sans JP`; English and Indonesian: `Inter` or an equivalent highly readable web font as candidates
- Manage font sizes using fixed tokens; do not scale text based on screen width
- Body text: 16px as the baseline; supplementary information: at least 14px; screen titles: 24px or smaller as the baseline
- Use digit grouping for numbers and display units immediately after values or in column headings
- Add acquisition timestamps to sensor values. Example: `DO 5.6 mg/L · 09:15 WIB`
- Explicitly label forecast values as "Forecast", actual values as "Actual", and estimated values as "Estimated"

### 4.3 Spacing and Shapes

- Use 4px as the base unit, with major spacing based on 8px, 16px, 24px, and 32px
- Limit corner radii for business panels to 4px or 8px
- Do not use oversized decorative rounded cards, multiple floating cards, or excessive shadows
- Clickable elements must have a visual state that makes their operability clear

## 5. Common Layout

### 5.1 Application Shell

```text
┌──────────────────────────────────────────────────────────────┐
│ Logo │ Farm / Estate Switch │ Last Updated │ Alerts │ Language │ User │
├──────────────┬───────────────────────────────────────────────┤
│ Side Nav     │ Breadcrumbs                                    │
│              ├───────────────────────────────────────────────┤
│              │ Conclusion / Page Title                         │
│              │ Filters / Period / Actions                      │
│              │ Main Content                                    │
└──────────────┴───────────────────────────────────────────────┘
```

- Fix the header at the top of the screen and make the current Role, target Farm, and latest update time visible.
- Switch side-navigation items according to Role; do not display screens for which the user has no permission.
- Breadcrumbs show the Estate > Farm > Pond hierarchy and clarify the return path.
- The top of the page should follow the order: "Conclusion or most important status" → "Filters" → "Details".
- Preserve Farm and Pond selection states after screen transitions to prevent incorrect operations.

### 5.2 Responsive Policy

| Display Width | Target | Policy |
| --- | --- | --- |
| 1280px or more | Farms Manager PC | Display 3–4 KPI columns, comparison tables, and detailed panels |
| 1024–1279px | Small PC / tablet landscape | Primarily 2 columns; convert detailed panels into drawers |
| 768–1023px | Technical Manager tablet | Stack Pond list and details vertically; keep operation buttons fixed |
| 767px or less | Future support | Prioritize essential confirmation and input; simplify complex comparisons |

Horizontal scrolling should be limited to necessary areas such as tables and should not occur across the entire page. Provide touch targets of at least 44px for primary touch operations.

## 6. Role-based Information Design

### 6.1 Farms Manager

Prioritize information in the following order to enable company-wide decisions in a short time.

1. Conclusion: Overall status and highest-priority action items
2. Status summary: Total Ponds, breakdown of Normal / Attention / Warning, latest update time
3. Three analysis areas: Revenue optimization, biological risk, future inventory
4. Alerts: Severity, Farm, Pond, occurrence time, recommended confirmation item
5. Evidence: Aggregated KPIs, analysis period, main factors, recommended actions

Do not display raw sensor values to the Farms Manager. The S-06 drill-down may display only aggregated KPIs, scores, trend summaries, analytical evidence, recommended actions, and data update time.

### 6.2 Technical Manager

Prioritize information in the following order to enable rapid field response.

1. Current abnormalities: Unacknowledged / critical Alerts, communication outages, sensor missing data
2. Pond list: Status, latest values, update time, and selection state for each Pond
3. Selected Pond details: Time series of DO, pH, water temperature, turbidity, TDS, and water level
4. Response support: AI Recommendation, thresholds, recommended actions
5. Business operations: Daily reports, weekly sampling, monitoring / approval of automatic control, and Override

The Technical Manager can view detailed IoT values, daily / weekly data, and detailed Alerts within assigned Farms.

## 7. Common Components

| Component | Display / Interaction Rules |
| --- | --- |
| KPI Card | Display value, unit, target period, Actual / Forecast / Estimated, comparison with previous value, and update time |
| Status Badge | Use label, icon, and status color together. Use Normal / Attention / Warning / Critical / Info |
| Table | Provide column headings, sorting, filtering, pagination, empty state, and update time |
| Time-series Chart | Display period, unit, thresholds, Actual / Forecast legend, and missing-data intervals |
| Alert Row | Display severity, target, occurrence time, status, recommended action, and detail navigation |
| Form | Clearly indicate required fields, units, input ranges, errors, saving state, and save completion |
| Confirmation Dialog | Display target of the operation, impact, Safety Layer constraints, and Execute / Cancel |
| Toast | Limit to short-lived notifications such as save and communication results; keep important information on the screen |
| Drawer | Use for detailed confirmation while maintaining list context. Do not use for complex input |

## 8. Status Display and Data Quality

### 8.1 Common Screen States

| State | Display Content |
| --- | --- |
| Loading | Show a loading indicator in the relevant area and prevent previous values from being mistaken for new values |
| Empty | Display why no data exists and possible next actions |
| Error | Display an overview of the retrieval failure, retry option, occurrence time, and contact information |
| Offline | Display communication outage, last successfully retrieved time, and operation restrictions |
| Stale | Add "Update Delayed" to data exceeding the allowed age and distinguish it from current values |
| Sensor Missing | Display the affected sensor, missing period, alternative information if available, and point of contact |
| Forbidden | Display the reason for insufficient permission and the accessible scope without allowing the user to infer hidden values |

### 8.2 Alert Status

Alerts have the following states, and the person and time of each state change must be recorded.

1. Unacknowledged: Occurred but has not been confirmed by the responsible person
2. Acknowledged: The responsible person has confirmed the contents
3. In Progress: Field response or monitoring has started
4. Resolved: The condition has been resolved and the response is complete

The detail screen displays what happened, current value, reference value, trend, occurrence time, recommended action, confirmer, and response history.

## 9. AI Recommendation and Automatic Control UI

### 9.1 AI Recommendation

AI Recommendation should be displayed as one coherent unit containing the following information:

- Recommendation
- Target Pond and forecast period
- Main supporting data and target period
- Confidence or certainty, when provided by the model
- Clear distinction between Recommendation / Forecast / Actual
- Record of whether the Recommendation was adopted, rejected, or put on hold
- Responsibility statement: "Final decision is made by the Technical Manager"

AI does not directly operate Actuators. Even when proceeding from a Recommendation to a control operation, confirmation by the Technical Manager and permission from the Safety Layer are required.

### 9.2 Automatic Control

S-14 displays the following for each device:

- Actuator name, current status, control mode, automatic / manual
- Permission status from the Safety Layer
- Most recent control reason, execution time, and execution result
- Approval, Override, and emergency-stop operation status
- User who executed the operation, execution time, and audit log

Display a confirmation dialog for critical operations and clearly show the target equipment, expected impact, and release conditions. Operations outside the permitted scope of the Safety Layer must be disabled and the reason for disabling them must be displayed.

## 10. Screen-specific UI Specifications

| Screen ID | Initial Display | Main Operations | Main Abnormalities / Restrictions |
| --- | --- | --- | --- |
| S-01 | Logo, language, login form | Login, language selection, retry after error | Authentication failure, disabled account, communication error |
| S-02 | Conclusion, status summary, three analysis areas, Alerts | Farm filtering, period change, navigate to details, export | Data delay, excluded aggregation targets, insufficient permission |
| S-03 | Recommended harvest date, profit forecast, point-in-time comparison | Select Pond / period, compare scenarios, confirm evidence | Insufficient forecast data, prevent mixing forecast and actual |
| S-04 | Risk distribution, Pond scores, main factors | Filter by severity / Farm / Pond, confirm response | Score cannot be calculated, not evaluated, stale data |
| S-05 | Supply forecast by week / size | Change period / size, confirm confidence, export | No supply forecast, outside forecast period |
| S-06 | Aggregated KPIs, analytical evidence, factors, recommendations | Return to S-02–S-05, change evidence period | Raw sensor values are not displayed |
| S-07 | Pond list and selected Pond IoT details | Switch Pond, change period, display thresholds, navigate to reports | Communication outage, sensor missing data, update delay |
| S-08 | Unacknowledged Alerts and details | Change to Acknowledged / In Progress / Resolved | Farm outside permission, duplicate Alerts, re-notification |
| S-09 | Thresholds by item and scope | Edit thresholds, save, view change history | Permission not finalized, out-of-range values, awaiting approval |
| S-10 | Forecast, Recommendation, evidence, confidence | Adopt, reject, hold, record response | Insufficient training data, confidence not calculated |
| S-11 | Daily report form | Input, save draft, confirm, view history | Required fields missing, out-of-range values, duplicate report |
| S-12 | Weekly sampling form | Input, save draft, confirm, view KPIs | Inconsistency between count and weight, editing after confirmation |
| S-13 | Daily / weekly report history | Search by period / Pond, view details, export | No data, outside permission, unconfirmed report |
| S-14 | Device status, control history, operations requiring approval | Approve, Override, emergency stop, view history | Safety Layer rejection, communication outage, prevention of duplicate operations |
| S-15 | User / Role list | Register, change Role, disable, view history | Outside current UI scope, Administrator Role not finalized |

## 11. Multilingual Support, Date/Time, and Units

- Indonesian is mandatory for the primary Technical Manager screens.
- Indonesian is the default for Farms Manager, with English available as a switchable language. Untranslated strings must be displayed in an identifiable state rather than as meaningless blanks.
- Language selection is retained per user and applied to all screens after login.
- Dates and times are generally displayed in Indonesia Western Time (WIB, UTC+7), with the time zone shown on the screen.
- Dates use `DD MMM YYYY` and times use `HH:mm` as the baseline. API and export formats will be separately defined in frontend design.
- Water-quality units follow the data definition: DO `mg/L`, water temperature `°C`, turbidity `NTU`, and water level `cm`.

## 12. Accessibility

- Do not rely on color alone; represent states using text, icons, and shapes.
- Define focus order and focus indicators so that primary operations can be completed using only a keyboard.
- Provide readable labels for input fields and associate errors with the relevant fields.
- Provide legends and table-format alternatives for charts.
- Primary touch-operation areas must be at least 44px.
- Limit animated displays to what is necessary for understanding information, and consider a design that allows users to suppress motion through user settings.

## 13. Open Issues and Handover to Subsequent Design

The following items remain pending as UI design assumptions and will be updated after business rules or permission design are finalized.

| Issue | Current Treatment |
| --- | --- |
| System Administrator Role | Official Role name and permissions for S-09 and S-15 are not finalized |
| Field Operator | Outside the current UI scope. Field-operation screens will be defined separately |
| Threshold Configuration | It is not yet determined whether the Technical Manager changes thresholds directly or uses a request / approval process |
| Notification Channels | The scope of Email, SMS, LINE, Push, and other channels in addition to web notifications is not finalized |
| AI Confidence | Definition and display method of confidence according to the model will be finalized in Data/ML design |
| Automatic Control Approval | Controlled equipment, approvers, and emergency-stop permissions will be finalized in the Safety Layer design |
| Profit / Supply Volume | Calculation definitions and safety margins will be reflected after business rules are finalized |
