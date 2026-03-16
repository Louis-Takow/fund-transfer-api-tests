# Fund Transfer API — Test Suite Documentation

**Date:** February 23, 2026  
**Author:** Louis Takow  
**Endpoint:** `POST /api/v1/transfers`

---

## Overview

- **Total Test Cases:** 9  
- **Total Assertions:** 37  
- **Mock Server:** Postman Mock Server (no live API provided)  
- **Approach:** Mock examples simulate `201`, `400`, `401`, `403`, `422` responses  

---

## How to Run

### Using Postman

1. Import `Fund_Transfer_API_Tests.postman_collection.json` into Postman  
2. Import `QA_Environment.postman_environment.json`  
3. Select **QA Environment** from the top-right dropdown  
4. Click **Run Collection**  
5. All 9 tests execute with 37 assertions — expect all green

### Using Newman (CLI)

1. Install Newman globally (if not installed):

```bash
npm install -g newman
````

2. Run tests from terminal/PowerShell:

```bash
newman run Fund_Transfer_API_Tests.postman_collection.json -e QA_Environment.postman_environment.json
```

3. Run with detailed report:

```bash
newman run Fund_Transfer_API_Tests.postman_collection.json -e QA_Environment.postman_environment.json -r cli,html
```

---

## CI/CD Integration with GitHub Actions

1. Create a workflow file: `.github/workflows/api-tests.yml`

```yaml
name: API Tests

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  postman-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install Newman
        run: npm install -g newman

      - name: Run Postman Collection
        run: newman run Fund_Transfer_API_Tests.postman_collection.json -e QA_Environment.postman_environment.json -r cli,html
```

2. Commit and push workflow to GitHub
3. Tests run automatically on every push or pull request

---

## CI/CD Flow Diagram

```text
+-----------------+      +----------------+      +--------------------+
|                 |      |                |      |                    |
|  Postman Tests  +----->+    Newman      +----->+  GitHub Actions    |
|                 |      |  CLI Execution |      |  CI/CD Pipeline    |
+-----------------+      +----------------+      +--------------------+

Flow:
1. Define and run Postman collection locally
2. Execute collection via Newman CLI for automated testing
3. Newman integrated into GitHub Actions for CI/CD automation
```

---

## Test Coverage

| TC_ID  | Name                 | Category         | Mock Code | Assertions |
| ------ | -------------------- | ---------------- | --------- | ---------- |
| TC_001 | Valid Transfer       | Happy Path       | 201       | 9          |
| TC_002 | Missing from_account | Field Validation | 400       | 4          |
| TC_003 | Negative Amount      | Field Validation | 400       | 3          |
| TC_004 | Invalid Currency     | Field Validation | 400       | 3          |
| TC_005 | Missing Token        | Auth             | 401       | 4          |
| TC_006 | Frozen Account       | Business Rules   | 403       | 3          |
| TC_007 | Insufficient Funds   | Business Rules   | 422       | 3          |
| TC_008 | Large Amount         | Edge Cases       | 201       | 4          |
| TC_009 | Recurring Enabled    | Edge Cases       | 201       | 4          |

---

## Test Data — Accounts Used

| Variable          | Value            | Purpose                 |
| ----------------- | ---------------- | ----------------------- |
| validFromAccount  | ACC-10001-USD    | Active source account   |
| validToAccount    | ACC-20002-USD    | Active destination      |
| frozenAccount     | ACC-30003-FROZEN | Frozen/blocked account  |
| lowBalanceAccount | ACC-40004-LOWBAL | Account with $1 balance |

---

## Test Data Considerations

### Amounts Tested

| Scenario  | Value        | Expected |
| --------- | ------------ | -------- |
| Normal    | 100.00       | 201      |
| Negative  | -100.00      | 400      |
| Large     | 999999999.99 | 201/400  |
| Recurring | 250.00       | 201      |

### Dates Used (relative to Feb 23, 2026)

| Field                | Value                | Why                 |
| -------------------- | -------------------- | ------------------- |
| scheduled_date       | 2027-03-01T10:00:00Z | Future date (valid) |
| recurring_end        | 2026-06-01T09:00:00Z | 3 months out        |
| estimated_completion | 2026-03-07T10:00:00Z | Next business day   |

---

## Currencies Tested

* **Valid:** USD
* **Invalid:** FAKECOIN

---

## Key Fintech Risks Addressed

* Frozen accounts cannot send money (TC_006)
* Insufficient balance caught before transfer (TC_007)
* No access without authentication (TC_005)
* Invalid inputs rejected with clear errors (TC_002–004)
* Large amounts do not crash the server (TC_008)
* Recurring transfers create valid transfer records (TC_009)

**All success responses include:**
`transfer_id`, `status`, `exchange_rate`, `fees`, `estimated_completion`

---

## Mock vs Real API Note

* Tests use **Postman Mock Server** since no live API was provided
* To run against a real API:

  * Change `baseUrl` in environment
  * Remove `x-mock-response-code` headers

All test assertions validate real response structure.

