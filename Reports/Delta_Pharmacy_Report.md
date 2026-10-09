# Delta Pharmacy — QC Report

End-to-end QA engagement on the **Delta Pharmacy** full-stack (React frontend + Spring Boot backend): requirements review, manual test design, live execution, and API test automation.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tools & Technologies](#2-tools--technologies)
3. [Deliverables](#3-deliverables)
4. [Execution Status](#4-execution-status)
5. [Confirmed Defects](#5-confirmed-defects)
6. [Suspected Issues (Pending Execution)](#6-suspected-issues-pending-execution)
7. [Requirements Review Highlights](#7-requirements-review-highlights)
8. [Test Design Approach](#8-test-design-approach)
9. [API Automation (Postman)](#9-api-automation-postman)
10. [How to Reproduce](#10-how-to-reproduce)

---

## 1. Project Overview

| Item | Detail |
|---|---|
| Website | Delta Pharmacy Management System |
| Frontend | React |
| Backend | Spring Boot REST API (JWT authentication, role-based access) |
| Roles | Customer, Pharmacist, Admin |
| Scope | Authentication, Registration, Products, Inventory, Cart, Orders, Payments, Prescriptions, Reviews, Support, Chat, Notifications, Users, Dashboard/Analytics, Navigation, UI/UX, E2E |

Test cases were derived from the actual source code (controllers, DTOs, validation annotations, security configuration, services). No features, fields, or rules were invented.

---

## 2. Tools & Technologies

- **Postman** — API collection, environment, and `pm.test` assertions
- **Microsoft Excel** — requirements gap log and test case matrix
- **Source analysis** — Spring Boot (Java) and React code review
- **Swagger UI** — endpoint reference

---

## 3. Deliverables

| Deliverable | File | Description |
|---|---|---|
| Requirements gap log | `Auth_Profile_Requirements_Gaps.xlsx` | 40 documented requirement gaps with type, priority, owner, and status |
| Manual test cases | `Delta_Pharmacy_Test_Cases.xlsx` | 396 test cases across 17 modules, with Actual Result / Status filled for executed cases |
| Postman collection | `Delta_Pharmacy_API.postman_collection.json` | 112 requests with automated assertions |
| Postman environment | `Delta_Pharmacy_Local.postman_environment.json` | Variables for base URL, tokens, and IDs |
| This report | `Delta_Pharmacy_Report.md` | Summary of approach, results, and defects |

---

## 4. Execution Status

### Overall

| Metric | Count |
|---|---|
| Total test cases | 396 |
| Executed | 396 |
| Passed | 337 |
| Failed | 57 |
| Blocked | 2 |
| Not run | 8 |

### By module

| Module | Total | Executed | Pass | Fail | Blocked | Not Run |
|---|---|---|---|---|---|---|
| Authentication – Login | 35 | 35 | 32 | 2 | 0 | 0 |
| Authentication – Registration | 61 | 61 | 37 | 24 | 0 | 0 |
| Products | 52 |  52| 48 | 24 | 0 | 0 |
| Inventory | 16 | 16 | 12 | 3 | 1 | 0 |
| Shopping Cart | 8 | 8 | 7 | 1 | 0 | 0 |
| Orders | 66 | 61 | 55 | 5 | 1 | 4 |
| Prescriptions | 27 | 27 | 24 | 2 | 0 | 0 |
| Reviews | 19 | 19 | 18 | 1 | 0 | 0 |
| Support | 28 | 28 | 27 | 1 | 0 | 0 |
| Chat | 18 | 17 | 16 | 1 | 0 | 1 |
| Notifications | 11 | 11 | 10 | 1 | 0 | 0 |
| Users Management | 17 | 15 | 8 | 8 | 0 | 2 |
| Dashboard & Analytics | 14 | 14 | 12 | 2 | 0 | 0 |
| API Security (Cross-Cutting) | 3 | 2 | 2 | 0 | 0 | 1 |
| UI / UX / Accessibility | 11 | 11 | 9 | 2 | 0 | 0 |
| End-to-End Scenarios | 8 | 8 | 8 | 0 | 0 | 0 |


### Blocked test case

| ID | Reason |
|---|---|
| AUTH-LOGIN-033 | Verifying JWT expiry requires waiting the full 24h token lifetime or altering server configuration/clock. Neither was done, to keep the application unchanged. |

---

## 5. Confirmed Defects

All three were reproduced against the running application.

### BUG-001 — Password hash exposed in profile response
- **Test case:** AUTH-LOGIN-028
- **Severity:** High
- **Endpoint:** `GET /auth/profile`
- **Steps:**
  1. Log in as any user and obtain a JWT.
  2. Call `GET /auth/profile` with `Authorization: Bearer <token>`.
- **Expected:** Response contains `id`, `email`, `fullName`, `role` only. No password data.
- **Actual:** HTTP 200. Response body includes a `password` field containing the full BCrypt hash.
- **Likely cause:** The `User` entity has no `@JsonIgnore` / write-only annotation on the `password` field, so it is serialized.
- **Impact:** An attacker who obtains the hash can attempt offline brute-force or dictionary attacks.

### BUG-002 — No auto-logout when token is invalid
- **Test case:** AUTH-LOGIN-034
- **Severity:** Medium
- **Steps:**
  1. Log in, then replace the `token` value in `localStorage` with an invalid string.
  2. Navigate to a page that calls an authenticated endpoint (e.g. `/orders`).
- **Expected:** Session is cleared and the user is redirected to `/login`.
- **Actual:** The backend returns **403** for the invalid token. The frontend stays on the page and the token remains in `localStorage`.
- **Likely cause:** The frontend response interceptor reacts to **401** only, while the backend returns **403** for malformed tokens.
- **Impact:** The user appears logged in while every protected call fails.

### BUG-003 — Logout button does nothing
- **Test case:** AUTH-LOGIN-035
- **Severity:** High
- **Steps:**
  1. Log in as a customer.
  2. Click the red **Logout** button in the sidebar.
- **Expected:** `token` and `user` removed from `localStorage`, redirect to `/login`.
- **Actual:** No effect. No navigation, no storage change, no console output, no network request. Reproduced with multiple click approaches.
- **Impact:** Users cannot sign out from the UI.

---

## 7. Requirements Review Highlights

The Epic *Authentication & User Profile Management* (6 user stories) was reviewed **before** test design. It produced **40 gaps**.

| Priority | Count |
|---|---|
| High | 19 |
| Medium | 15 |
| Low | 6 |

Representative questions raised:

- Which registration fields are mandatory, and what are the validation rules for email, password, phone, name, and address?
- Can a role be supplied at registration, and who is allowed to change a role later?
- What are the expected HTTP status codes and response structure for success and failure on each endpoint?
- Must password or hash data be explicitly excluded from every response?
- `GET /auth/profile` returns only id/email/fullName/role, yet `PUT /auth/update` can change phone and address. Is this intentional?
- Does a password change invalidate existing JWTs? Is there a logout or token-invalidation mechanism?
- BCrypt is described as "encryption", though it is a one-way hash. Please confirm intent.

Several execution findings trace back to these gaps (role assignment rules, password exposure, logout behavior).

---

## 8. Test Design Approach

Scenarios were derived systematically from two sources: validation annotations on DTOs, and manual checks inside service methods.

For each field, the following were considered where applicable:

- Missing field, empty value, `null`, whitespace-only
- Invalid data type, invalid format, invalid value
- Boundary values (minimum, below minimum, maximum, above maximum)
- Duplicate data, non-existing ID, already-deleted resource
- Unauthorized (no/invalid token) and forbidden (wrong role)
- Malformed JSON
- SQL injection and XSS payloads (safe payloads only)

Expected status codes were predicted by tracing which exception handler would fire, rather than assuming 400. Where actual behavior diverges from the intended behavior, it is recorded as a defect.

---

## 9. API Automation (Postman)

- **112 requests** across 18 folders, covering the documented Swagger endpoints for Auth, Products, Inventory, Search, Orders, Payments, Prescriptions, Reviews, Chat, Support, Notifications, Users, Dashboard, and Reports.
- Happy-path and negative scenarios for each endpoint.
- `pm.test` assertions on status code and response fields.
- Dynamic variables (tokens, IDs) are set from earlier requests, so the collection must be run **in order**, starting with the `0. Setup` folder.
- Requests tagged `[SECURITY]`, `[CRITICAL SECURITY]`, or `[IDOR FINDING CHECK]` assert the safe behavior. A failure on these indicates a confirmed vulnerability.

> **Note:** The Postman collection has been built and imported, but a full collection run report has not been attached to this repository yet.

---

## 10. How to Reproduce

### Prerequisites
- Backend running at `http://localhost:8545/pharmacy-api`
- Frontend running at `http://localhost:3001`
- Seeded users available (Admin, Pharmacist, Customer)

### API tests
1. Import the collection and environment into Postman.
2. Select the **Delta Pharmacy – Local** environment.
3. Run the collection in order, starting with `0. Setup (run first)`.
4. Review the Test Results panel for failed assertions.

### Manual tests
1. Open `Delta_Pharmacy_Test_Cases.xlsx`, sheet **Test Cases**.
2. Follow Preconditions, Test Steps, and Test Data for each row.
3. Record Actual Result and Status.
4. Do not edit Expected Result to match observed behavior.

---

## 11. Limitations & Next Steps

**Limitations**
- Items in section 6 are inferred from code review and still need execution.
- JWT expiry (AUTH-LOGIN-033) was not verifiable without altering configuration.
- Responsive and accessibility cases have not been executed.
