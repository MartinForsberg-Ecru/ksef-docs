# **KSeF 2.0 RC4 - Guide for Integrators**

27.08.2025

This document is a knowledge compendium for developers, analysts, and system integrators implementing integration with the National e-Invoicing System (KSeF) version 2.0. The guide focuses on technical and practical aspects related to communication with the KSeF system API.

## Introduction to KSeF 2.0

The National e-Invoicing System (KSeF) is a central IT system used for issuing and retrieving structured invoices in electronic form.

## Table of Contents

The guide is divided into thematic sections corresponding to key functions and integration areas within the KSeF API:

* [Overview of Key Changes in KSeF 2.0](przeglad-kluczowych-zmian-ksef-api-2-0_en.md)
* [Changelog](api-changelog_en.md)
* Authentication

  * [Obtaining Access](uwierzytelnianie_en.md)
  * [Session Management](auth/sesje_en.md)
* [Authorizations](uprawnienia_en.md)
* [KSeF Certificates](certyfikaty-KSeF_en.md)
* Offline Modes

  * [Offline Modes](tryby-offline_en.md)
  * [Technical Correction](offline/korekta-techniczna_en.md)
* [QR Codes](kody-qr_en.md)
* [Interactive Session](sesja-interaktywna_en.md)
* [Batch Session](sesja-wsadowa_en.md)
* [Invoice Retrieval](pobieranie-faktur_en.md)
* [Managing KSeF Tokens](tokeny-ksef_en.md)

For each area, the following are provided:

* a detailed functional description,
* a call example in C# and Java,
* links to the [OpenAPI specification](https://ksef-test.mf.gov.pl/docs/v2) and reference library code.

The code examples in this guide are based on the official open-source libraries:

* [ksef-client-csharp](https://github.com/CIRFMF/ksef-client-csharp) – C# library
* [ksef-client-java](https://github.com/CIRFMF/ksef-client-java) – Java library

Both libraries are maintained and developed by the Ministry of Finance teams and are publicly available under open-source licenses. They provide full support for KSeF 2.0 API functionality, including all endpoints, data models, and sample call scenarios. Using these libraries significantly simplifies the integration process and minimizes the risk of misinterpreting API contracts.

## System Environments

An overview of KSeF API 2.0 environments is described in the document [KSeF API 2.0 Environments](srodowiska_en.md)
