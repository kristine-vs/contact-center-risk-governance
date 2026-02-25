## Data Dictionary

This dictionary defines the 20-field schema used for this project. It documents the relationship between raw operational data and the final audit outcomes.

---

## 1. Interaction & Agent Identification

Core identifiers for record tracking and workforce attribution.

### interaction_id
**Type:** String  
**Description:** Unique identifier for each call or chat session.  
**Role:** Primary Key.

### date
**Type:** Date  
**Description:** The calendar date the interaction occurred.  
**Role:** Enables time-series trend analysis and compliance aging.

### agent_id
**Type:** String  
**Description:** Unique ID for the employee handling the record.  
**Role:** Facilitates agent-level performance auditing and scorecards.

### agent_tenure_months
**Type:** Integer  
**Description:** Total months of service for the agent.  
**Role:** Used to correlate training/experience levels with escalation failure probabilities.

---

## 2. Customer & Product Context

Demographic and account-level data used to identify high-value or high-risk segments.

### customer_id
**Type:** String  
**Description:** Unique identifier for the customer.  
**Role:** Used to track "Frequent Filers" or recurring legal threats.

### product_type
**Type:** String  
**Description:** The specific financial product (e.g., Everyday Blue, Travel Platinum).  
**Role:** Identifies product-specific regulatory risks.

### customer_tier
**Type:** String  
**Description:** The service level of the customer (e.g., Preferred, Private Wealth).  
**Role:** Analyzes if risk handling varies across different customer "values."

### tenure_months
**Type:** Integer  
**Description:** The length of the customer's relationship with the bank.  
**Role:** Context for sentiment and loyalty analysis.

---

## 3. Operational Performance Attributes

Data points reflecting the mechanics of the interaction and agent efficiency.

### queue
**Type:** String  
**Description:** The support department (e.g., Collections, Fraud, Tech Support).  
**Role:** Segment-level reporting on departmental risk exposure.

### interaction_type
**Type:** String  
**Description:** The channel used for communication (Call vs. Chat).  
**Role:** Compares compliance effectiveness across different media.

### duration_sec
**Type:** Integer  
**Description:** Total length of the interaction in seconds.  
**Role:** Base for Average Handle Time (AHT) calculation.

### sentiment_score
**Type:** Float  
**Description:** NLP-generated score ranging from -1.0 (Very Negative) to 1.0 (Very Positive).  
**Role:** Used to prioritize audits for interactions with high friction.

### disconnected_by
**Type:** String  
**Description:** Identifies who ended the session (Agent, Customer, or System).  
**Role:** Critical for detecting "Malicious Hangups" during legal escalations.

---

## 4. Governance & Audit Indicators

The primary drivers for the governance framework and compliance metrics.

### agent_disposition
**Type:** String  
**Description:** The final status code the agent applied to the record.  
**Role:** Used to verify if the agent's documented intent matches the "Ground Truth" risk.

### transferred_flag
**Type:** String (Y/N)  
**Description:** Indicates if the call was physically routed to another department.  
**Role:** Action verification for escalation SOP.

### department_transferred
**Type:** String  
**Description:** The target destination of the transfer (e.g., Trust & Safety, Supervisor).  
**Role:** Verifies "Wrong Route" failures vs. correct escalation paths.

### legal_mention
**Type:** String  
**Description:** The specific customer phrase detected (e.g., "I am calling the CFPB").  
**Role:** Raw evidence for Regulatory/Legal risk auditing.

### high_risk_phrase
**Type:** String  
**Description:** Phrases indicating Vulnerability (e.g., "facing eviction") or Abuse.  
**Role:** Evidence for Customer Safety and Duty of Care auditing.

### scenario_label
**Type:** String  
**Description:** The system-generated ground truth label injected by the data engine.  
**Role:** Used for logic validation and testing (Development only).

### escalation_validity
**Type:** String  
**Description:** The final audit outcome (Valid, Missed, Failed, or Invalid).  
**Role:** **Master Driver for all KPIs.** Defines the operational health of the system.

---

## Author

Kristine Soliman
Data & Operations Analyst | Chandler, AZ
