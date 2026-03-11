# TestMart API — Data Driven Test Collection

Postman/Newman data-driven test suite for the **Authentication** endpoints of the TestMart API. Covers user registration validation and login validation using CSV data files, with a single parameterised request per endpoint replacing individual test cases.

---

## Table of Contents

- [Overview](#overview)
- [Project Files](#project-files)
- [Collection Variables](#collection-variables)
- [CSV Data Files](#csv-data-files)
- [Running the Tests](#running-the-tests)
- [Test Assertions](#test-assertions)

---

## Overview

Instead of maintaining one Postman request per test scenario, this collection uses a **data-driven approach** — a single request per endpoint iterates over a CSV file where each row defines the inputs, expected status, and assertions for one test case.

This makes it easy to add new scenarios by adding a CSV row without touching the collection itself.

---

## Project Files

| File | Description |
|---|---|
| `Data_Driven_collection.json` | Main Postman collection with Login and Registration folders |
| `login_validation_data.csv` | CSV data file for login negative/validation tests |
| `registration_data.csv` | CSV data file for registration negative/validation tests |
| `reports/` | Output folder for generated HTML reports (auto-created) |

---

## Collection Variables

All configuration is stored as **collection variables** inside `Data_Driven_collection.json`. No separate environment file is needed.

| Variable | Description | Default Value |
|---|---|---|
| `base_url` | API base URL | `http://localhost:5001/api` |
| `reg_email` | Registered user email for login tests | `validation@test.com` |
| `reg_password` | Registered user password | `Str0ng!Pass#2024` |
| `reg_saved_email` | Email saved after valid registration | set at runtime |
| `blocked_user` | Email of a blocked user account | `blocked@test.com` |
| `blocked_user_pass` | Password of the blocked user account | `BlockedPass123!` |
| `dynamic_body` | Request body built dynamically per iteration | set at runtime |

> **Important:** Ensure you run node seedUser.js to populate your database with the required test users before running the tests.

---

## CSV Data Files

### login_validation_data.csv

Each row is one login test scenario. Column reference:

| Column | Description |
|---|---|
| `test_id` | Test case ID shown in the report |
| `test_name` | Test case name shown in the report |
| `email` | Email to send. Use `EMPTY` to omit the field. Use `{{collection_var}}` to resolve from collection variables |
| `password` | Password to send. Use `EMPTY` to omit the field. Use `{{collection_var}}` to resolve from collection variables |
| `expected_status` | Expected HTTP status code. Use `200_OR_401_OR_400` for the case sensitivity check |
| `expect_success` | Expected value of `success` field (`true` or `false`) |
| `error_keyword` | Keyword expected anywhere in the error response body. Use `EMPTY` to skip this check |
| `extra_check` | `no_token` to assert `accessToken` is absent. `case_sensitivity` for LG-004 behaviour logging |
| `scenario` | Internal scenario key used to trigger specific assertion blocks |

**Scenarios covered:**

| Test ID | Scenario | Expected Status |
|---|---|---|
| LG-002 | Wrong password | 401 |
| LG-003 | Non-existent user | 401 |
| LG-004 | Case sensitivity check | 200 or 401 or 400 |
| LG-005 | Blocked user account | 403 |
| LG-006 | Missing email field | 400 |
| LG-007 | Missing password field | 400 |
| LG-008 | Empty credentials | 400 |
| LG-009 | Invalid email format | 401 |

---

### registration_data.csv

Each row is one registration validation scenario. Column reference:

| Column | Description |
|---|---|
| `test_id` | Test case ID shown in the report |
| `test_name` | Test case name shown in the report |
| `email` | Email to send. Use `EMPTY` to omit the field |
| `password` | Password to send. Use `EMPTY` to omit. Wrap values containing `$` in double quotes e.g. `"Pa$$word"` |
| `name` | Name to send. Use `EMPTY` to omit the field |
| `include_unknown_field` | `true` to include an unknown field in the body |
| `expected_status` | Expected HTTP status code |
| `expect_success` | Expected value of `success` field |
| `error_keyword` | Keyword expected in error response body |
| `scenario` | Internal scenario key |

**Scenarios covered:**

| Test ID | Scenario | Expected Status |
|---|---|---|
| REG-003 | Invalid email format | 400 |
| REG-004 | Weak password | 400 |
| REG-005 | Missing multiple required fields | 400 |
| REG-006 | Missing email field | 400 |
| REG-007 | Missing password field | 400 |
| REG-008 | Missing name field | 400 |
| REG-009 | Empty body request | 400 |
| REG-010 | Unknown field in body | 400 |
| REG-011 | Whitespace only email | 400 |
| REG-012 | Password below minimum length | 400 |

> **CSV Rule:** Use `EMPTY` as a placeholder for fields you want omitted from the request body. Do **not** leave consecutive columns blank as this causes column shifting in Newman.

---

## Running the Tests

### Option 1: Newman Manual Commands

**Login validation tests:**
```bash
newman run Data_Driven_collection.json \
  --folder "Login" \
  --iteration-data login_validation_data.csv \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/login_tests_report.html \
  --reporter-htmlextra-title "Login Validation Tests"
```

**Registration validation tests:**
```bash
newman run Data_Driven_collection.json \
  --folder "Registration" \
  --iteration-data registration_data.csv \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/registration_tests_report.html \
  --reporter-htmlextra-title "Registration Validation Tests"
```

### Option 2: Postman Collection Runner

1. Open the collection in Postman
2. Click **Run collection**
3. Select the folder you want to run (**Login** or **Registration**)
4. Under **Data**, click **Select File** and upload the matching CSV file
5. Iterations will auto-fill based on the number of rows
6. Click **Run**

---

## Test Assertions

### Login Validation — assertions run per iteration

| Assert | Description |
|---|---|
| Status code | Matches `expected_status` from CSV |
| Success flag | `response.success` is `false` |
| Error keyword | Response body contains `error_keyword` from CSV |
| No token | `accessToken` is absent for failed logins |
| No user enumeration | Response does not leak `user not found` or `email does not exist` (LG-002, LG-003) |
| Blocked message | Correct blocked account message returned (LG-005) |
| Response time | Under 2000ms |

### Registration Validation — assertions run per iteration

| Assert | Description |
|---|---|
| Status code | Matches `expected_status` from CSV |
| Success flag | `response.success` is `false` |
| Error keyword | Response body contains `error_keyword` from CSV |
| No data leak | `id`, `email`, `password` are absent from error response |
| Response time | Under 2000ms |

---

