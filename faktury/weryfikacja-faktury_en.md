# Invoice Validation

29.07.2025

Invoices submitted to the KSeF system undergo a series of technical and semantic checks. The verification includes the following criteria:

## XSD Schema Compliance

The invoice must be prepared in XML format, encoded in UTF-8, and compliant with the current schema published by the Ministry of Finance.

## Invoice Uniqueness

* KSeF detects invoice duplicates globally based on data stored in the system. The criteria for identifying a duplicate include the combination of:

  1. Seller’s NIP (`Podmiot1:NIP`)
  2. Invoice type (`RodzajFaktury`)
  3. Invoice number (`P_2`)
* In case of a duplicate, error code 440 (“Duplicate invoice”) is returned.
* Invoice uniqueness is maintained in KSeF for 10 full years, counted from the end of the calendar year in which the invoice was issued.
* The uniqueness criterion always refers to the seller (`Podmiot1:NIP`). If different units (e.g., branches, organizational units of local governments, other authorized entities) issue invoices on behalf of the same entity, they must coordinate numbering rules to avoid duplicates.

## Date Validation

The invoice issue date (`P_1`) cannot be later than the date the document is received by the KSeF system.

## File Size

* Maximum invoice size without attachments: **1 MB**
* Maximum invoice size with attachments: **3 MB**

## Quantity Limits

* Maximum number of invoices in a single session (both interactive and batch): 10,000,000
* In batch submission, a maximum of 50 ZIP packages can be sent, each up to 100 MB in size.

## Correct Encryption

* The invoice must be encrypted using AES-256-CBC (256-bit symmetric key, 128-bit IV, PKCS#7 padding).
* The symmetric key must be encrypted using RSAES-OAEP (SHA-256/MGF1).

## Metadata Consistency in Interactive Sessions

* Calculation and verification of the invoice hash along with file size.
* Calculation and verification of the encrypted invoice hash along with file size.

## Attachment Restrictions

* Sending invoices with attachments is only allowed in batch mode.
  **Exception:** In the case of sending an [offline invoice technical correction](../offline/korekta-techniczna_en.md), the use of an interactive session is permitted.
* Sending invoices with attachments requires prior declaration of this option in the `e-Urząd Skarbowy` service.

## Permission Requirements

Sending an invoice to KSeF requires appropriate authorization to issue it on behalf of the relevant entity.
