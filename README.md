# Contact Center Legal Risk & Escalation Governance

## Overview

This project is a high-fidelity **Legal Risk & Escalation Governance System** designed for a financial services contact center.

It demonstrates how raw operational data can be transformed into a structured, audit-ready governance framework used to:

* Detect high-risk customer interactions (Legal, Vulnerability, Abuse)
* Monitor escalation control failures in real time
* Evaluate agent compliance and process dumping behaviors
* Support regulatory oversight (CFPB, BBB, FTC)
* Enable targeted coaching and internal audit workflows

**Note on Data Privacy**

Because banking and regulatory data is highly sensitive, this project uses a custom-built Python simulation engine to generate a synthetic dataset of 5,000 records.
The data reflects realistic operational patterns, agent tenure–based error rates, and legal phrase taxonomies without compromising confidentiality.

---

## Business Problem

In highly regulated environments, failures in escalation especially involving litigation threats and consumer rights, can lead to:

* Regulatory penalties (CFPB / OCC fines)
* Legal exposure (unaddressed lawsuits)
* Customer harm (failure to identify vulnerable customers)
* Reputational damage

Most operational dashboards focus on efficiency (e.g., Average Handle Time).

This project focuses on **control effectiveness, compliance outcomes, and governance risk.**

---

## Technical Challenges & DAX Engineering

### The Denominator Mismatch Problem

Early versions of the dashboard experienced “metric spill,” where rates exceeded 100%.

This occurred due to:

* Numerators and denominators pulling from different filter contexts
* Decomposition Tree cross-filtering
* The “Accuracy Paradox” caused by dominant low-risk interactions

---

### The Defensible Solution

A nested DAX framework was implemented using:

* VAR logic
* KEEPFILTERS
* Centralized risk universe definitions

This locks all governance metrics to validated risk interactions.

#### Example: Risk Compliance Gap Logic

```dax
Risk Compliance Gap % =
VAR RiskFailures =
CALCULATE(
[Total Risk Interactions],
KEEPFILTERS('Interactions'[escalation_validity] IN {"Missed_Escalation","Failed_Escalation"})
)
RETURN
DIVIDE(RiskFailures, [Total Risk Interactions], 0)
```

This guarantees that compliance gaps remain mathematically valid.

---

## Dashboard Architecture

The Power BI solution is organized into three functional layers.

---

### 1. Governance Overview

Executive-level monitoring of organizational risk exposure.

Key Features:

* Risk and legal exposure distributions
* Primary Risk Drivers (Decomposition Tree)
* Compliance and accuracy trends
* Product risk analysis
* Threshold-based KPI alerts

Bookmark Views:

* All Risk / Legal Focus
* Call / Chat channels

---

### 2. Compliance Audit

Audit-ready interaction review for quality assurance and investigation.

Key Features:

* Row-level interaction logs
* Conditional formatting for failures
* Hangup detection on legal calls
* Legal phrase evidence tracking
* Transfer validation

---

### 3. Agent Compliance

Performance and coaching analytics for frontline staff.

Key Features:

* Agent risk handling breakdown
* Compliance defect trends
* Governance accuracy scoring
* Process dumping detection
* Coaching prioritization

---

## Dataset Design & Risk Taxonomy

The dataset was engineered in Python to simulate worst-case regulatory risk scenarios.

### Risk Categories

| Category      | Examples                                        |
| ------------- | ----------------------------------------------- |
| Legal         | Implied threats, regulatory filings, litigation |
| Vulnerability | Financial hardship, medical distress, eviction  |
| Abuse         | Harassment, hostility, threats                  |

---

### Governance Rules

Each interaction is classified into one of the following outcomes:

* Valid_Escalation — Correct identification and routing
* Missed_Escalation — Risk present, no action taken (highest risk)
* Failed_Escalation — Escalation attempted but ineffective
* Invalid_Disposition — Process dumping behavior
* No_Escalation_Required — Normal interaction
* Review_Needed — Manual review required

These classifications power all compliance metrics.

---

## Key Metrics

### System-Level

* Risk Compliance Gap %
* Operational Accuracy
* Governance Accuracy
* Risk Volume %

### Legal Risk

* Legal Compliance Gap
* Legal Resolution Rate
* Legal Mention Rate
* Litigation Severity Rate
* Regulatory Mention Rate

### Agent Performance

* Agent Compliance Score
* Missed Escalation Rate
* Failed Escalation Rate
* Invalid Disposition Rate
* Agent Hangups
* Average Handle Time

---

## Screenshots

### Governance Overview — All Risk

![Governance Overview - All Risk](screenshot/governance_overview_all_risk.png)

---

### Governance Overview — Legal Focus

![Governance Overview - Legal Focus](screenshot/governance_overview_legal_focus.png)

---

### Compliance Audit

![Compliance Audit](screenshot/compliance_audit.png)

---

### Agent Compliance

![Agent Compliance](screenshot/agent_compliance.png)

---

## Technology Stack

* Python (Pandas, NumPy, Faker)
* Power BI
* DAX
* GitHub

---

## Repository Structure

```text
contact-center-risk-governance/
│
├── data/              # Raw synthetic datasets
├── dashboard/        # Power BI (.pbix) files
├── docs/              # Technical documentation (DAX, Dictionary, Logic)
├── screenshot/        # Dashboard images for review
└── README.md          # Project overview
```
---

## How to Use This Repository

1. View the Dashboard
   Download the `.pbix` file from the `dashboard/` folder.

2. Review the Logic
   See `docs/` for DAX measures and governance rules.

3. Explore the Data
   The Python generator script is included in the repository.

---

## Limitations

* Data is fully synthetic
* No live regulatory systems are connected
* Legal references are simulated
* Metrics are designed for demonstration purposes

These constraints are intentional to preserve confidentiality.

---

## Future Enhancements

* Time-series policy change simulation
* Agent learning curves
* Advanced data quality failures
* Audit sampling models
* Economic impact modeling
* Case lifecycle tracking

---

## Author

Kristine Soliman <br>
Data & Operations Analyst | Chandler, AZ  
