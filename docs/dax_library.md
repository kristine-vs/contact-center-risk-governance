
## Overview

This document defines the full DAX architecture used in the **Contact Center Legal Risk & Escalation Governance System**.

The measures are intentionally structured around:

- A centralized risk universe
- Filter-context control using `CALCULATE` and `KEEPFILTERS`
- Defensive denominator logic
- Clear separation between governance accuracy and operational accuracy

The model is designed to prevent metric distortion caused by cross-filtering and denominator drift.

---

## 1. Core Volumetric Measures
*Basic counts used as denominators for high-level ratios.*

**Total Interactions**
`Total Interactions = COUNTROWS('Interactions')`

**Total Risk Interactions**
`Total Risk Interactions = CALCULATE(COUNTROWS('Interactions'), 'Interactions'[escalation_validity] IN {"Valid_Escalation", "Failed_Escalation", "Missed_Escalation", "Review_Needed"})`

**Total Legal Mentions**
`Total Legal Mentions = CALCULATE([Total Interactions], 'Interactions'[Is_Risk] = TRUE(), 'Interactions'[Risk_Type] = "Legal")`

**Records Under Review**
`Records Under Review = COUNTROWS(Interactions)`

**Count Valid Escalations**
`Count Valid Escalations = 
COALESCE(
    CALCULATE(
        COUNTROWS('Interactions'),
        'Interactions'[escalation_validity] = "Valid_Escalation"
    ),
    0
)`

**Count Failed Escalations**
`Count Failed Escalations =
COALESCE(
    CALCULATE(
        COUNTROWS('Interactions'),
        'Interactions'[escalation_validity] = "Failed_Escalation"
    ),
    0
)`

**Count Missed Escalations**
`Count Missed Escalations =
COALESCE(
    CALCULATE(
        COUNTROWS('Interactions'),
        'Interactions'[escalation_validity] = "Missed_Escalation"
    ),
    0
)`

**Count Invalid Dispositions**
`Count Invalid Dispositions =
COALESCE(
    CALCULATE(
        COUNTROWS('Interactions'),
        'Interactions'[escalation_validity] = "Invalid_Disposition"
    ),
    0
)`


---

## 2. Governance & Compliance (Gap Metrics)
*Nested measures using KEEPFILTERS to maintain 100% mathematical validity.*

**Critical Escalation Failures**
`Critical Escalation Failures = CALCULATE(COUNTROWS('Interactions'), KEEPFILTERS('Interactions'[escalation_validity] IN {"Missed_Escalation", "Failed_Escalation"}), 'Interactions'[Is_Risk] = TRUE())`

**Risk Compliance Gap %**
`Risk Compliance Gap % = DIVIDE([Critical Escalation Failures], [Total Risk Interactions], 0)`

**Risk Resolution Rate**
`Risk Resolution Rate = VAR ValidCount = CALCULATE(COUNTROWS('Interactions'), KEEPFILTERS('Interactions'[escalation_validity] = "Valid_Escalation"), 'Interactions'[Is_Risk] = TRUE()) RETURN DIVIDE(ValidCount, [Total Risk Interactions], 0)`

**Legal Compliance Gap**
`Legal Compliance Gap = VAR Legal_Failures = CALCULATE(COUNTROWS('Interactions'), KEEPFILTERS('Interactions'[escalation_validity] IN {"Missed_Escalation", "Failed_Escalation"}), 'Interactions'[Is_Risk] = TRUE(), 'Interactions'[Risk_Type] = "Legal") RETURN DIVIDE(Legal_Failures, [Total Legal Mentions], 0)`

**Legal Resolution Rate**
`Legal Resolution Rate = VAR Legal_Success = CALCULATE(COUNTROWS('Interactions'), KEEPFILTERS('Interactions'[escalation_validity] = "Valid_Escalation"), 'Interactions'[Is_Risk] = TRUE(), 'Interactions'[Risk_Type] = "Legal") RETURN DIVIDE(Legal_Success, [Total Legal Mentions], 0)`

---

## 3. Accuracy & Status Logic
*Measures determining the qualitative health of the operation.*

**Governance Accuracy**
`Governance Accuracy = 
VAR RiskTotal = [Total Risk Interactions]
VAR ValidCount = COALESCE([Count Valid Escalations], 0)
RETURN
IF(
    RiskTotal = 0,
    BLANK(),
    DIVIDE(ValidCount, RiskTotal)
)`

**Operational Accuracy**
`Operational Accuracy = 
VAR TotalInt = [Total Interactions]
VAR Correct_Decisions =
    COALESCE([Count Valid Escalations], 0) +
    CALCULATE(
        [Total Interactions],
        'Interactions'[escalation_validity] = "No_Escalation_Required"
    )
RETURN
IF(
    TotalInt = 0,
    BLANK(),
    DIVIDE(Correct_Decisions, TotalInt)
)`

**Governance Accuracy Status**
`Governance Accuracy Status = 
VAR RiskTotal = [Total Risk Interactions]
VAR Acc = [Governance Accuracy]
RETURN
SWITCH(
    TRUE(),
    RiskTotal = 0, "Zero Risk",        
    Acc >= 0.95, "Optimal",
    Acc >= 0.90, "Moderate Risk",
    Acc >= 0.85, "High Risk",
    "Critical"
)`

**Operational Accuracy Status**
`Operational Accuracy Status = 
VAR CurrentAccuracy = [Operational Accuracy]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(CurrentAccuracy), "No Data",
    CurrentAccuracy >= 0.95, "Optimal",
    CurrentAccuracy >= 0.90, "Moderate Risk",
    CurrentAccuracy >= 0.85, "High Risk",
    "Critical"
)`

---

## 4. Agent Performance Metrics
*Individual level KPIs for coaching and tenure-based analysis.*

**Agent Compliance Score**
`Agent Compliance Score = 1 - [Risk Compliance Gap %]`

**Agent Failed Rate**
`Agent Failed Rate = DIVIDE([Count Failed Escalations], [Total Risk Interactions], 0)`

**Agent Missed Rate**
`Agent Missed Rate = DIVIDE([Count Missed Escalations], [Total Risk Interactions], 0)`

**Agent Valid Escalation Rate**
`Agent Valid Escalation Rate = DIVIDE([Count Valid Escalations], [Total Risk Interactions], 0)`

**Process Dumping Rate**
`Process Dumping Rate = DIVIDE([Count Invalid Dispositions], [Total Interactions], 0)`

**Agent Hangups**
`Agent Hangups = CALCULATE(COUNTROWS('Interactions'), 'Interactions'[disconnected_by] = "Agent")`

**Avg Handle Time (Min)**
`Avg Handle Time (Min) = DIVIDE(AVERAGEX(FILTER('Interactions', 'Interactions'[duration_sec] > 0), 'Interactions'[duration_sec]), 60, 0)`

---

## 5. Metadata Columns & Sorting
*Calculated columns used for slicing and dicing the data.*

**Is_Risk**
`Is_Risk = IF('Interactions'[Risk_Type] <> "None", TRUE(), FALSE())`

**Risk_Type**
`Risk_Type = SWITCH(TRUE(), 'Interactions'[Legal_Category] <> "None", "Legal", CONTAINSSTRING('Interactions'[scenario_label], "Risk_Vulnerability"), "Vulnerability", CONTAINSSTRING('Interactions'[scenario_label], "Risk_Abuse"), "Abuse", "None")`

**Legal_Category**
`Legal_Category = SWITCH(TRUE(), CONTAINSSTRING('Interactions'[scenario_label], "Legal_Litigation"), "Litigation", CONTAINSSTRING('Interactions'[scenario_label], "Legal_Explicit"), "Explicit Regulatory", CONTAINSSTRING('Interactions'[scenario_label], "Legal_Inexplicit"), "Inexplicit", CONTAINSSTRING('Interactions'[scenario_label], "Legal_Implied"), "Implied", "None")`

**Tenure_Band**
`Tenure_Band = IF('Interactions'[agent_tenure_months] < 6, "< 6 Months", "6+ Months")`

---

## Author

Kristine Soliman  
Data & Operations Analyst | Chandler, AZ

