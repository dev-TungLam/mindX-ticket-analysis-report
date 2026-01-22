# Week 4 Analysis Report: Helpdesk Ticket Patterns & System Health

## 1. Executive Summary

**Finding**: The Technical Support team is in a "fire-fighting" state (**62% High/Urgent** tickets).
**Root Cause**: **40%** of volume stems from **CRM/LMS Synchronization failures** (Enrollment & Payments).
**Recommendation**: Automate the enrollment fix process immediately, then refactor the integration.

---

## 2. Methodology

To arrive at these conclusions, we implemented the following analytical framework in Odoo:

1. **Data Cleaning**: Normalized a raw dataset of 131 tickets, merging fragmented tags (e.g., "CRM", "LMS") and standardizing priority levels.
2. **Odoo Analytics**: Built a custom Odoo module (`helpdesk_analytics`) to visualize ticket distribution by Stage, Team, and Tags.
3. **Pattern Recognition**: Utilized Pivot Tables to correlate "System Tags" (CRM/LMS) with "Issue Types" (Enrollment/Payment).

---

## 3. Recurring Patterns & Findings

Week 4 Odoo Tickets Analysis Charts: 

https://devlam-onboardingmindx.odoo.com/dashboard/share/4/5f8e1c9d-39f0-475e-a8dc-af1d783436f2

### A. The "Urgency Trap" (Workload Analysis)

The team is operating in crisis mode, leaving little capacity for proactive work.

| Priority Class          | Ticket Count |      Share      | Status               |
| :---------------------- | :----------: | :-------------: | :------------------- |
| **Urgent / High** | **82** | **62.5%** | 🔴**Critical** |
| Normal / Low            |      49      |      37.5%      | 🟢 Stable            |

![Priority Analysis](./priority%20chart.png)

**Implication**: The "High" priority flag is heavily overused, drowning out genuine emergencies (e.g., System Down).

### B. Top System Failures (Category Analysis)

Three breakdown areas drive **41%** of all reported volume:

| Top System       |     Share     | Primary Issue Context                       |
| :--------------- | :-----------: | :------------------------------------------ |
| **1. CRM** | **19%** | Sales Data & Customer Info mismatch         |
| **2. LMS** | **17%** | Student Access & Course Enrollment failures |
| **3. TMS** | **7%** | Transport & Logistics scheduling errors     |

![Tags Analysis](./Tag%20chart.png)

### C. Specific Recurring Incidents

| Issue Category                 | Tags                            | Impact                                                                                         |
| :----------------------------- | :------------------------------ | :--------------------------------------------------------------------------------------------- |
| **1. Sync Failures**     | `enroll HV` (4)               | **Critical**: Paid students cannot learn. Triggers immediate "Urgent" tickets.           |
| **2. Manual Data Fixes** | `Fix giá`, `Xóa payment`  | **High Volume**: Ops team lacks tools to correct entry errors, forcing Dev intervention. |
| **3. Provisioning**      | `Các vấn đề về mail` (5) | **Toil**: Manual email creation requests that should be fully automated.                 |

---

## 4. Impact Analysis (Business Risk)

1. **Student Experience**: The high volume of "Enrollment" and "Email" issues directly blocks students from learning *after* they have paid. This is a churn risk.
2. **Developer Burnout**: Managing "Data Fixes" (e.g., *Xóa payment*) distracts engineers from building features.
3. **SLA Breaches**: With 62% of tickets marked urgent, genuine emergencies (e.g., System Down) risk getting lost in the noise.

---

## 5. Action Plan & Recommendations

### Phase 1: The "Self-Correction" Tool

* **Problem**: Ops cannot fix failed enrollments without a Dev.
* **Solution**: Build a **"Retry Enrollment" Button** in Odoo.
  * *Mechanism*: Server Action that re-triggers the sync webhook for a specific student/order.
  * *Result*: Shifts ~25 tickets/month from "Technical Support" to "Ops Self-Service".

### Phase 2: Systemic Resolution

* **Action 2.1**: **Automated Reconciliation Job (Cron)**.
  * *Mechanism*: A nightly script that compares "Paid Students" in CRM vs "Enrolled Students" in LMS. Any discrepancy is auto-fixed (Self-Healing).
  * *Result*: Guarantees data consistency even if the real-time sync fails.
* **Action 2.2**: **Integration Hardening (Queue)**.
  * *Detail*: Upgrade the webhook to an idempotent message queue to handle traffic spikes.
* **Action 2.3**: **Strict Validation UI**.
  * **Tech Stack**: Odoo Server Action $\to$ **Node.js API Endpoint**.
  * **Execution**:
    1. Create an Odoo Server Action that sends a POST request to the Node.js middleware webhook.
    2. **Node.js Service**: The service receives the order payload, validates the data structure using `zod`, and re-triggers the enrollment flow in the LMS.
  * *Result*: Decouples logic from Odoo. Ops can retry safely via the main backend logic.

### Phase 3: Prevention UI

* **Problem**: Users enter wrong prices, then ask Devs to "Fix giá".
* **Solution**: Add **Validation Constraint** on Order Creation.
  * *Result*: Blocks invalid data entry at the source.
