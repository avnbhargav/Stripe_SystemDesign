# 💳 System Design & Distributed Systems Interview Prep
### *Fintech Engine Architecture, Failure Modes & Production Code (Stripe & FAANG Edition)*

This repository contains in-depth theoretical analysis, high-level system designs (HLD), database schemas, and clean low-level implementations (LLD in Python & Java) for **50 core backend and payment system design challenges**.

The curriculum is engineered to build deep intuition for financial correctness, ACID guarantees, zero-data-loss durability, idempotency, and high-throughput distributed resiliency required for Senior and Staff Software Engineering interviews at Stripe, Adyen, Block, PayPal, and FAANG companies.

---

## 🛠️ Tech Stack & Language Support

- **Languages:** Python 3.10+, Java 17+
- **Databases & Storage:** PostgreSQL, Redis, Apache Kafka
- **Patterns Covered:** Distributed Locks, Double-Entry Accounting, Transactional Outbox, Event Sourcing, Circuit Breakers, HMAC Verification, Token Vaulting, Sliding Window Rate Limiting.

---

## 🗺️ Master Curriculum & Progress

### 💳 Part 1: Core Payment & Ledger Systems
- [ ] **01.** Design a Core Payment Gateway API
- [ ] **02.** Design a Distributed Double-Entry Ledger System
- [x] **03.** [Design an Idempotency Framework for a Payments API](#question-3-design-an-idempotency-framework) *(Completed)*
- [ ] **04.** Design a Payment Intent State Machine
- [ ] **05.** Design a Real-Time Foreign Exchange (FX) Ledger
- [ ] **06.** Design an Active-Active Multi-Region Ledger
- [ ] **07.** Design a Merchant Payout/Settlement Engine
- [ ] **08.** Design a High-Throughput Payment Batch Processing System
- [ ] **09.** Design an Instant Payout System (Visa Direct / Mastercard Send)
- [ ] **10.** Design a Multi-Processor Smart Routing Engine
- [ ] **11.** Design a Partial Refund System
- [ ] **12.** Design a Point-of-Sale (POS) Terminal Syncing System

### 🚀 Part 2: Webhooks, Eventing, & Infrastructure
- [ ] **13.** Design a Scalable Webhook Delivery System
- [ ] **14.** Design an Outbox Pattern Event Publisher
- [ ] **15.** Design a Multi-Tenant Rate Limiter
- [ ] **16.** Design a Circuit Breaker for External Bank APIs
- [ ] **17.** Design a Webhook Security Validation Layer (HMAC Signature)
- [ ] **18.** Design an Event Sourcing Engine for Audit Trails
- [ ] **19.** Design a Transactional Queue Dead-Letter Queue (DLQ) Strategy
- [ ] **20.** Design a Merchant-Facing Event Stream API
- [ ] **21.** Design an Elastic Scaling Strategy for Flash Sales
- [ ] **22.** Design a Multi-Region Data Pinning Infrastructure (GDPR Compliance)

### 🛡️ Part 3: Fraud, Security, & Compliance
- [ ] **23.** Design a Card Tokenization Vault (PCI-DSS Boundary)
- [ ] **24.** Design a Real-Time Fraud Detection Pipeline (<100ms)
- [ ] **25.** Design a Chargeback & Dispute Management System
- [ ] **26.** Design a Secure 3D-Secure (3DS) Redirect Flow
- [ ] **27.** Design an Identity & KYC Document Verification System
- [ ] **28.** Design a Global Sanctions Screening Engine (AML Watchlists)
- [ ] **29.** Design an Encrypted Secret Management Store
- [ ] **30.** Design an Anomalous Activity Detector
- [ ] **31.** Design a Card-Present Fraud Mitigation System

### 📊 Part 4: SaaS Billing, Subscriptions, & Financial Products
- [ ] **32.** Design a Subscription Billing Engine
- [ ] **33.** Design a Metered Usage Billing System
- [ ] **34.** Design a Smart Dunning (Retry) System
- [ ] **35.** Design a Corporate Card Issuing Platform
- [ ] **36.** Design a Merchant Lending Platform
- [ ] **37.** Design an Automated Tax Calculation API
- [ ] **38.** Design a Revenue Recognition Engine
- [ ] **39.** Design a Marketplace Split-Payout System
- [ ] **40.** Design a Coupon and Promotion Management Service
- [ ] **41.** Design a B2B Invoicing Platform

### 📉 Part 5: Data Analytics, Reconciliation, & Developer Platform
- [ ] **42.** Design an Automated Bank Reconciliation Pipeline
- [ ] **43.** Design a Real-Time Financial Analytics Dashboard
- [ ] **44.** Design a High-Throughput CSV Transaction Exporter
- [ ] **45.** Design an API Versioning Infrastructure
- [ ] **46.** Design a Stripe-like "Radar" Rules Engine
- [ ] **47.** Design a Financial Database Archiving Layer
- [ ] **48.** Design a Global Search Engine for Transactions
- [ ] **49.** Design a Sandbox/Testmode Environment Strategy
- [ ] **50.** Design a Discrepancy Alerting Engine

---

## 📌 Module Summaries & Completed Modules

### Question 3: Design an Idempotency Framework

#### Core Theory & Mechanics
Prevents duplicate charges caused by network ambiguity, retries, and dropped responses by establishing mathematical idempotency ($f(f(x)) = f(x)$).

```
[ Client ] ---( 1. POST /v1/payments Header: Idempotency-Key )---> [ Gateway ]
                                                                       │
┌──────────────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────┐
│                                                       IDEMPOTENCY STATE MACHINE                                                             │
├───────────────────────────────────────────────────┬─────────────────────────────────────────────────┬───────────────────────────────────────┤
│ 1. New Request Key                                │ 2. In-Flight Duplicate Key                      │ 3. Completed Key                      │
│ -> Create lock (IN_PROGRESS)                      │ -> Reject with HTTP 409 Conflict                │ -> Replay cached HTTP response        │
│ -> Store SHA-256 Request Hash                     │ -> Prevent concurrent duplicate execution       │ -> Zero downstream execution          │
└───────────────────────────────────────────────────┴─────────────────────────────────────────────────┴───────────────────────────────────────┘
```

#### Storage Schema (PostgreSQL Durability Layer)
```sql
CREATE TABLE idempotency_keys (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(64) NOT NULL,
    idempotency_key VARCHAR(255) NOT NULL,
    request_path VARCHAR(255) NOT NULL,
    request_hash CHAR(64) NOT NULL,
    status VARCHAR(20) NOT NULL CHECK (status IN ('IN_PROGRESS', 'COMPLETED', 'FAILED')),
    response_code INT NULL,
    response_body JSONB NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    CONSTRAINT uk_user_key UNIQUE (user_id, idempotency_key)
);
```

#### Code Implementation
The logic handles request hashing, atomic key locking (SETNX paradigm), payload mismatch detection (HTTP 400), race condition handling (HTTP 409), and cached response replay.

- **Python Implementation:** `idempotency/python/idempotency_manager.py`
- **Java Implementation:** `idempotency/java/IdempotencyFramework.java`

---

## 🚀 How to Run Code Implementations

### Running Python Implementations
```bash
python3 -m idempotency.python.idempotency_manager
```

### Running Java Implementations
```bash
javac idempotency/java/IdempotencyFramework.java
java idempotency.java.IdempotencyFramework
```

---

## 💡 Core System Design Rules to Remember

1. **Failure Boundaries:** Acknowledge requests only when safe durability guarantees are met.
2. **Double-Entry Balance:** Debits must always balance credits to zero in ledger systems ($ \sum \text{Debits} - \sum \text{Credits} = 0 $).
3. **Optimistic Locking:** Utilize version column constraints (`WHERE version = current_version`) for high-concurrency database updates over heavy pessimistic DB locks.
4. **Outbox Pattern:** Avoid Two-Phase Commits (2PC) across service boundaries; write domain events to an outbox table within the local DB transaction and publish asynchronously.