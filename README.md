# Delta-Pharmacy-API-Testing

A complete QA engagement on the Delta Pharmacy full-stack application (React + Spring Boot), covering the full quality assurance lifecycle: requirements analysis, test design, live execution, and API test automation.

What this project includes:

Requirements Review — Analyzed a 6-User-Story Epic (Authentication & User Profile Management) before any test design, and identified 40 documented requirement gaps (missing validations, ambiguities, contradictions, missing security requirements), each written as a specific, answerable question for BA/PO/Dev, tracked in a structured gap-log spreadsheet.
Manual Test Design — Derived 396 manual test cases directly from source-code analysis (controllers, DTOs, validation annotations, Spring Security config) across 17 modules — Authentication, Registration, Products, Orders, Payments, Prescriptions, Reviews, Support, Chat, Notifications, Users, Dashboard/Analytics, Security, Navigation, UI/UX, and E2E — covering happy path, negative/boundary cases, and security scenarios (IDOR, privilege escalation, SQLi/XSS).
API Test Automation — Built a 112-request Postman collection (plus environment) covering every documented endpoint, with automated pm.test assertions for happy/negative paths and dedicated checks for IDOR and privilege-escalation vulnerabilities uncovered during the review.

Tech/tools used: Spring Boot (Java) & React source analysis, Postman (Collections, Environments, test scripting), Excel-based test management (RTM-style gap log, test case matrix).
