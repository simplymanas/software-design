# 💸 How UPI Works — Complete System Design Guide

> **Study Guide** | Author: Manas Ranjan Dash | Level: Senior → Staff Engineer
>
> A first-principles deep dive into the Unified Payments Interface (UPI) — India's real-time payment backbone powering Google Pay, PhonePe, Paytm and 50+ apps. Covers the NPCI architecture, transaction flows, VPA resolution, 2FA, idempotency, fraud detection, and designing a UPI-like system at 10 billion transactions/month.

---

## 📋 Table of Contents

1. [What is UPI?](#1-what-is-upi)
2. [The Ecosystem — Players & Roles](#2-the-ecosystem--players--roles)
3. [Core Concepts](#3-core-concepts)
4. [How a UPI Payment Works — Step by Step](#4-how-a-upi-payment-works--step-by-step)
5. [UPI Architecture — Deep Dive](#5-upi-architecture--deep-dive)
6. [VPA Resolution System](#6-vpa-resolution-system)
7. [Authentication & Security](#7-authentication--security)
8. [How Google Pay Works on UPI](#8-how-google-pay-works-on-upi)
9. [How PhonePe Works on UPI](#9-how-phonepe-works-on-upi)
10. [How Paytm Works on UPI](#10-how-paytm-works-on-upi)
11. [Idempotency & Exactly-Once Payments](#11-idempotency--exactly-once-payments)
12. [Failure Modes & Resilience](#12-failure-modes--resilience)
13. [Fraud Detection at Scale](#13-fraud-detection-at-scale)
14. [Designing a UPI-Like System](#14-designing-a-upi-like-system)
15. [Observability](#15-observability)
16. [Interview Cheat Sheet](#16-interview-cheat-sheet)

---

## 1. What is UPI?

**Unified Payments Interface (UPI)** is a real-time payment system developed by the **National Payments Corporation of India (NPCI)** and launched in 2016. It enables instant money transfers between any two bank accounts using a smartphone — 24x7, 365 days a year, free of charge for customers.

```mermaid
flowchart LR
    USER["User — Manas\n(SBI Account)"] -->|sends 500 via\nGoogle Pay| GPAY["Google Pay\n(PSP App)"]
    GPAY -->|UPI API| NPCI["NPCI Switch\n(Central Router)"]
    NPCI -->|debit| SBI["SBI\n(Payer Bank)"]
    NPCI -->|credit| HDFC["HDFC\n(Payee Bank)"]
    HDFC -->|notification| PAYEE["User — Ravi\n(HDFC Account)"]

    style NPCI fill:#f97316,color:#fff,stroke:#ea580c
    style GPAY fill:#4285f4,color:#fff
    style SBI fill:#1e40af,color:#fff
    style HDFC fill:#15803d,color:#fff
```

### UPI by the Numbers (2025)

| Metric | Value |
|--------|-------|
| Monthly transactions | ~15 billion |
| Monthly value | ~20 lakh crore INR |
| Active PSP apps | 50+ |
| Banks on UPI | 600+ |
| Peak TPS | 5,000+ |
| Average transaction time | < 3 seconds |
| Customer cost | Zero (free for P2P) |

---

## 2. The Ecosystem — Players & Roles

```mermaid
flowchart TB
    subgraph REGULATOR["Regulator"]
        RBI["Reserve Bank of India\nSets rules, licensing"]
    end

    subgraph NPCI_BOX["NPCI — The Backbone"]
        NPCI["National Payments Corporation of India\n• Owns UPI rails\n• Central switch\n• Routing and settlement"]
    end

    subgraph PSP["PSP — Payment Service Providers"]
        GPAY["Google Pay\n(PSP Bank: Axis)"]
        PHONEPE["PhonePe\n(PSP Bank: Yes Bank)"]
        PAYTM["Paytm\n(PSP Bank: Paytm Payments Bank)"]
        BHIM["BHIM\n(PSP Bank: NPCI)"]
    end

    subgraph BANKS["Banks"]
        PAYER_BANK["Payer Bank\n(sender's account)"]
        PAYEE_BANK["Payee Bank\n(receiver's account)"]
    end

    RBI --> NPCI_BOX
    NPCI_BOX <--> PSP
    NPCI_BOX <--> BANKS
    PSP --> BANKS

    style NPCI_BOX fill:#f97316,color:#fff
    style PSP fill:#4285f4,color:#fff
    style BANKS fill:#15803d,color:#fff
    style REGULATOR fill:#7c3aed,color:#fff
```

### Role Definitions

| Player | Who | Role |
|--------|-----|------|
| **NPCI** | National body | Owns UPI protocol, central switch, routes all transactions |
| **PSP** | Google Pay, PhonePe, Paytm | User-facing app — collects payment intent, calls UPI APIs |
| **PSP Bank** | Axis (GPay), Yes Bank (PhonePe) | Bank the PSP is registered with; API bridge to NPCI |
| **Payer Bank** | SBI, HDFC, ICICI | Holds sender's account; debits money |
| **Payee Bank** | Any UPI-enabled bank | Holds receiver's account; credits money |

> **Key insight:** Google Pay is NOT a bank. It is an app registered with NPCI via its PSP bank (Axis Bank). When you pay via Google Pay, money moves directly between your bank and the recipient's bank — Google never holds your money.

---

## 3. Core Concepts

### Virtual Payment Address (VPA)

A VPA (also called UPI ID) is a unique identifier that maps to a bank account — like an email address for money.

```
Format: username@psp_handle

Examples:
  manas@oksbi       Google Pay handle for SBI account
  ravi@ybl          PhonePe handle (ybl = Yes Bank Limited)
  priya@paytm       Paytm handle
  9876543210@upi    Phone number-based VPA (BHIM)

Benefits over IFSC + Account Number:
  Human-readable and easily shareable
  One VPA can link to multiple accounts (switch default any time)
  Hides actual account details — privacy by design
  Portable even if you change banks
```

### UPI Transaction Types

| Type | Description | Example |
|------|-------------|---------|
| **P2P Pay** | Person to Person push | Manas sends 500 to Ravi |
| **P2M Collect** | Merchant pulls from customer | Swiggy, Amazon checkout |
| **Collect Request** | Payee requests money from payer | Merchant sends request link |
| **UPI AutoPay** | Recurring mandates | Netflix subscription, SIP |
| **UPI Lite** | Offline small txns under 500 | Quick tap payments |
| **UPI 123PAY** | Feature phone UPI | IVR or missed-call based |

### Transaction Reference Numbers

```
Every UPI transaction gets 3 reference numbers:

1. UPI Transaction ID (UTR)
   12-digit number generated by NPCI
   Used for end-to-end tracking and dispute resolution

2. PSP Reference ID
   Generated by Google Pay / PhonePe / Paytm
   Used for internal tracking in PSP systems

3. Bank Reference Number (RRN)
   Generated by the payer bank
   Used for bank-level reconciliation
```

---

## 4. How a UPI Payment Works — Step by Step

### Scenario: Manas pays 500 to Ravi via Google Pay

Manas VPA: `manas@oksbi` linked to SBI
Ravi VPA: `ravi@ybl` linked to HDFC via PhonePe

```mermaid
sequenceDiagram
    actor M as Manas (Google Pay)
    participant GPay as Google Pay App
    participant GPayPSP as Google PSP Server
    participant NPCI as NPCI Switch
    participant SBI as SBI (Payer Bank)
    participant HDFC as HDFC (Payee Bank)
    actor R as Ravi

    Note over M,R: Phase 1 — VPA Resolution

    M->>GPay: Enter ravi@ybl and 500, tap Pay
    GPay->>GPayPSP: Resolve VPA ravi@ybl
    GPayPSP->>NPCI: /validate-vpa {vpa: "ravi@ybl"}
    NPCI->>HDFC: Who owns ravi@ybl?
    HDFC-->>NPCI: Ravi Kumar, account XXXX4521
    NPCI-->>GPayPSP: VPA valid
    GPayPSP-->>GPay: Display "Ravi Kumar" to Manas

    Note over M,R: Phase 2 — Authentication

    M->>GPay: Confirm payee, enter UPI PIN
    GPay->>GPay: Encrypt PIN with SBI public key on-device
    GPay->>GPayPSP: {txn data, encrypted_pin}

    Note over M,R: Phase 3 — Transaction Initiation

    GPayPSP->>NPCI: /pay {from: manas@oksbi, to: ravi@ybl, amount: 500, enc_pin}
    NPCI->>SBI: Debit 500 from Manas (verify PIN, check balance)
    SBI->>SBI: Decrypt PIN OK, Balance OK, Debit 500

    Note over M,R: Phase 4 — Credit and Settlement

    SBI-->>NPCI: Debit SUCCESS, RRN: RRN123
    NPCI->>HDFC: Credit 500 to ravi@ybl
    HDFC->>HDFC: Credit 500 to Ravi account
    HDFC-->>NPCI: Credit SUCCESS
    NPCI-->>GPayPSP: Transaction SUCCESS, UTR: 421234567890
    GPayPSP-->>GPay: Payment Successful
    GPay-->>M: 500 sent to Ravi Kumar
    HDFC-->>R: SMS + notification: 500 credited

    Note over M,R: Total time: 2 to 3 seconds
```

### The 8 Key Steps

```
Step 1   Manas opens Google Pay, enters Ravi's VPA and 500
Step 2   Google Pay resolves the VPA — confirms "Ravi Kumar" at HDFC
Step 3   Manas reviews and enters his 6-digit UPI PIN
Step 4   PIN is encrypted on-device using SBI's public key (never seen by Google)
Step 5   Google Pay sends the encrypted transaction to NPCI
Step 6   NPCI forwards debit request to SBI — SBI decrypts PIN, checks balance, debits
Step 7   NPCI forwards credit request to HDFC — HDFC credits Ravi's account
Step 8   Success confirmation flows back — both users notified in under 3 seconds
```

---

## 5. UPI Architecture — Deep Dive

```mermaid
flowchart TB
    subgraph PSP_LAYER["PSP Layer"]
        APP["Customer App\nGoogle Pay / PhonePe / Paytm"]
        PSP_SERVER["PSP Application Server\n• VPA management\n• Transaction initiation\n• Notification service"]
        PSP_SDK["UPI SDK on-device\n• PIN encryption\n• Device binding\n• Biometric auth"]
    end

    subgraph NPCI_LAYER["NPCI Layer"]
        UPI_SWITCH["UPI Central Switch\n• Routing\n• Transaction orchestration\n• Deduplication"]
        MAPPER["NPCI Mapper\n• VPA to bank mapping\n• Account registry"]
        SETTLEMENT["Settlement Engine\n• Batch settlement\n• Net position calculation"]
        FRAUD["Fraud and Risk Engine\n• Real-time ML scoring\n• Velocity checks"]
    end

    subgraph BANK_LAYER["Bank Layer"]
        CBS["Core Banking System\n• Account debit and credit\n• PIN verification\n• Balance check"]
        HSM["Hardware Security Module\n• PIN decryption\n• Key management"]
        BANK_NOTIF["Bank Notification\n• SMS gateway\n• Push notifications"]
    end

    APP --> PSP_SDK
    APP --> PSP_SERVER
    PSP_SERVER <-->|ISO 8583 or REST| UPI_SWITCH
    UPI_SWITCH <--> MAPPER
    UPI_SWITCH --> FRAUD
    UPI_SWITCH <-->|ISO 8583| CBS
    CBS <--> HSM
    CBS --> BANK_NOTIF
    UPI_SWITCH --> SETTLEMENT

    style NPCI_LAYER fill:#f97316,color:#fff
    style PSP_LAYER fill:#4285f4,color:#fff
    style BANK_LAYER fill:#15803d,color:#fff
```

### NPCI UPI APIs — Key Endpoints

```
All PSPs call NPCI over mTLS with ISO 8583 or XML/JSON

Core APIs:

ReqValAdd       Validate a VPA (resolve virtual address to account)
ReqPay          Initiate payment — P2P push or P2M
ReqCollect      Send collect request — pull payment from payer
ReqAuthDetails  Request device authentication challenge
ReqRegMob       Register mobile number with UPI
ReqOtp          Request OTP for bank account linking
ReqSetCre       Set or change UPI PIN
ReqVerifyOtp    Verify OTP during onboarding
ReqTxnStatus    Check transaction status (polling)
ReqMandate      Create recurring payment mandate for AutoPay

Response codes:
  00  Success
  01  Pending — check again
  BT  Bank Timeout
  Z9  Bank Declined
  XH  Invalid VPA
  YF  Insufficient Funds
```

### Settlement Flow

UPI is real-time for customers but settlement between banks is batch-based:

```mermaid
flowchart LR
    subgraph REALTIME["Real-Time T+0"]
        RT1["Customer sees money moved"]
        RT2["Banks record the obligation"]
    end

    subgraph BATCH["Batch Settlement T+0 multiple times per day"]
        B1["NPCI calculates net positions across all banks"]
        B2["SBI owes HDFC 10Cr, HDFC owes ICICI 5Cr\nNet: SBI pays 5Cr to ICICI"]
        B3["Settlement via RBI RTGS rails"]
    end

    REALTIME --> BATCH

    style REALTIME fill:#22c55e,color:#fff
    style BATCH fill:#f97316,color:#fff
```

---

## 6. VPA Resolution System

The NPCI Mapper is a distributed registry mapping every VPA to its bank and account number.

```mermaid
flowchart TB
    subgraph MAPPER["NPCI Mapper Architecture"]
        direction LR
        M1["Shard 1\nVPAs: a to f"]
        M2["Shard 2\nVPAs: g to n"]
        M3["Shard 3\nVPAs: o to z"]
        CACHE["Redis Cluster\nTTL: 5 minutes"]
    end

    PSP["PSP Server"] -->|ReqValAdd: ravi@ybl| CACHE
    CACHE -->|miss| M2
    M2 -->|bank: HDFC, account: XXXX4521| CACHE
    CACHE -->|response| PSP

    style MAPPER fill:#f97316,color:#fff
```

### VPA Onboarding — First-Time Setup

```mermaid
sequenceDiagram
    actor U as User
    participant APP as PhonePe App
    participant PSP as PhonePe Server
    participant NPCI as NPCI
    participant BANK as User Bank (HDFC)

    U->>APP: Open PhonePe, add bank account
    APP->>APP: Read SIM to detect mobile number
    APP->>PSP: Register mobile 9876543210
    PSP->>NPCI: ReqRegMob {mobile, psp: phonepe}
    NPCI->>BANK: Verify mobile is linked to account
    BANK-->>NPCI: Account XXXX4521 confirmed
    NPCI-->>PSP: OTP sent to mobile

    U->>APP: Enter OTP
    APP->>PSP: Verify OTP
    PSP->>NPCI: ReqVerifyOtp
    NPCI-->>PSP: OTP Valid

    U->>APP: Set 6-digit UPI PIN
    APP->>APP: Encrypt PIN with bank public key on-device
    APP->>PSP: ReqSetCre with encrypted PIN
    PSP->>NPCI: Forward to HDFC
    NPCI->>BANK: Store PIN hash in HSM
    BANK-->>NPCI: PIN set
    NPCI-->>PSP: VPA created: 9876543210@ybl
    PSP-->>APP: Ready to pay
```

---

## 7. Authentication & Security

### Two-Factor Authentication

UPI mandates 2FA for every transaction:

```
Factor 1 — Device Binding (something you HAVE)
  Mobile number verified via OTP at setup
  Device fingerprint (IMEI + SIM) registered with NPCI
  Each device gets a unique Device ID stored in NPCI Mapper
  Transaction rejected if device ID does not match registered device

Factor 2 — UPI PIN (something you KNOW)
  6-digit PIN set by user
  PIN NEVER leaves the device in plaintext
  Encrypted on-device using the bank's RSA public key
  Decrypted only inside the bank's Hardware Security Module
  NPCI, Google, and PhonePe can never see your PIN
```

### PIN Encryption Flow

```mermaid
flowchart LR
    subgraph DEVICE["On Device — Secure"]
        PIN["User enters PIN: 123456"]
        PUBKEY["Bank RSA Public Key\ndownloaded at setup"]
        ENC["Encrypted PIN block\nPIN + PAN + random nonce\nRSA-2048 encrypted"]
        PIN --> ENC
        PUBKEY --> ENC
    end

    subgraph BANK["Bank HSM — Only Decryption Point"]
        HSM_DEC["HSM decrypts\nusing private key"]
        VERIFY["Verify PIN against stored hash"]
        RESULT["Authorized or Declined"]
        HSM_DEC --> VERIFY --> RESULT
    end

    ENC -->|passes through NPCI untouched| HSM_DEC

    style DEVICE fill:#4285f4,color:#fff
    style BANK fill:#15803d,color:#fff
```

### Security Layers

| Layer | Mechanism | Protects Against |
|-------|-----------|-----------------|
| Device Binding | IMEI + SIM + Device ID | SIM swap, unauthorized devices |
| UPI PIN | RSA-2048, HSM decryption | Interception, PSP breach |
| mTLS | Mutual TLS — PSP to NPCI to Bank | Man-in-the-middle |
| Transaction Limits | 1 lakh per txn, 2 lakh per day | Damage limitation |
| Collect Approval | Collect requests need explicit PIN | Phishing-based fraud |
| Fraud Engine | Real-time ML scoring | Anomaly and velocity fraud |

---

## 8. How Google Pay Works on UPI

```mermaid
flowchart TB
    subgraph GPAY["Google Pay Architecture"]
        APP["GPay App\n• UPI SDK\n• Contacts integration\n• QR scanner"]

        subgraph BACKEND["GPay Backend — Google Cloud"]
            API_GW["API Gateway"]
            TXN_SVC["Transaction Service"]
            VPA_SVC["VPA Management"]
            NOTIF["Notification Service FCM"]
            REWARDS["Rewards Engine\nScratch cards, cashback"]
            ANALYTICS["Analytics — BigQuery"]
        end

        PSP_BANK["PSP Bank: Axis Bank\nAPI bridge to NPCI"]
    end

    APP --> API_GW
    API_GW --> TXN_SVC
    API_GW --> VPA_SVC
    TXN_SVC --> PSP_BANK --> NPCI["NPCI Switch"]
    TXN_SVC --> NOTIF
    TXN_SVC --> REWARDS
    TXN_SVC --> ANALYTICS

    style GPAY fill:#4285f4,color:#fff
```

### Google Pay VPA Handles

```
manas@okaxis      User linked Axis Bank account via GPay
manas@okhdfc      User linked HDFC account via GPay
manas@oksbi       User linked SBI account via GPay

Format: username@ok{bankcode}
  ok prefix = Google Pay identifier
  bankcode = sbi, hdfc, icici, axis, etc.
```

### Google Pay Unique Features

```
Rewards Engine
  Scratch cards triggered post-payment
  Cashback funded by Google merchant partnerships
  Runs entirely outside UPI rails

NFC Payments
  Uses tokenized card, not bank account
  Separate from UPI — uses Visa or Mastercard network

Merchant Analytics
  GST-linked receipts
  Revenue dashboard for business accounts

Contacts-First UX
  Pay using phone contacts directly
  No need to know the VPA — Google resolves it
```

---

## 9. How PhonePe Works on UPI

```mermaid
flowchart TB
    subgraph PHONEPE["PhonePe Architecture"]
        APP["PhonePe App\n• UPI payments\n• Insurance\n• Mutual Funds\n• Recharges"]

        subgraph BACKEND["PhonePe Backend — AWS India"]
            API_GW["API Gateway — Kong"]
            TXN_SVC["Transaction Orchestrator"]
            SWITCH["PhonePe Payment Switch\n• Primary: Yes Bank\n• Failover: ICICI Bank"]
            RISK["Risk Engine — Real-time ML"]
        end
    end

    APP --> API_GW
    API_GW --> TXN_SVC
    TXN_SVC --> SWITCH
    TXN_SVC --> RISK
    SWITCH --> NPCI["NPCI Switch"]

    style PHONEPE fill:#5f259f,color:#fff
```

### PhonePe VPA Handles

```
@ybl    Yes Bank Limited — primary PhonePe handle
@ibl    IndusInd Bank via PhonePe
@axl    Axis Bank via PhonePe

Example: 9876543210@ybl
```

### PhonePe's Payment Switch — Key Advantage

PhonePe built an internal payment switch that:

```
1. Manages connections to multiple PSP banks simultaneously
2. Auto-fails over from Yes Bank to ICICI if primary is slow or down
3. Load balances transactions across bank connections
4. Maintains transaction state for retry and reconciliation
5. Handles collect flows for merchant integrations

This dual-PSP-bank architecture gives PhonePe better uptime
than single-bank PSPs — a key reason for its 47% market share.
```

---

## 10. How Paytm Works on UPI

Paytm is unique — it operates its own Payments Bank, making it both a PSP and a bank.

```mermaid
flowchart TB
    subgraph PAYTM["Paytm Architecture"]
        APP["Paytm Super App\n• UPI payments\n• Paytm Wallet\n• FASTag\n• Lending, Insurance"]

        subgraph BACKEND["Paytm Backend"]
            API_GW["API Gateway"]
            TXN_SVC["Transaction Service"]
            WALLET["Paytm Wallet\nPrepaid Instrument"]
            PAYMENTS_BANK["Paytm Payments Bank\nRBI Licensed\n• Savings accounts\n• UPI PSP Bank"]
            RISK["Fraud and Risk Engine"]
        end
    end

    APP --> API_GW
    API_GW --> TXN_SVC
    TXN_SVC --> WALLET
    TXN_SVC --> PAYMENTS_BANK
    PAYMENTS_BANK <-->|direct| NPCI["NPCI Switch"]

    style PAYTM fill:#00baf2,color:#fff
```

### Paytm Dual Mode — Wallet vs UPI

```
Mode 1: Paytm Wallet (prepaid instrument)
  Money stored inside Paytm's own system
  Regulated as PPI (Prepaid Payment Instrument) by RBI
  Instant wallet-to-wallet — no UPI round-trip
  Limits: 10,000/month without KYC, 1 lakh with full KYC

Mode 2: UPI via Paytm Payments Bank
  User links their SBI/HDFC/etc. account
  Money moves directly between banks via NPCI
  Paytm acts as PSP — never holds the money
  VPA: 9876543210@paytm

Merchant advantage:
  Merchant can choose to receive in Paytm Wallet (instant)
  or bank account (UPI settlement cycle)
```

### PSP Comparison

| Feature | Google Pay | PhonePe | Paytm |
|---------|-----------|---------|-------|
| PSP Bank | Axis Bank | Yes Bank | Paytm Payments Bank |
| VPA handle | @oksbi, @okhdfc | @ybl, @ibl | @paytm |
| Has Wallet | No | No | Yes |
| Is a Bank | No | No | Yes (Payments Bank) |
| Market share (volume) | ~37% | ~47% | ~8% |
| Backend Cloud | Google Cloud | AWS | Own DC + Cloud |
| PSP Bank Failover | No | Yes — ICICI backup | Not applicable |
| Super App | No | Yes | Yes |

---

## 11. Idempotency & Exactly-Once Payments

> The hardest problem in payments. A payment must execute exactly once — never zero times, never twice.

### The Problem

```mermaid
sequenceDiagram
    participant APP as PhonePe App
    participant NPCI as NPCI Switch
    participant SBI as SBI Bank

    APP->>NPCI: Pay 500 with txn_id TXN001
    NPCI->>SBI: Debit 500
    SBI->>SBI: Debited 500 successfully
    SBI-->>NPCI: SUCCESS

    Note over NPCI,APP: Network timeout — response lost

    NPCI--xAPP: Response never arrives

    APP->>APP: Unknown outcome — did it go through?

    APP->>NPCI: Retry Pay 500 with new txn_id TXN002 — WRONG
    NPCI->>SBI: Debit 500 again
    SBI-->>NPCI: Debited again — double charge
```

### Solution: Transaction State Machine

```mermaid
stateDiagram-v2
    [*] --> INITIATED : User taps Pay

    INITIATED --> PENDING : Sent to NPCI
    PENDING --> SUCCESS : Bank confirms debit and credit
    PENDING --> FAILED : Bank declines or timeout confirmed
    PENDING --> PENDING : Poll for status every 10 seconds

    SUCCESS --> [*] : Show success to user
    FAILED --> [*] : Show failure, allow fresh retry

    note right of PENDING
        Danger zone.
        Network failures happen here.
        Never re-initiate.
        Only poll for status.
    end note
```

```python
class UPITransactionService:

    def initiate_payment(self, payer_vpa, payee_vpa, amount, user_ref):
        # Generate deterministic idempotency key from user intent
        txn_id = self.generate_txn_id(user_ref, payer_vpa, payee_vpa, amount)

        # Check if already exists — prevent duplicate initiation
        existing = self.db.get_transaction(txn_id)
        if existing:
            return existing.status

        # Persist INITIATED state BEFORE calling NPCI
        self.db.create_transaction({
            "txn_id": txn_id,
            "status": "INITIATED",
            "payer": payer_vpa,
            "payee": payee_vpa,
            "amount": amount,
        })

        try:
            response = self.npci_client.pay(txn_id=txn_id,
                                            payer=payer_vpa,
                                            payee=payee_vpa,
                                            amount=amount)

            if response.status == "SUCCESS":
                self.db.update_transaction(txn_id, "SUCCESS", response.utr)
                return "SUCCESS"

            elif response.status == "PENDING":
                self.start_status_poller(txn_id)
                return "PENDING"

            else:
                self.db.update_transaction(txn_id, "FAILED")
                return "FAILED"

        except NetworkTimeout:
            # Do NOT retry — poll instead
            self.start_status_poller(txn_id)
            return "PENDING"

    def poll_transaction_status(self, txn_id):
        """Async job — polls every 10s for up to 5 minutes."""
        response = self.npci_client.check_status(txn_id)

        if response.status in ["SUCCESS", "FAILED"]:
            self.db.update_transaction(txn_id, response.status)
            self.notify_user(txn_id, response.status)
            return

        self.schedule_next_poll(txn_id, delay_seconds=10)
```

### The Deemed Success Rule

```
If NPCI cannot confirm SUCCESS or FAILURE within the timeout window,
and the payer's account HAS been debited — it is a "Deemed Success"

NPCI guarantees:
  If SBI debited the money, the payee WILL be credited — even if delayed
  Maximum credit delay: T+1 (next business day)
  If credit ultimately fails: automatic refund within T+5 business days

This is why you sometimes see "Payment Pending" —
the system is working through a deemed success scenario.
```

---

## 12. Failure Modes & Resilience

```mermaid
flowchart TD
    subgraph FAILURES["Failure Scenarios"]
        F1["App crash mid-payment"]
        F2["PSP server outage"]
        F3["NPCI overload on festival day"]
        F4["Bank CBS timeout"]
        F5["Network partition — NPCI to Bank"]
        F6["User taps Pay twice"]
    end

    subgraph SOLUTIONS["Resilience Strategies"]
        S1["State machine plus polling\nShow Payment Pending"]
        S2["Multi-PSP-bank failover\nPhonePe: Yes Bank + ICICI"]
        S3["Queue plus circuit breakers\nRate limiting at NPCI ingress"]
        S4["Async debit with timeout\nStatus polling job"]
        S5["Idempotency keys\nExactly-once guarantee"]
        S6["Deduplication window\nSame txn_id is a no-op"]
    end

    F1 --> S1
    F2 --> S2
    F3 --> S3
    F4 --> S4
    F5 --> S5
    F6 --> S6
```

### Refund Flow

```mermaid
sequenceDiagram
    participant USER as User
    participant PSP as Google Pay
    participant NPCI as NPCI
    participant PAYER_BANK as SBI
    participant PAYEE_BANK as HDFC

    USER->>PSP: Request refund — UTR 421234567890
    PSP->>NPCI: ReqPay reverse {original_utr}
    NPCI->>HDFC: Debit 500 from Ravi (reversal)
    HDFC-->>NPCI: Debit SUCCESS
    NPCI->>SBI: Credit 500 back to Manas
    SBI-->>NPCI: Credit SUCCESS
    NPCI-->>PSP: Refund SUCCESS
    PSP-->>USER: 500 refunded

    Note over USER,PAYEE_BANK: Refund is a brand-new UPI transaction in reverse
    Note over USER,PAYEE_BANK: Timeline: instant to 5 business days depending on bank
```

---

## 13. Fraud Detection at Scale

At 15 billion transactions per month (~5,800 TPS average, 10,000+ TPS peak), fraud decisions must be made in real-time under 100ms.

```mermaid
flowchart LR
    TXN["Incoming Transaction"] --> FE["Fraud Engine\nunder 50ms decision"]

    subgraph SIGNALS["Risk Signals"]
        V1["Velocity: transactions per minute per user"]
        V2["Device fingerprint: new device?"]
        V3["Geographic anomaly: Bengaluru then Russia in 1hr"]
        V4["Payee reputation: new VPA, flagged account?"]
        V5["Transaction pattern: unusual amount or time"]
        V6["Graph signal: connected to known fraud accounts?"]
    end

    SIGNALS --> FE

    FE -->|score below 30| ALLOW["Allow"]
    FE -->|score 30 to 70| STEP_UP["Step-up auth — additional OTP"]
    FE -->|score above 70| BLOCK["Block and Alert"]

    style FE fill:#ef4444,color:#fff
    style ALLOW fill:#22c55e,color:#fff
    style BLOCK fill:#7f1d1d,color:#fff
```

### Common UPI Fraud Patterns

| Fraud Type | How It Works | Mitigation |
|-----------|-------------|-----------|
| **Collect Request Fraud** | Fraudster sends collect request posing as a bank | Red warning UI for collect requests |
| **SIM Swap** | Attacker clones SIM to take over phone number | Device binding + 7-day cooling period |
| **Vishing** | Caller tricks user into sharing OTP or PIN | UPI PIN is never shareable by design |
| **Fake QR Code** | Sticker placed over merchant QR code | Merchant name shown before payment |
| **Money Mule** | Stolen money routed through innocent accounts | Graph analysis, rapid velocity flags |

### Real-Time Risk Engine

```python
class UPIRiskEngine:
    """
    Must complete in under 50ms.
    Uses pre-computed features from Redis — no DB round-trips.
    """
    def __init__(self):
        self.redis = RedisCluster(...)
        self.ml_model = load_model("xgb_fraud_v12")  # XGBoost

    def score_transaction(self, txn):
        features = self.extract_features(txn)
        score = self.ml_model.predict(features)

        if score > 70:
            return RiskDecision(action="BLOCK", score=score)
        elif score > 30:
            return RiskDecision(action="STEP_UP", score=score)
        return RiskDecision(action="ALLOW", score=score)

    def extract_features(self, txn):
        # All reads from Redis — pre-computed velocity counters
        return {
            "txns_last_1min":    self.redis.get(f"vel:1m:{txn.payer}"),
            "txns_last_1hr":     self.redis.get(f"vel:1h:{txn.payer}"),
            "amount_zscore":     self.compute_zscore(txn),
            "payee_vpa_age":     self.redis.get(f"vpa:age:{txn.payee}"),
            "device_seen_before":self.redis.get(f"dev:{txn.device_id}"),
            "is_new_payee":      not self.redis.get(f"rel:{txn.payer}:{txn.payee}"),
            "hour_of_day":       txn.timestamp.hour,
            "is_weekend":        txn.timestamp.weekday() >= 5,
        }
```

---

## 14. Designing a UPI-Like System

### System Requirements

```
Functional:
  Register users with bank accounts and assign VPAs
  P2P payments via VPA push and pull
  Merchant payments via QR code
  Transaction history and dispute management
  Refunds

Non-Functional:
  10,000 TPS peak throughput
  p99 latency under 3 seconds end-to-end
  99.99% availability — under 52 minutes downtime per year
  Exactly-once payment guarantee
  Strong security with 2FA and encrypted PIN
```

### High-Level Design

```mermaid
flowchart TB
    subgraph CLIENT["Client"]
        APP["Mobile App\niOS and Android"]
        SDK["UPI SDK\nPIN encryption, device binding"]
    end

    subgraph GATEWAY["API Gateway"]
        GW["Kong or Nginx\n• Auth\n• Rate limiting\n• TLS termination"]
    end

    subgraph SERVICES["Core Microservices"]
        USER_SVC["User Service\n• VPA management\n• Account linking\n• Device registry"]
        TXN_SVC["Transaction Orchestrator\n• State machine\n• Idempotency\n• Timeout handling"]
        RISK_SVC["Risk Engine\n• Real-time scoring\n• Velocity checks"]
        NOTIF_SVC["Notification Service\n• Push via FCM and APNs\n• SMS gateway"]
        MAPPER_SVC["VPA Mapper\n• VPA to bank routing\n• Cache layer"]
    end

    subgraph DATA["Data Layer"]
        POSTGRES["PostgreSQL\nTransactions, users\nRead replicas"]
        REDIS["Redis Cluster\nVPA cache, velocity\nSession, idempotency keys"]
        KAFKA["Kafka\nEvent streaming\nAsync notifications"]
        CASSANDRA["Cassandra\nTransaction history at scale"]
    end

    subgraph BANK_CONN["Bank Connectivity"]
        BANK_ADAPTER["Bank Adapter\n• ISO 8583 and REST\n• Per-bank connectors\n• Retry and circuit breaker"]
        HSM_CONN["HSM Interface\n• PIN verification\n• Key management"]
    end

    CLIENT --> GATEWAY
    GATEWAY --> SERVICES
    SERVICES --> DATA
    SERVICES --> BANK_CONN
    TXN_SVC --> KAFKA
    KAFKA --> NOTIF_SVC

    style SERVICES fill:#f97316,color:#fff
    style DATA fill:#4285f4,color:#fff
    style BANK_CONN fill:#15803d,color:#fff
```

### Core Database Schema

```sql
-- VPA Registry
CREATE TABLE vpa_registry (
    vpa          VARCHAR(100) PRIMARY KEY,
    user_id      UUID NOT NULL,
    bank_code    VARCHAR(20) NOT NULL,
    account_num  VARCHAR(50) NOT NULL,   -- encrypted at rest
    ifsc         VARCHAR(11) NOT NULL,
    is_default   BOOLEAN DEFAULT FALSE,
    created_at   TIMESTAMP DEFAULT NOW()
);

-- Transactions — high write volume table
CREATE TABLE transactions (
    txn_id       VARCHAR(50) PRIMARY KEY,   -- idempotency key
    utr          VARCHAR(20) UNIQUE,        -- NPCI reference
    payer_vpa    VARCHAR(100) NOT NULL,
    payee_vpa    VARCHAR(100) NOT NULL,
    amount       BIGINT NOT NULL,           -- in paise: 500 INR = 50000
    status       VARCHAR(20) NOT NULL,      -- INITIATED, PENDING, SUCCESS, FAILED
    created_at   TIMESTAMP DEFAULT NOW(),
    updated_at   TIMESTAMP DEFAULT NOW()
);

-- Device Registry for 2FA
CREATE TABLE device_registry (
    device_id    VARCHAR(100) PRIMARY KEY,
    user_id      UUID NOT NULL,
    sim_hash     VARCHAR(64),
    registered_at TIMESTAMP DEFAULT NOW(),
    is_active    BOOLEAN DEFAULT TRUE
);
```

### Capacity Estimates

```
Peak TPS: 10,000 transactions per second

DB writes per transaction: 3 (create + 2 status updates)
Peak DB write load: 30,000 writes/sec
Requires: write-sharded PostgreSQL or Cassandra for transactions

VPA lookups: 10,000 per second (one per transaction)
Cache hit rate target: 95%
Cache handles: 9,500 RPS
DB handles: 500 RPS

Transaction storage:
  1 row ~ 500 bytes
  10,000 TPS x 86,400 seconds = 864M transactions per day
  864M x 500 bytes = 432 GB per day
  Partition by date, archive to cold storage after 90 days

Redis for velocity counters:
  50M active users x 10 counters x 32 bytes = 16 GB
  Fits comfortably in a 3-shard Redis Cluster
```

---

## 15. Observability

### Key Metrics

```mermaid
mindmap
  root((UPI Observability))
    Transaction Metrics
      Success rate percent
      Failure breakdown by code
      Pending transactions over 30 seconds
      p50 p95 p99 latency
    Bank Metrics
      Per-bank success rate
      Per-bank latency
      CBS timeout rate
    Fraud Metrics
      Block rate
      Step-up auth rate
      False positive rate
    Infrastructure
      NPCI API latency
      Redis cache hit rate
      Kafka consumer lag
      DB replication lag
```

### Alert Rules

```yaml
groups:
  - name: upi_payments
    rules:

      - alert: PaymentSuccessRateLow
        expr: |
          rate(upi_transactions_total{status="SUCCESS"}[5m]) /
          rate(upi_transactions_total[5m]) < 0.95
        for: 2m
        severity: critical
        summary: "UPI success rate below 95%"

      - alert: PaymentLatencyHigh
        expr: |
          histogram_quantile(0.99,
            rate(upi_transaction_duration_seconds_bucket[5m])
          ) > 3.0
        for: 1m
        severity: critical
        summary: "UPI p99 latency above 3 seconds"

      - alert: PendingTransactionsSpike
        expr: upi_pending_transactions_count > 10000
        for: 2m
        severity: warning
        summary: "High pending backlog — possible NPCI or bank issue"

      - alert: BankTimeoutRateHigh
        expr: |
          rate(upi_bank_timeouts_total[5m]) /
          rate(upi_bank_requests_total[5m]) > 0.02
        for: 1m
        severity: critical
        summary: "Bank timeout rate above 2%"
```

---

## 16. Interview Cheat Sheet

### The Golden Mental Model

```
UPI     = a protocol + switch (NPCI) connecting any bank to any other bank
PSP     = the app — Google Pay, PhonePe — never holds your money
VPA     = human-readable alias for a bank account, like email for payments
Flow    = App → PSP Server → NPCI Switch → Payer Bank → Payee Bank
2FA     = Device binding (HAVE) + UPI PIN (KNOW) — PIN encrypted on device only
Settlement = instant for users, net batch settlement between banks via RBI
```

### Flow in 5 Steps

```
1. RESOLVE    VPA lookup: ravi@ybl maps to Ravi Kumar at HDFC XXXX4521
2. AUTH       User enters PIN — encrypted on-device with bank's RSA public key
3. INITIATE   PSP sends ReqPay to NPCI with idempotency key
4. ROUTE      NPCI debits payer bank, credits payee bank
5. NOTIFY     SMS and push notification to both parties within 3 seconds
```

### Common Interview Questions

| Question | Key Answer |
|----------|-----------|
| What is a VPA? | Virtual Payment Address — human-readable alias like manas@oksbi that maps to a bank account in NPCI Mapper |
| Does Google Pay hold money? | No. GPay is a PSP — a UI layer over UPI. Money moves bank to bank via NPCI |
| How is UPI PIN secured? | Encrypted on-device with bank's RSA public key. Decrypted only in bank's HSM. NPCI and PSPs never see it |
| What is 2FA in UPI? | Factor 1: device binding (SIM + IMEI). Factor 2: UPI PIN |
| How is double payment prevented? | Idempotency key (txn_id) — same txn_id is a no-op at NPCI |
| What happens on network timeout? | Move to PENDING, start polling — never re-initiate a new transaction |
| What is deemed success? | If payer is debited but credit status is unknown — NPCI guarantees credit will happen |
| How does settlement work? | Real-time for users, banks settle net positions via NPCI batch multiple times per day |
| Difference between PhonePe and Paytm? | PhonePe is a pure PSP via Yes Bank. Paytm owns its Payments Bank — it is both PSP and bank |
| How does collect request work? | Payee sends request, payer receives notification, payer must approve and enter PIN |
| How does UPI handle 10k TPS? | Distributed NPCI switch, Redis for VPA cache, async notifications via Kafka, per-bank connection pools |
| What is UPI Lite? | Offline payments under 500 INR using on-device balance — no PIN and no NPCI round-trip required |

### Numbers to Remember

| Metric | Value |
|--------|-------|
| Monthly UPI transactions | ~15 billion |
| Monthly UPI value | ~20 lakh crore INR |
| Max P2P transaction | 1 lakh INR |
| Daily limit (most banks) | 2 lakh INR |
| Wrong PIN attempts before lock | 3 |
| Lock duration after 3 wrong PINs | 24 hours |
| PhonePe market share | ~47% |
| Google Pay market share | ~37% |
| Average transaction time | under 3 seconds |
| UPI uptime SLA | 99.9% |
| Banks on UPI | 600+ |

---

## Further Reading

- [NPCI UPI Product Overview](https://www.npci.org.in/what-we-do/upi/product-overview)
- [UPI 2.0 Features — Overdraft, Signed QR, One-Time Mandates](https://www.npci.org.in/what-we-do/upi/upi2.0)
- [PhonePe Engineering Blog](https://engineering.phonepe.com/)
- [RBI Payment Vision Document](https://www.rbi.org.in)
- [ISO 8583 — Financial Message Standard](https://en.wikipedia.org/wiki/ISO_8583)
- [UPI Circular — RBI Guidelines](https://www.rbi.org.in/Scripts/BS_PressReleaseDisplay.aspx)

---

*Study guide by Manas Ranjan Dash · Part of the [software-design](https://github.com/simplymanas/software-design) series*
*Prev: How LLMs Work | Next: Notification System*
