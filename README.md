# Healthcare Payer-Provider Claims Reconciliation

> **Disclaimer:**  
> This project was developed independently using synthetic data for demonstration purposes only.  
> All code is original and does not contain any proprietary logic, data, or assets.
>
> **Data Privacy Note:**  
> All data was generated using Python’s [Faker library](https://faker.readthedocs.io/en/master/).  
> No real patient, provider, or payer information is included anywhere in this project.

## Quickstart

1. Clone this repo and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. All input data is in `/data` as CSV files.

3. Open `notebook/synthetic-healthcare-claims-recon.ipynb` and run all cells top-to-bottom.

4. Output is saved to:  
   - `/recon_results/claims_crosswalk.csv` (matched payer claims with provider MRN assignment and match level)
   - `/recon_results/unmapped_provider_claims.csv` (provider MRNs not found in the payer data)
   - `/recon_results/payer_provider_data_merged.csv` (payer data enriched with provider data after mapping)
   - `/recon_results/underpayment_summary.csv` (CPT-level underpayment analytics)

---

## Overview

In healthcare, providers rarely receive full payment from insurance companies for every claim submitted.
When providers have questions about payments, payers send reports showing what they believe was processed or owed.
However, these reports are often messy; missing key identifiers, using inconsistent formats, or omitting important fields altogether.
Similarly, the data providers receive from their clearinghouse can be incomplete, inconsistent, or out of sync with payer reports.

This lack of alignment between payer and provider data creates significant challenges in verifying payments and identifying underpayments, especially when there is rarely a single, reliable field that links the two datasets together.
Without effective reconciliation, providers risk losing substantial revenue due to claims that are unpaid, partially paid, or incorrectly processed.
The reconciliation process bridges these gaps, enabling providers to “translate” payer data into their own system’s terms, accurately match claims, detect discrepancies, and pursue
recovery of owed payments.

---

## Data Description

This project uses two synthetic datasets to simulate a real-world payer-provider reconciliation workflow:

- **Payer Data (`payer_data_clean.csv`)**  
  Represents the payer’s claims file, typically received as an encounter level payment report.  
  Each row is a claim as reported by the payer, containing:
    - Payer-assigned claim/control number
    - Account number, policy ID, service date, CPT/procedure code, patient identifiers (synthetic)
    - Payment amount as recorded by the payer

- **Provider Data (`provider_claims.csv`)**  
  Represents the provider’s internal record of submitted claims and encounters.  
  Each row is an encounter or billed claim the provider believes should be paid, containing:
    - Provider-assigned MRN (medical record number), all matching fields (claim number, account number, policy ID, service date, CPT/procedure code, patient identifiers)
    - Billed charge amount, expected/contracted payment, and what the provider believes was paid

**Relationship:**  
The provider dataset is intended to reflect the set of claims and encounters the provider system *expects* to match what the payer reports.  
In practice, some claims will be unmatched in one or both datasets, simulating real-world gaps due to rejections, denials, timing, or data errors.

Both files are 100% synthetic, generated with the Python [Faker](https://faker.readthedocs.io/en/master/) library, and contain no real patient, provider, or payer data.

---

## Why This Project?

Out-of-the-box tools can't handle the reality of matching these imperfect datasets at scale. This project provides:

- **Flexibility**: Works with any payer report, even with missing fields.  
- **Efficiency**: Handles millions of records without manual intervention.  
- **Transparency**: Clearly documents how and where claims are matched, enabling audit, payment recovery, and payer negotiations.  
- **Defensibility**: Provides evidence to pursue underpayments and resolve disputes.
- **Payment Integrity**: Quantifies underpayments by claim, procedure, and payer; identifies patterns and severe revenue gaps to support actionable recovery and process improvement.

---

## The Solution

A stepwise reconciliation pipeline, starting with exact identifiers, then falling back to demographic and fuzzy matching, reliably maps as many payer claims as possible to internal MRNs.

---

## Matching Logic: Levels Explained

| Level  | Criteria                                                      |
|--------|---------------------------------------------------------------|
| l1     | Exact match on claim number (payer control number)            |
| l2     | Match on account number or patient control number             |
| l3     | Demographic: policy ID, DOB, service date, name fragments     |
| l3_1   | Demographics + procedure code (CPT)                           |
| l3_2   | Fuzzy initials + DOB, service date, procedure code            |
| l3_3   | Name fragments + service date + procedure code                |
| l3_4   | Initials + service date + procedure code                      |

Each claim is only counted at its first successful match level.  
Mapping success rates are reported at each level and overall.

---

## Business Impact

- **Proven in Practice**: A similar reconciliation framework at my organization supported the recovery of substantial underpayments across multiple payers.  
- **Negotiation Leverage**: Provides a clear, reproducible record of underpaid claims and the match logic used. 
- **Auditability and Process Improvement**: Helps uncover missed claims, resolve data discrepancies, and drive improvements in clearinghouse workflows to prevent revenue loss.  
- **Scalable Across Payers**: Designed to handle any payer TIN report, giving leadership confidence in both reconciliation accuracy and internal data quality.

---

## How It Works (At a Glance)

- **Stepwise Matching**: Strictest fields first (claim number), then account number, demographics, fuzzy initials + date + CPT, etc.  
- **Waterfall Logic**: Once a claim is matched, it does not move to the next step.  
- **Assignment**: If multiple matches exist, the script selects one MRN per claim (while tracking non-unique mappings for transparency).  
- **Auditability**: Reports mapping success and counts at every match level.
- **Underpayment Analysis**: After reconciliation, this workflow benchmarks actual payments received (from payer data) against contracted amounts (from provider data). 
---
