# NovaCred: Credit Application Governance Report
**DEGO 2606 | Group Project**
*A comprehensive audit of data quality, algorithmic fairness, and GDPR compliance.*

## 1. Executive Summary
This repository contains the findings of the Data Governance Task Force auditing the `raw_credit_applications.json` dataset for NovaCred. Our objective was to ensure the integrity of the credit approval algorithm by addressing data quality issues, identifying algorithmic bias, and implementing privacy controls. We successfully transformed the raw nested data into a clean tabular format, remediated critical data quality issues, and identified a Disparate Impact Ratio of 0.77 concerning gender.

## 2. Data Engineering & Quality Audit
To ensure the integrity of the downstream algorithmic fairness and privacy analysis, the Data Engineering team ingested the nested `raw_credit_applications.json` and executed a robust cleaning pipeline, resulting in a clean dataset of **495 records**.

### 2.1. Data Ingestion & Flattening
The original JSON contained deeply nested structures (`applicant_info`, `financials`, `decision`) and dynamic arrays (`spending_behavior`). 
* **Flattening:** We utilized `pd.json_normalize` to unpack nested dictionaries into standard tabular columns.
* **Array Pivoting:** The `spending_behavior` array was pivoted into 11 distinct numerical features (e.g., `spending_Healthcare`, `spending_Rent`). This crucial transformation allows the Data Scientist to run direct statistical correlations between individual spending categories and loan approval outcomes.

### 2.2. Data Quality Findings & Remediation
We conducted a systematic audit across the four core dimensions of data quality. Below is the summary of the intentional issues discovered and the remediation applied:

| Quality Dimension | Issue Discovered | Remediation Applied |
| :--- | :--- | :--- |
| **Consistency (Schema & Formats)** | • Misaligned keys: Some rows used `annual_salary` instead of `annual_income`.<br>• Inconsistent labels: Gender contained 'M'/'F' alongside 'Male'/'Female'.<br>• Data Types: `annual_income` was loaded as text, and dates were strings. | • Consolidated `annual_salary` into `financials.annual_income`.<br>• Standardized gender values to 'Male', 'Female', or 'Unknown'.<br>• Cast financial columns to `float64` and `date_of_birth` to `datetime`. |
| **Validity (Logical Rules)** | • Impossible financial values: We detected negative values in `credit_history_months` and `savings_balance`.<br>• Malformed strings: 11 malformed email addresses were detected. | • Converted impossible negative values to `NaN`.<br>• Replaced all numeric `NaN` values using **median imputation** to preserve the distribution without dropping records. |
| **Completeness (Missing Data)** | • Missing critical variables: 4 records had missing/invalid dates of birth; missing categorical data (e.g., emails). | • Categorical `NaN`s were labeled explicitly as 'Unknown'.<br>• Invalid dates were standardized, excluding the 4 unparseable DOBs from age-dependent analysis. |
| **Accuracy (Duplication)** | • Entity duplication: Found 3 unique SSNs that were duplicated across 11 different application records. | • Dropped exact row duplicates, keeping only the first valid entity representation based on SSN. |

## 3. Algorithmic Bias & Fairness Analysis
Our fairness evaluation assessed whether demographic attributes like gender or age improperly influenced loan approval outcomes.
* **Disparate Impact:** The Disparate Impact (DI) ratio for gender is **0.77** (Female approval rate: 50.6%, Male: 65.8%). This falls below the industry-standard 0.8 threshold (the four-fifths rule), indicating a disparate impact on female applicants.
* **Proxy Discrimination:** A Chi-Square test revealed a significant relationship between **ZIP Code and Gender** (p-value < 0.05), indicating geographic data is acting as a proxy variable for gender in the algorithmic pipeline.
* **Age & Financial Correlation:** Approval outcomes are primarily associated with age-related financial characteristics (annual income, credit history length) rather than being driven by a single attribute alone.

## 4. Privacy & Governance Gaps
The governance audit evaluated privacy risks and regulatory compliance related to the credit application dataset.
* **Identified PII:** Social Security Numbers (SSN), Full Names, Email Addresses, Date of Birth, ZIP Codes, and IP Addresses were identified as direct or quasi-identifiers capable of revealing or indirectly linking to individual identities.
* **Re-identification Risk:** A k-anonymity assessment showed that combinations of demographic attributes (age, gender, ZIP code) can uniquely identify individuals in the dataset, indicating potential linkage risks even after removing direct identifiers.
* **Governance Gaps:** The dataset stores highly sensitive identifiers and lacks explicit privacy controls, audit traceability, and fairness monitoring mechanisms within the data pipeline.
* **Proposed Governance Controls:** Implementation of pseudonymization (SHA-256 hashing), anonymization of quasi-identifiers (generalization and masking), and governance controls aligned with GDPR data minimization and EU AI Act requirements for high-risk credit decision systems.

## 5. Repository Structure
To ensure full reproducibility of our ETL pipeline and analysis, the repository is structured as follows:
* `data/raw/`: Contains the original `raw_credit_applications.json`.
* `data/processed/`: Contains the `cleaned_credit_applications.json` ready for ML models and downstream analysis.
* `notebooks/`: 
  * `01-data-quality.ipynb`: The ETL and data cleaning pipeline handling nested JSON parsing and quality remediation.
  * `02-bias-analysis.ipynb`: The algorithmic fairness, disparate impact, and proxy variable analysis using `fairlearn`.
  * `03-privacy-remediation.ipynb`: *(WIP)* Implementation of GDPR-compliant data masking and pseudonymization.

## 6. Team Contributions
* **Alessandro Rollandin - Data Engineer & Product Lead:** Developed the JSON flattening logic, performed the 4-dimensional data quality audit, built the remediation pipeline, and structured the project repository.
* **Alexia Sousa - Data Scientist & Product Lead:** Conducted disparate impact and proxy discrimination analysis using Fairlearn.
* **Lucia Musizzano - Governance Officer & Product Lead:**  Governance Officer & Product Lead:** Performed the privacy and governance audit, identifying PII, implementing pseudonymization via SHA-256 hashing, evaluating re-identification risk through k-anonymity, and defining GDPR- and AI Act-aligned governance controls for the credit decision system.
