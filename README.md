
# 🛡️ ConsentGuard
### PS-002: Purpose-Based Consent Recording & Withdrawal

<p align="center">
  <strong>A purpose-based consent management and enforcement design</strong>
</p>

<p align="center">
  <a href="https://youtu.be/X_m1AcXzKdA">
    ▶️ Watch Project Introduction
  </a>
</p>

---

> **Project Status:** Proposed design — not yet a deployed or tested implementation.

## 🎥 Introduction Video

Watch the introduction to ConsentGuard:

**[▶️ Watch on YouTube](https://youtu.be/X_m1AcXzKdA)**

## 📌 Overview

**ConsentGuard** is a proposed consent-management workflow designed to connect purpose-specific consent records with downstream data-access decisions.

The goal is to prevent downstream services from relying on outdated consent after a user changes or withdraws their decision.

The proposed system uses:
- One synthetic service
- Synthetic users
- Three separate data-sharing purposes
- Purpose-specific consent grants and withdrawals
- Notice versioning and SHA-256 hashing
- A conceptual enforcement layer
- A mock downstream data-sharing API
- An exportable audit trail

ConsentGuard is intended to demonstrate how consent records can be connected to access-control decisions and how those decisions can be recorded for auditing.

## 🚨 Problem Statement

Recording consent is not sufficient if downstream services continue processing data without checking the user's latest decision.

Common problems include:

- ❌ A withdrawal is recorded, but subsequent data-sharing requests are still allowed.
- ❌ Consent granted for one purpose is incorrectly reused for another purpose.
- ❌ Changes to privacy notices cannot be reliably associated with earlier decisions.
- ❌ Missing or unverifiable consent is handled ambiguously.
- ❌ Audit records may not provide sufficient evidence of access decisions.

ConsentGuard proposes an explicit enforcement check before a downstream request is allowed to access data.

## 🎯 Objectives

1. Maintain consent separately for each purpose.
2. Record grants and withdrawals as individual events.
3. Associate consent decisions with notice versions and SHA-256 hashes.
4. Check the latest consent state for every access request.
5. Deny requests when consent is missing or cannot be verified.
6. Record allow/deny decisions in an audit trail.
7. Support planned CSV and JSON audit exports.

## 🧩 Three Consent Purposes

| ID | Purpose | Description |
|---|---|---|
| P1 | Service Notifications | Consent for service-related notifications |
| P2 | Recommendations | Consent for personalised recommendations |
| P3 | Anonymous Usage Analytics | Consent for anonymous usage analytics |

Each purpose has its own consent state. A grant for one purpose must not automatically authorise access for another purpose.

## ⚙️ Key Features

### 1. Purpose-Based Consent Registry
Maintains consent state for each synthetic user and purpose.

Possible states:
- `GRANTED`
- `WITHDRAWN`

The proposed registry preserves event history rather than relying exclusively on the latest status.

### 2. Notice Versioning
Associates each consent decision with a notice version identifier and its SHA-256 hash.

This helps identify which version of a notice was associated with a decision.

> A hash can help detect changes in notice content. It does not prove that a user read or understood the notice.

### 3. Access Enforcement
A conceptual enforcement layer checks:
- User identity
- Requested purpose
- Latest consent state
- Applicable notice-version policy

Requests are denied by default when consent cannot be verified.

### 4. Mock Downstream Consumer
Represents a synthetic client requesting data for a particular purpose through a mock API.

### 5. Audit Trail
The proposed audit trail records consent events and access decisions, including:
- Request ID
- User ID
- Purpose ID
- Decision
- Decision reason
- Timestamp

CSV and JSON export are planned capabilities.

## 🏗️ System Architecture

```text
              +----------------------+
              |    Synthetic User    |
              +----------+-----------+
                         |
                         v
              +----------------------+
              |  Consent Interface   |
              |   Grant / Withdraw   |
              +----------+-----------+
                         |
                         v
+------------------------+-------------------------+
|             ConsentGuard Backend                |
|                                                 |
|  +-------------------------------------------+  |
|  |               Backend API                 |  |
|  +---------------------+---------------------+  |
|                        |                        |
|                        v                        |
|  +-------------------------------------------+  |
|  |           Enforcement Layer               |  |
|  |   Check state, purpose and policy         |  |
|  +----+----------------+------------------+--+  |
|       |                |                  |     |
|       v                v                  v     |
|  +-----------+  +-------------+  +------------+ |
|  |  Consent  |  |   Notice    |  |   Audit    | |
|  |  Registry |  |  Versions   |  |   Trail    | |
|  +-----------+  +-------------+  +------------+ |
+-------------------------------------------------+
                         ^
                         |
              +----------+-----------+
              | Mock Downstream      |
              | Consumer / API       |
              +----------------------+

                 ALLOW / DENY
                       |
                       v
              Audit Decision Record
                       |
                       v
                CSV / JSON Export
                  (Planned)
```

**Important:** This diagram represents the intended architecture. It does not indicate that the components have already been implemented.

## 🔄 Proposed Workflow

### Step 1: Grant Consent
A synthetic user grants consent for P1.

The registry records:
- Purpose: P1
- State: `GRANTED`
- Notice version: v1
- SHA-256 hash of the notice
- Event timestamp

### Step 2: Authorised Request
The downstream consumer requests data for P1.

The enforcement layer checks the current consent state. If the request satisfies the policy, the proposed result is `ALLOW`.

### Step 3: Withdraw Consent
The user withdraws consent for P1.

A new `WITHDRAWN` event is appended to the event history.

### Step 4: Denied Request
The consumer submits another request for P1.

The proposed enforcement layer returns `DENY` because consent has been withdrawn. The decision is added to the audit trail.

### Step 5: Notice Change
A new notice version, such as v2, is registered with its SHA-256 hash.

Under the proposed policy, consent associated with an older notice version requires re-confirmation.

## 🔀 Data Flow Diagram (DFD) — Level 1

### External Entities
- **Synthetic User:** Grants or withdraws consent.
- **Mock Downstream Consumer:** Requests data for a purpose.
- **Service Operator:** Manages notice publication and related operations.

### Processes

| Process | Description |
|---|---|
| 1.0 — Record Consent Decision | Records grant and withdrawal events |
| 2.0 — Register Notice Version | Registers notice versions and hashes |
| 3.0 — Evaluate Access Request | Checks consent and determines allow/deny |

### Data Stores

| Store | Description |
|---|---|
| D1 — Consent Registry | Consent states and event history |
| D2 — Notice Versions | Notice version identifiers and hashes |
| D3 — Audit Log | Consent events and access decisions |

### Main Data Flows

- Grant / withdraw
- Append event
- Publish notice
- Notice version and hash
- Data request
- Latest consent state
- Current notice version
- Allow / deny result
- Decision record
- Audit export

A request is proposed to be denied if its consent cannot be verified or if it fails the applicable purpose and notice-version policy.

## 🔐 Security and Audit Principles

| Principle | Proposed Approach |
|---|---|
| Purpose limitation | Maintain separate consent states for each purpose |
| Latest-state enforcement | Check current consent for every request |
| Default denial | Deny when consent is missing or unverifiable |
| Notice traceability | Store notice version and SHA-256 hash |
| Event history | Preserve grant and withdrawal events |
| Auditability | Record access decisions and their reasons |
| Data minimisation | Use synthetic data for the demonstration |

These are design principles. Their effectiveness must be validated through implementation and testing.

## 🧪 Planned Test Coverage

The project plan calls for **at least 12 lifecycle and enforcement tests**.

| Test | Expected Behaviour |
|---|---|
| Grant consent for P1 | Record the grant |
| Grant consent for P2 | Record the separate grant |
| Grant consent for P3 | Record the separate grant |
| Request after valid grant | Allow if all checks pass |
| Request without consent | Deny |
| Withdraw consent | Append withdrawal event |
| Request after withdrawal | Deny |
| Purpose mismatch | Deny |
| Notice version changes | Apply re-confirmation policy |
| Re-grant consent | Record the new grant |
| Unknown user or purpose | Deny safely |
| Missing consent state | Deny by default |
| Unverifiable consent | Deny by default |
| Audit export | Verify expected records are present |

These are planned test cases, not completed test results.

## 🛠️ Proposed Technology Components

The implementation technology choices remain to be finalised. A possible stack is:

| Component | Candidate Technology |
|---|---|
| Backend API | Python with FastAPI |
| Consent registry | SQLite |
| Notice hashing | Python `hashlib` |
| Data exchange | JSON |
| Audit export | Python `csv` and `json` |
| Frontend | HTML, CSS and JavaScript |
| Testing | `pytest` |
| Version control | Git and GitHub |

These are **suggested technologies**, not a claim that the current project already uses them.

## 🗺️ Development Roadmap

1. **Define:** Finalise purposes, data fields and notice policy.
2. **Design:** Finalise architecture, DFD and record schemas.
3. **Build:** Implement the registry, notice hashing, mock API and consumer.
4. **Enforce and Audit:** Implement access decisions and audit export.
5. **Test:** Run lifecycle and enforcement tests.
6. **Demonstrate:** Prepare a working demonstration, documentation and submission materials.

The development schedule is to be confirmed against the relevant hackathon timeline.

## 📚 Standards and References

1. [MeitY — Data Protection Framework](https://meity.gov.in/data-protection-framework)
2. Government of India — Digital Personal Data Protection Act, 2023.
3. MeitY — Digital Personal Data Protection Rules, 2025.
4. [GDPR — Regulation (EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
5. [NIST FIPS 180-4 — Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)
6. [RFC 7662 — OAuth 2.0 Token Introspection](https://www.rfc-editor.org/rfc/rfc7662)
7. [RFC 7009 — OAuth 2.0 Token Revocation](https://www.rfc-editor.org/rfc/rfc7009)
8. [RFC 8785 — JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785)
9. ISO/IEC 29184:2020 — Online privacy notices and consent.
10. Kantara Initiative — Consent Receipt Specification v1.1.

These references inform the proposed design; they are not implemented wholesale by the system. Verify the exact legal provisions before making legal claims.

## ⚠️ Scope and Limitations

- ConsentGuard is a proposed design, not a deployed system.
- No completed prototype, test results or performance benchmarks are claimed.
- The service, users and purposes are synthetic examples.
- The mock consumer is intended to demonstrate the access-control workflow.
- Notice-version re-confirmation is a proposed policy choice.
- The design does not provide legal certification or guarantee universal DPDP/GDPR compliance.

## 📬 Project Information

- **Project:** ConsentGuard
- **Problem Statement:** PS-002 — Purpose-Based Consent Recording & Withdrawal
- **Category:** Cyber Security
- **Contact:** [untitled.sahil@gmail.com](mailto:untitled.sahil@gmail.com)
- **Date:** 14 October 2026

## 🎬 Introduction Video

[▶️ Watch the ConsentGuard Introduction on YouTube](https://youtu.be/X_m1AcXzKdA)

---

<p align="center">
  <strong>ConsentGuard</strong><br>
  Purpose-based consent recording, enforcement and auditability.
</p>

<p align="center">
  <em>Proposed design — not a deployed or legally certified system.</em>
</p>
