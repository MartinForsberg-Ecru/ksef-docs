## API 2.0 Changes

### Version 2.0.0 RC4

* **KSeF Certificates**

  * Added a new `type` property in KSeF certificates.
  * Available certificate types:

    * `Authentication` – certificate for system authentication in KSeF,
    * `Offline` – certificate restricted to confirming the authenticity of the issuer and invoice integrity in offline mode (KOD II).
  * Updated documentation for processes: `/certificates/enrollments`, `/certificates/query`, `/certificates/retrieve`.

* **QR Codes**

  * Clarified that KOD II can only be generated using a certificate of type `Offline`.
  * Added a security warning that `Authentication` certificates cannot be used to issue offline invoices.

* **Session Status**

  * Authorization update – retrieving session, invoice, and UPO information now requires the permission: `InvoiceWrite`.
  * Status code for *processing* changed: from `300` to `150` for batch sessions.

* **Retrieving Invoice Metadata (`/invoices/query/metadata`)**
  Response model extended with fields:

  * `fileHash` – SHA256 hash of the invoice,
  * `hashOfCorrectedInvoice` – SHA256 hash of the corrected offline invoice,
  * `thirdSubjects` – list of third parties,
  * `authorizedSubject` – authorized entity (new `InvoiceMetadataAuthorizedSubject` object with `identifier`, `name`, `role`),

* Added filtering by document type (`InvoiceQueryFormType`), allowed values: `FA`, `PEF`, `RR`.

* Field `schemaType` marked as deprecated – planned for removal in future API versions.

* **Documentation**

  * Added document describing [KSeF number](faktury/numer-ksef_en.md).
  * Added document describing [technical correction](offline/korekta-techniczna_en.md) for invoices issued in offline mode.
  * Clarified the method for [detecting duplicates](faktury/weryfikacja-faktury_en.md).

* **OpenAPI**

  * Retrieving invoice metadata list:

    * Added property: `hasMore` (boolean) – indicates the availability of the next result page. `totalCount` marked as deprecated (temporarily kept for backward compatibility).
    * In `dateRange` filtering, the `to` (end date) property is no longer mandatory.
  * Permission search response – added `hasMore`, removed `pageSize`, `pageOffset`.
  * Authentication status response – removed redundant `referenceNumber`, `isCurrent`.
  * Unified pagination – endpoint `/sessions/{referenceNumber}/invoices` (retrieve session invoices) now uses `x-continuation-token` request header; `pageOffset` removed, `pageSize` unchanged. First page without header; subsequent pages retrieved using token returned by API. Change aligned with other resources using `x-continuation-token` (e.g., `/auth/sessions`, `/sessions/{referenceNumber}/invoices/failed`).
  * Removed support for `InternalId` identifier in `targetIdentifier` during indirect permission grants (`/permissions/indirect/grants`). Only `Nip` identifier is now allowed.
  * Permission grant operation status – extended response status codes:

    * 410 – Provided identifiers are inconsistent or improperly related.
    * 420 – Used credentials lack permission to perform this operation.
    * 430 – Identifier context does not match the required role or permissions.
    * 440 – Operation not allowed for the specified identifier relationships.
    * 450 – Operation not allowed for the given identifier or its type.
  * Added error **21418** – "Provided continuation token is invalid" for all endpoints using `continuationToken` pagination (`/auth/sessions`, `/sessions`, `/sessions/{referenceNumber}/invoices`, `/sessions/{referenceNumber}/invoices/failed`, `/tokens`).
  * Clarified invoice package retrieval process:

    * `/invoices/exports` – starts invoice package creation,
    * `/invoices/async-query/{operationReferenceNumber}` – checks status and retrieves the ready package.
  * Renamed model `InvoiceMetadataQueryRequest` to `QueryInvoicesMetadataReponse`.
  * Extended `PersonPermissionsAuthorIdentifier` enum with new value `System` (system identifier), used for permissions granted by KSeF based on ZAW-FA application. Affects endpoint: `/permissions/query/persons/grants`.

### Version 2.0.0 RC3

* **Added endpoint for retrieving invoice metadata list**

  * `/invoices/query` (mock) replaced by `/invoices/query/metadata` – production endpoint for retrieving invoice metadata.
  * Updated related data models.

* **Updated mock endpoint `invoices/async-query` for initializing invoice retrieval**

  * Updated related data models.

* **OpenAPI**

  * Added required permissions (`x-required-permissions`) to endpoint specs.
  * Added `403 Forbidden` and `401 Unauthorized` responses to endpoint specs.
  * Added `required` attribute in permission query responses.
  * Updated description of `/tokens` endpoint.
  * Removed duplicate `enum` definitions.
  * Unified `SessionInvoiceStatusResponse` model for `/sessions/{referenceNumber}/invoices` and `/sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}`.
  * Added validation status 400: “Authentication failed | No assigned permissions.”

* **Session Status**

  * Added status `Cancelled` – "Session cancelled. Batch session timeout exceeded or no invoices sent in interactive session."
  * Added new error codes:

    * 415 – "Cannot send invoice with attachment"
    * 440 – "Session cancelled, send timeout exceeded"
    * 445 – "Verification error, no valid invoices"

* **Invoice Sending Status**

  * Added date `AcquisitionDate` – date of KSeF number assignment.
  * Field `ReceiveDate` replaced by `InvoicingDate` – date invoice was accepted by KSeF system.

* **Invoice Sending in Session**

  * Added [validation](faktury/weryfikacja-faktury_en.md#ograniczenia-ilo%C5%9Bciowe) of zip package size (100 MB) and number of packages (50) in batch sessions.
  * Added [validation](faktury/weryfikacja-faktury_en.md#ograniczenia-ilo%C5%9Bciowe) of invoice count in interactive and batch sessions.
  * Status code "Processing" changed from 300 to 150.

* **Authentication using XAdES signature**

  * Fixed `ContextIdentifier` in `AuthTokenRequest` XSD. Correct version of [XSD schema](https://ksef-test.mf.gov.pl/docs/v2/schemas/authv2.xsd) must be used. [Preparing XML document](uwierzytelnianie_en.md#1-przygotowanie-dokumentu-xml-authtokenrequest)
  * Added error code `21117` – “Invalid entity identifier for the specified context type.”

* **Removed anonymous invoice download endpoint `invoices/download`**

  * Anonymous invoice download functionality removed; only available via the web-based KSeF tool for verification and download.

* **Test Data – support for invoices with attachments**

  * Added new endpoints enabling testing of invoice sending with attachments.

* **KSeF Certificates – Validation of key type and length in CSR**

  * Added description to POST endpoint `/certificates/enrollments` on requirements for private key types in CSR (RSA, EC),
  * Added new error code 25010 in response 400: “Invalid key type or length.”

* **Updated format for public certificates**

  * `/security/public-key-certificates` – returns certificates in DER format, Base64 encoded.

### Version 2.0.0 RC2

* **New endpoints for managing authentication sessions**
  Allow viewing and revoking active authentication sessions.
  [Authentication Session Management](auth/sesje_en.md)

* **New endpoint for retrieving invoice sending session list**
  `/sessions` – allows retrieval of metadata for sending sessions (interactive and batch), with filtering options including status, closing date, and session type.
  [Retrieving session list](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo_en.md#1-pobranie-listy-sesji)

* **Change in permission listing**
  `/permissions/query/authorizations/grants` – added `queryType` in filtering [entity-level permissions](uprawnienia_en.md#pobranie-listy-uprawnień-podmiotowych-do-obsługi-faktur).

* **Support for new FA(3) invoice schema version**
  Interactive and batch session opening now supports choosing the FA(3) schema.

* **Added `invoiceFileName` field in batch session response**
  `/sessions/{referenceNumber}/invoices` – added `invoiceFileName` field with the invoice file name. Field only appears for batch sessions.
  [Retrieving information about submitted invoices](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo_en.md#3-pobranie-informacji-na-temat-przesłanych-faktur)
