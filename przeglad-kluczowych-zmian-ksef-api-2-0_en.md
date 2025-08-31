# KSeF API 2.0 – Overview of Key Changes

09.06.2025

# Introduction

This document is intended for technical teams and developers familiar with KSeF API version 1.0. It provides an overview of the most significant changes introduced in version 2.0, along with new capabilities and practical integration improvements.

The goals of this document are:

* To highlight key differences compared to version 1.0
* To present the benefits of migrating to version 2.0
* To facilitate integration preparation for the system requirements effective from February 1, 2026

---

## Documentation and Tools Supporting KSeF API 2.0 Integration

To ease the transition to the new API version and ensure correct implementation, the Ministry provides a set of official resources and tools for integrators:

**Technical Documentation (OpenAPI)**

KSeF API version 2.0 is described using the OpenAPI standard, enabling both convenient browsing by developers and automated generation of integration code.

* **Documentation** (interactive online version):
  A portal interface describing methods, data structures, parameters, and examples — designed for developers and integration analysts.
  Test Environment (TE): $[link](https://ksef-test.mf.gov.pl/docs/v2/index.html)$
  Production Environment (PRD)

* **Specification** (OpenAPI JSON file):
  Raw OpenAPI specification in JSON format, usable in automation tools (e.g., code generators, contract validators).
  Test Environment (TE): $[link](https://ksef-test.mf.gov.pl/docs/v2/openapi.json)$
  Production Environment (PRD)

**Official KSeF 2.0 Client Integration Library (Open Source)**

A public, open-source library maintained in parallel with API releases and fully aligned with the specification. It is the recommended integration tool for tracking changes and reducing risk of incompatibility.

* **C#:** $[link](https://github.com/CIRFMF/ksef-client-csharp)$
* **Java:** $[link](https://github.com/CIRFMF/ksef-client-java)$

**Published Packages**

The KSeF 2.0 Client library will be available in official package repositories for popular programming languages — as a NuGet package for .NET, and a Maven Central artifact for Java. This allows easy integration and automatic version tracking.

**Step-by-Step Guide**

* **Integration Guide / Tutorial:**
  Practical step-by-step instructions with code samples showing how to use key system endpoints.
  $[link](https://github.com/CIRFMF/ksef-docs)$

# Key Changes in API 2.0

## New JWT-Based Authentication Model

In version 1.0, authentication was tightly coupled with interactive session initiation, creating integration complexity.

In version 2.0:

* Authentication is now **a separate process**, decoupled from session creation.
* **Standard JWT tokens** are introduced for authorization of all protected operations.

Benefits:

* alignment with industry standards
* reuse of the token across multiple sessions
* **support for token refreshing and revocation**

Authentication process details: $[link](uwierzytelnianie_en.md)$

## Unified Initialization Process for Batch and Interactive Sessions

In API 2.0, session initiation is unified and independent of the session type. After obtaining an authentication token, you can open either:

* interactive session: `POST /sessions/online`
* batch session: `POST /sessions/batch`

Both use a simple JSON structure containing:

* form code (`formCode`)
* AES encryption key for invoice data (`encryptionKey`)

For batch sessions, a list of partial files and metadata is also included.

Usage examples:

* [Interactive session](sesja-interaktywna_en.md)
* [Batch session](sesja-wsadowa_en.md)

## Mandatory Encryption for All Invoices

In version 1.0, encryption was mandatory only in batch mode and optional in interactive mode.

In version 2.0, encryption is **required** for all invoices in both modes.

Each invoice or invoice package must be encrypted locally using an **AES key**, which:

* is generated per session
* is sent to the system during session initialization (`encryptionKey`)

## Asymmetric Encryption

In version 1.0, RSA key pairs and the RSA PKCS#1 v1.5 algorithm remained unchanged.

Version 2.0 introduces a new 4096-bit RSA key pair with OAEP padding using SHA-256 and MGF1-SHA256.

## Consistent and Simplified Naming Convention in API 2.0

One of the major improvements in API 2.0 is consistent and simplified naming of resources, parameters, and JSON models.

Version 2.0 changes include:

* **REST-style endpoint names** (e.g., `sessions/online`, `auth/token`, `permissions/entities/grants`)
* **Simplified operation names** (e.g., `grant`, `revoke`, `refresh`)
* Consistent structure of **headers, parameters, and data formats**
* **Flat, clear data structures** — identifier and permission types use explicit enums (`Nip`, `Pesel`, `Fingerprint`)

While this document does not cover the full change map, it is available in OpenAPI v2 docs and GitHub code examples.

These changes do **not affect core business logic** but make the API clearer and easier to use.

Migration to version 2.0 should be treated as a contract change and requires implementation updates. Use of the official **KSeF 2.0 Client** library is recommended for easing migration and ensuring compatibility with future versions.

## New Module for Managing Internal Certificates

API 2.0 introduces mechanisms for issuing and managing internal **KSeF certificates**. These certificates are required for:

* [authentication](certyfikaty-KSeF_en.md)
* issuing invoices in offline mode

After successful authentication, entities can:

* apply for an internal KSeF certificate
* download the issued certificate
* check the request status
* retrieve metadata of issued certificates
* check their certificate issuance limits

## Improved Batch Submission Process

API 1.0 rejected the entire batch if any invoice had an error.

API 2.0 processes **each invoice independently** within a batch:

* errors affect only individual invoices
* error count is returned in session status
* a dedicated endpoint provides details on failed invoices

This greatly improves batch reliability and usability.

## Invoice Duplicate Verification

Duplicate detection now checks invoice business data (`Subject1:NIP`, `InvoiceType`, `P_2`) instead of the file hash.
Details – [Duplicate Verification](faktury/weryfikacja-faktury_en.md)

## Changes in the Permissions Module

In response to feedback, version 2.0 introduces **indirect permission delegation**, replacing the old inheritance model.

The new system allows granting view/issue permissions to an entity (e.g., accounting office), with an option to further delegate to employees.

Now:

* permissions can be split between viewing/issuing invoices on behalf of a client and self-entity access
* entities can grant **general permissions** to employees for managing all clients they are authorized for
* permission logic is decoupled — employees no longer inherit all of the entity’s client permissions

This model improves internal role management and reduces excessive permissions.

Additionally:

* A new permission type supports **self-billing by EU entities**,
* Login can be done in the context of a Polish NIP and an EU VAT number.
* This enables authorized EU representatives to issue invoices in the name of Polish entities.

All permissions are grouped logically in the API definition by functional area.

## API Rate Limiting

Version 2.0 introduces a **predictable rate limiting mechanism**, replacing the rigid model of version 1.0.

Each endpoint has call limits per:

* second,
* minute,
* hour,
* day.

Limits are:

* publicly documented,
* less restrictive in test vs production environments,
* differentiated per operation:

  * for protected endpoints – limits apply per JWT token
  * for open endpoints (e.g., auth, invoice download) – limits apply per client IP

This allows better scalability, transparency, and test reliability.

## Test Data Generation API (Test Environment)

In the test environment, KSeF 2.0 includes a **supporting API for generating test data**, enabling fast creation of test entities and structures.

This allows simulation of:

* **new business entity creation**,
* **permission granting via ZAW-FA**,
* **JST unit structure creation**,
* **VAT group creation with members**,
* **enforcement bodies and bailiffs**

Since real entities don’t exist in the test environment, **this API is essential** for building complete test scenarios.
