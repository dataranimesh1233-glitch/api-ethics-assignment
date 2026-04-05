# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

| Field           | Classification                   | Action                          | Justification                                                                                                                   |
| --------------- | -------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| full_name       | Direct PII                       | Drop or Pseudonymize            | Full name directly identifies an individual. It is not required for research. If needed for tracking, replace with a unique ID. |
| email           | Direct PII                       | Drop                            | Email is a unique identifier and unnecessary for analysis. Keeping it increases privacy risk.                                   |
| date_of_birth   | Indirect PII                     | Mask (convert to age/age range) | Exact DOB can be used for re-identification. Converting to age preserves analytical value.                                      |
| zip_code        | Indirect PII                     | Mask (keep partial)             | Full ZIP codes can identify individuals. Keep only first 2–3 digits or region.                                                  |
| job_title       | Indirect PII                     | Generalize                      | Some job titles may be unique. Group into broader categories (e.g., “Engineer”, “Healthcare Worker”).                           |
| diagnosis_notes | Sensitive Data (may include PII) | Clean and Mask                  | Free text may contain names or identifiers. Remove personal details while keeping medical insights.                             |

### Summary

* Remove direct identifiers completely.
* Reduce the detail of indirect identifiers.
* Clean unstructured text carefully to remove hidden PII.

---

## Task 2 — Audit the API Script for Ethical Compliance

### Issue 1: Hardcoded API Key

**Problem:**
The API key is written directly in the code. This is insecure and may violate API provider policies. It can expose the key if the code is shared publicly.

**Solution:**
Store the API key in an environment variable instead.

```python
import requests
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("HEALTH_API_KEY")

if not API_KEY:
    raise ValueError("API key not found. Set it as an environment variable.")
```

---

### Issue 2: Excessive Data Collection and Storage

**Problem:**

* The script collects 100 pages of data without checking limits.
* It stores raw personal data permanently.
* This violates data minimization and privacy principles.

**Solution:**

* Limit the number of pages collected.
* Add rate limiting.
* Store only cleaned and anonymized data.

```python
import requests
import time
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("HEALTH_API_KEY")

def clean_record(record):
    return {
        "patient_id": hash(record["email"]),  # pseudonymized ID
        "age": 2026 - int(record["date_of_birth"][:4]),
        "zip_prefix": record["zip_code"][:3],
        "job_category": record["job_title"],
        "diagnosis_notes": record["diagnosis_notes"][:200]
    }

records = []

for page in range(1, 21):  # reduced data collection
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})

    if response.status_code != 200:
        break

    data = response.json()

    for record in data["results"]:
        records.append(clean_record(record))

    time.sleep(1)  # prevent hitting API too fast

# Save only cleaned data
save_to_database(records)
```

---

## Final Notes

* Follow **data minimization** and **privacy-by-design** principles.
* Always review API Terms of Service before collecting data.
* Avoid storing sensitive personal information unless absolutely necessary.
* Ensure compliance with data protection regulations.

---
