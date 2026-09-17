# User Role and Permission Definition

**Role-Based Access Control / RBAC**

## 1. Purpose

Define the user Roles and basic permission scope in the Smart Shrimp Pond Management System.

This document does not define the following:

- Business processes
- Screen structure
- UI/UX
- Detailed KPI and data items
- AI analysis and prediction logic
- Internal system implementation

These items will be defined in subsequent documents.

---

## 2. User Roles

The following three Roles are defined as business users of the system.

| Role | Responsibility |
| --- | --- |
| **Farms Manager** | Oversees multiple Farms and makes overall business and production decisions for the aquaculture operation |
| **Technical Manager** | Manages the condition of assigned Farms and Ponds and makes technical and operational decisions |
| **Field Operator** | Performs required field operations at Farms and Ponds |

---

## 3. Permissions

User permissions consist of the following four types.

| Permission | Meaning |
| --- | --- |
| **View** | View information |
| **Acknowledge** | Confirm an Alert / Notification |
| **Decide** | Make a business or operational decision |
| **Execute** | Execute an authorized operation or task |

---

## 4. Role × Permission

| Target | Farms Manager | Technical Manager | Field Operator |
| --- | --- | --- | --- |
| Company information | View / Decide | - | - |
| Farm information | View / Decide | View | - |
| Pond information | View | View / Decide | - |
| IoT information | View (aggregated values and analysis results only) | View (detailed values for assigned Farms) | - |
| Alert / Notification | Acknowledge | View / Acknowledge / Decide | - |
| AI Recommendation | View / Decide | View / Decide | - |
| Production / Harvest information | View / Decide | View / Decide | - |
| Inventory / Supply information | View / Decide | View | - |
| Field operations | - | Decide | Execute |
| Actuator operation | - | Execute* | - |

\* Actuator operation by the Technical Manager is limited to the scope permitted by the Safety Layer and other safety mechanisms.

---

## 5. Data Access Scope

### Farms Manager

- Can view company-wide information
- Can view information for multiple Farms under their responsibility
- Can access aggregated KPIs, analytical evidence, and trend summaries from Farm to Pond level
- Cannot view raw sensor values or detailed field input values
- Does not perform field operations on Ponds

### Technical Manager

- Can view and manage information for assigned Farms
- Can view information for Ponds within assigned Farms
- Can view detailed IoT values, daily and weekly inputs, and alert details for Ponds within assigned Farms
- Makes technical and operational decisions for Ponds
- Can operate Actuators within the authorized scope

### Field Operator

- Performs assigned field operations
- Does not generally require access to business management or analytical information

---

## 6. Permissions Related to AI

The AI provides **Recommendations that support decision-making**.

The AI does not have permission to directly operate Actuators.

The basic responsibility allocation is as follows:

```text
AI
  ↓
Recommendation
  ↓
Human Decision
  ↓
Technical Manager
  ↓
Safety Layer / PID
  ↓
Actuator
```

Therefore:

- AI: Recommendation
- Farms Manager: Business and production Decision
- Technical Manager: Technical and operational Decision / authorized Execute
- Field Operator: Field-operation Execute

---

## 7. Permission Design Principles

1. Follow the principle of **least privilege**
2. Provide only the information and operations required for each Role
3. Separate information-viewing permissions from operation permissions
4. Separate AI Recommendations from human Decisions
5. Place Actuator operations under the control of safety mechanisms
6. Do not define detailed business processes, screens, or UI/UX in this document

---

## 8. Items to Be Determined

The following items will be determined in subsequent requirements definition.

- Final approval scope of the Farms Manager
- Scope of Actuators that the Technical Manager may operate
- Specific permissions for Human Override
- System functions to be provided to the Field Operator
- Farm / Pond assignment rules
- How Decisions regarding AI Recommendations will be recorded
