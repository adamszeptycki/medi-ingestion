# MediBatch Ingestion System

## Overview
A HealthTech platform that processes large nightly CSV dumps from legacy Hospital Information Systems (HIS).  
encounter_id: String (Unique identifier)
patient_mrn: String (Medical Record Number)
patient_name: String (e.g., "John Doe")
gender: String (M/F/U)
encounter_date: Timestamp (ISO 8601)
diagnosis_code: String (ICD-10 format, e.g., "J01.90")
billable_amount: Float

---

## 🚚 Ingestion & Validation
Each hospital uploads a large CSV containing patient visit records.  
The microservice must:

1. Accept CSV file uploads.
2. Persist Data in Database.