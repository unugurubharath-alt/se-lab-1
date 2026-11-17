# Software Requirements Specification (SRS)
## Problem Statement #35: Customizable Subscription Box Scheduler
**Course**: PES University - Dept. of CSE | Lab 1: Requirements Engineering & UML Use-Case Modelling  
**Domain**: Retail, E-Commerce & Finance  
**Target Stakeholders / Actors**: Subscriber, Fulfillment Lead, Payment Gateway / Billing Service  

---

## 1. Problem Context & Overview

The **Customizable Subscription Box Scheduler** is an e-commerce subscription management platform that enables customers (*Subscribers*) to receive recurring, curated product boxes (e.g., artisanal coffees, boutique books, gourmet snacks) tailored to their personalized preference profiles. Subscribers have the freedom to review provisional box configurations, swap individual items with alternative catalog selections, configure dietary or genre preference tags, and temporarily pause or skip upcoming billing cycles. 

Simultaneously, warehouse operations (*Fulfillment Leads*) require real-time aggregation of locked box orders, deterministic inventory picking lists, batch shipping label generation, and automated lockout schedules to satisfy stringent dispatch SLAs and carrier pickup windows without inventory discrepancies.

---

## 2. Complete Requirements Table

The table below outlines exactly **5 Functional Requirements (FR-001 to FR-005)** and **2 Non-Functional Requirements (NFR-001 & NFR-002)**, each detailing its **ID, Type, Description, Priority, Acceptance Criteria, and Rationale**.

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FR-001** | Functional | The system shall allow subscribers to swap default curated monthly box items with alternative catalog selections or pause deliveries up to 48 hours prior to the monthly billing renewal date. | **High** | **Pass**: Selection updated in database, provisional fulfillment manifest refreshed, and confirmation alert dispatched upon saving swaps prior to the 48-hour cutoff.<br>**Fail**: Customization interface locks out prematurely (> 48 hours before renewal) or changes discarded after user confirmation. | Core value proposition of subscription boxes is personalization; subscribers churn if they receive unwanted repeat or unsuitable items. |
| **FR-002** | Functional | The system shall recommend and dynamically filter curated product selections (e.g., books, coffee roast intensity, dietary snack tags) based on user-defined preference tags and dietary/genre profiles. | **High** | **Pass**: Product catalog dynamically filters items matching chosen preference tags; default monthly box matches tagged criteria; preference tag updates persist across sessions.<br>**Fail**: Recommended box includes items with conflicting tags (e.g., non-vegan snacks for a vegan tag) or preference changes fail to save. | Minimizes customer effort by automatically pre-populating boxes with relevant selections, maximizing engagement and perceived subscription value. |
| **FR-003** | Functional | The system shall allow subscribers to skip an upcoming billing cycle or place their subscription on hold for 1 to 3 consecutive months without cancellation penalties or loss of loyalty tier. | **High** | **Pass**: Next billing renewal date recalculates and shifts forward by the chosen duration, no renewal charge is dispatched to the Payment Gateway, and subscriber status changes to 'Paused'.<br>**Fail**: Account is charged during paused period, or subscription status is erroneously cancelled upon pause request. | Providing self-serve pause/skip options drastically reduces churn caused by temporary travel, surplus inventory, or temporary customer financial constraints. |
| **FR-004** | Functional | The system shall dispatch automated multi-channel notifications (email and SMS/in-app push) to active subscribers at 5 days (T-5) and 48 hours (T-48h cutoff warning) prior to the billing renewal date, detailing provisional box items and the lockout deadline. | **Medium** | **Pass**: Notification triggers at scheduled intervals (T-5 days, T-48h) containing direct deep-links to customization portal with accurate provisional items; notification event logged in audit history.<br>**Fail**: Notifications fail to dispatch, dispatch after the 48-hour cutoff, or display incorrect dates/provisional items. | Timely notifications maximize subscriber engagement, reduce disputes over auto-renewed selections, and ensure transparent billing cutoff awareness. |
| **FR-005** | Functional | The system shall aggregate finalized monthly customizations and allow the Fulfillment Lead to generate, preview, and lock the monthly fulfillment dispatch manifest once the 48-hour customization window closes. | **High** | **Pass**: Manifest consolidates all active (non-paused, non-skipped) boxes with exact item SKUs, destination postal tags, and batch IDs; further subscriber edits are strictly rejected after lock timestamp.<br>**Fail**: Cancelled/paused accounts appear on the manifest, SKU counts do not match subscriber selections, or changes can be submitted post-lockout. | Warehouse operations require locked, deterministic packing manifests to pick, pack, and stage subscription boxes efficiently without inventory discrepancies. |
| **NFR-001** | Performance & Scalability | The fulfillment manifest generator must aggregate finalized orders and export bulk shipping labels and manifests for 10,000 monthly boxes in under 1 minute (< 60 seconds) under simulated peak load. | **High** | **Pass**: Benchmarking tests confirm end-to-end generation and export of 10,000 packing manifests and PDF/ZPL shipping labels completes in <= 58 seconds with average server CPU utilization <= 75% and RAM <= 4 GB.<br>**Fail**: Manifest export exceeds 60 seconds, returns gateway 504 timeouts, or memory leaks cause service termination during bulk processing. | End-of-month fulfillment dispatch windows are tightly bound to courier pickup schedules; processing latency bottlenecks stall warehouse packaging lines. |
| **NFR-002** | Security & Compliance | The system shall ensure subscriber financial details comply with PCI-DSS standards (storing only tokenized payment identifiers via third-party gateways) and personal data is encrypted using AES-256 at rest and TLS 1.3 in transit, with strict Role-Based Access Control (RBAC). | **Critical** | **Pass**: Penetration audits verify zero plaintext card data stored in application databases or logs; RBAC tests confirm Fulfillment Leads access only shipping addresses and item pick lists without seeing payment credentials.<br>**Fail**: Unencrypted sensitive PII/payment data exposed via API endpoints, stored in logs, or accessible to non-privileged roles. | Mandatory regulatory compliance (PCI-DSS, GDPR/DPDP) to protect customer data against identity theft, credential stuffing, and unauthorized internal data exposure. |

---

## 3. Requirement Traceability Matrix (RTM)

| Requirement ID | Stakeholder / Actor | System Subsystem | Verification Method |
| :--- | :--- | :--- | :--- |
| **FR-001** | Subscriber | Box Customization Engine & Inventory Gateway | Automated Functional Test & E2E UI Test |
| **FR-002** | Subscriber | Preference Tagging & Recommendation Service | Unit Test (Tag Filtering) & Integration Test |
| **FR-003** | Subscriber | Subscription Lifecycle & Billing Scheduler | Automated Integration Test with Mock Gateway |
| **FR-004** | Subscriber | Notification & Event Scheduling Service | Scheduled Worker Integration Test & Mock SMTP |
| **FR-005** | Fulfillment Lead | Manifest Generation & Order Lockout Service | Batch Processing Test & Database Lock Test |
| **NFR-001** | Fulfillment Lead | Bulk Label Export & Manifest Pipeline | Load & Stress Testing (10,000 Concurrent Orders) |
| **NFR-002** | All Stakeholders | Identity, RBAC & Data Encryption Subsystems | Security Audit, Static Code Analysis & Penetration Scan |
