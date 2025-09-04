## KSeF Certificates

31.08.2025

### Introduction

A KSeF certificate is a digital credential confirming an entity's identity, issued by the KSeF system upon user request.

A certificate request can only be submitted for the identity data found in the certificate used for [authentication](uwierzytelnianie_en.md). Based on this data, the endpoint [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get) returns identification information that must be used in the certificate request.

> The system does not allow requesting a certificate on behalf of another entity.

Two types of certificates are available – each certificate can have **only one type** (`Authentication` or `Offline`). It is not possible to issue a certificate combining both functions.

| Type             | Description                                                                                                                                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Authentication` | Certificate intended for authentication in the KSeF system.<br/>**keyUsage:** Digital Signature (80)                                                                                                                             |
| `Offline`        | Certificate intended only for issuing invoices in offline mode. Used to confirm issuer authenticity and invoice integrity via [QR Code II](kody-qr.md). It does not allow authentication.<br/>**keyUsage:** Non-Repudiation (40) |

#### Certificate Request Process

The certificate request process includes several steps:

1. Checking available limits,
2. Downloading certificate enrollment data,
3. Submitting the request,
4. Retrieving the issued certificate.

---

### 1. Checking Limits

Before an API client submits a new certificate request, it's recommended to verify the certificate limits.

The API provides information on:

* the maximum number of certificates allowed,
* the number of currently active certificates,
* whether another request can be submitted.

GET [/certificates/limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1limits/get)

#### Example in C#:

```csharp
var limits = await ksefClient.GetCertificateLimitsAsync(accessToken, cancellationToken);
```

#### Example in Java:

```java
var limits = ksefClient.getCertificateLimits();
```

---

### 2. Retrieving Certificate Enrollment Data

To begin the certificate request process, you must retrieve the identity data set by calling:
GET [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get)

This data is read from the certificate used for authentication, which can be:

* a qualified certificate of a natural person – containing PESEL or NIP,
* a qualified organization certificate (a.k.a. company seal) – containing NIP,
* Trusted Profile (ePUAP) – used by individuals, contains PESEL,
* an internal KSeF certificate – issued by KSeF, not qualified but accepted for authentication.

The system returns the full X.500 Distinguished Name (DN) attributes, which must be used to build the CSR. Modifying this data will cause the request to be rejected.

**Note**: Certificate data retrieval is only possible when authenticating using a digital signature (XAdES). Authentication using a KSeF system token does not allow certificate requests.

#### Example in C#:

```csharp
var enrollmentData = await ksefClient
    .GetCertificateEnrollmentDataAsync(accessToken, cancellationToken)
    .ConfigureAwait(false);
```

#### Example in Java:

```java
CertificateEnrollmentsInfoResponse enrollmentData = ksefClient.getCertificateEnrollmentInfo();
```

#### Table of possible returned fields (OID-based):

| OID      | Field Name             | Description                         | Natural Person | Company Seal |
| -------- | ---------------------- | ----------------------------------- | -------------- | ------------ |
| 2.5.4.3  | commonName             | Common Name                         | ✔️             | ✔️           |
| 2.5.4.4  | surname                | Surname                             | ✔️             | ❌            |
| 2.5.4.5  | serialNumber           | Serial Number (e.g. PESEL, NIP)     | ✔️             | ❌            |
| 2.5.4.6  | countryName            | Country Code (e.g., PL)             | ✔️             | ✔️           |
| 2.5.4.10 | organizationName       | Organization / Company Name         | ❌              | ✔️           |
| 2.5.4.42 | givenName              | Given Name(s)                       | ✔️             | ❌            |
| 2.5.4.45 | uniqueIdentifier       | Unique Identifier (optional)        | ✔️             | ✔️           |
| 2.5.4.97 | organizationIdentifier | Organization Identifier (e.g., NIP) | ❌              | ✔️           |

`givenName` can appear multiple times and is returned as a list.

---

### 3. Preparing the CSR (Certificate Signing Request)

To request a KSeF certificate, prepare a Certificate Signing Request (CSR) in PKCS#10 standard, DER format, Base64-encoded. The CSR must include:

* identity information (DN – Distinguished Name),
* public key to bind to the certificate.

Private key requirements:

* Allowed types:

  * RSA (OID: 1.2.840.113549.1.1.1), minimum length: 2048 bits
  * EC (OID: 1.2.840.10045.2.1), minimum length: 256 bits
* EC keys are recommended.

All X.509 identity data must exactly match values returned by `/certificates/enrollments/data`. Any changes will result in rejection.

#### Example in C# (with `ICryptographyService`):

```csharp
(var csrBase64Encoded, var privateKeyBase64Encoded) = cryptographyService.GenerateCsr(enrollmentData);
```

#### Example in Java:

```java
CsrResult csr = cryptographyService.generateCsr(enrollmentData);
```

* `csrBase64Encoded` – CSR in Base64 format, ready to send to KSeF
* `privateKeyBase64Encoded` – corresponding private key in Base64, needed for signing

**Note**: The private key should be securely stored according to your organization’s security policy.

---

### 4. Submitting the Certificate Request

Send the Base64-encoded CSR to:
POST [/certificates/enrollments](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post)

Include:

* **certificate name** – appears in metadata for identification,
* **certificate type** – `Authentication` or `Offline`,
* **CSR** – Base64 encoded,
* (optional) **validFrom** – start date of validity. If omitted, validity starts at issuance.

Ensure CSR matches exactly the data from `/certificates/enrollments/data`.

#### Example in C#:

```csharp
var request = SendCertificateEnrollmentRequestBuilder.Create()
    .WithCertificateName("Test Certificate")
    .WithCertificateType(CertificateType.Authentication)
    .WithCsr(csrBase64Encoded)
    .WithValidFrom(DateTimeOffset.UtcNow.AddDays(1))
    .Build();

var certificateEnrollmentResponse = await ksefClient.SendCertificateEnrollmentAsync(request, accessToken, cancellationToken);
```

#### Example in Java:

```java
var request = new SendCertificateEnrollmentRequestBuilder()
    .withCertificateName("certificateName")
    .withCsr(csr.csr())
    .withValidFrom(OffsetDateTime.now())
    .build();

return ksefClient.sendCertificateEnrollment(request);
```

Response contains a `referenceNumber` for tracking and retrieving the certificate.

---

### 5. Checking Certificate Request Status

Certificate issuance is asynchronous. Use the `referenceNumber` to check status:

GET [/certificates/enrollments/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1%7BreferenceNumber%7D/get)

If rejected, the response will include error details.

#### Example in C#:

```csharp
await ksefClient.GetCertificateEnrollmentStatusAsync(
    certificateEnrollmentResponse.ReferenceNumber, 
    accessToken, 
    cancellationToken);
```

#### Example in Java:

```java
ksefClient.getCertificateEnrollmentStatus(referenceNumber);
```

Once you receive the `certificateSerialNumber`, you can retrieve the certificate and metadata.

---

### 6. Retrieving Certificates

Use serial numbers to retrieve issued internal certificates. Returned in DER format, Base64-encoded.

POST [/certificates/retrieve](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1retrieve/post)

#### Example in C#:

```csharp
var certificates = await ksefClient.GetCertificateListAsync(
    new CertificateListRequest { CertificateSerialNumbers = certificateSerialNumbers }, 
    accessToken, 
    cancellationToken);
```

#### Example in Java:

```java
var certificates = ksefClient.getCertificateList(new CertificateListRequest(List.of(certificateSerialNumber)))
    .getCertificates();
```

Each response element includes:

| Field                     | Description                                   |
| ------------------------- | --------------------------------------------- |
| `certificateSerialNumber` | Certificate's serial number                   |
| `certificateName`         | Certificate name assigned during registration |
| `certificate`             | Base64-encoded certificate (DER format)       |
| `certificateType`         | `Authentication` or `Offline`                 |

---

### 7. Retrieving Certificate Metadata

You can retrieve metadata for internal certificates submitted by the entity. Includes both active and historical certificates.

POST [/certificates/query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1query/post)

Optional filters:

* `status` – (`Active`, `Blocked`, `Revoked`, `Expired`)
* `expiresAfter` – certificate expiration date
* `name` – certificate name
* `type` – `Authentication` or `Offline`
* `certificateSerialNumber` – serial number
* `pageSize` – default 10
* `pageOffset` – default 0

#### Example in C#:

```csharp
var request = GetCertificateMetadataListRequestBuilder
    .Create()
    .WithCertificateSerialNumber(serialNumber)
    .WithName(name)
    .Build();

var metadataList = await ksefClient.GetCertificateMetadataListAsync(accessToken, request, 20, 0, cancellationToken);
```

#### Example in Java:

```java
var request = new CertificateMetadataListRequestBuilder()
    .withName("name")
    .withCertificateSerialNumber("certificateSerialNumber")
    .withStatus(CertificateListItemStatus.ACTIVE)
    .build();

ksefClient.getCertificateMetadataList(request, 10, 0)
    .getCertificates();
```

Returns metadata for certificates.

---

### 8. Revoking Certificates

Certificates can only be revoked by the owner, e.g., in case of key compromise, termination of use, or organizational change. Once revoked, they cannot be used for further authentication or operations.

POST [/certificates/{certificateSerialNumber}/revoke](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1%7BcertificateSerialNumber%7D~1revoke/post)

#### Example in C#:

```csharp
var request = RevokeCertificateRequestBuilder.Create()
    .WithRevocationReason(CertificateRevocationReason.KeyCompromise) // optional
    .Build();

await ksefClient.RevokeCertificateAsync(request, certificateSerialNumber, accessToken, cancellationToken)
    .ConfigureAwait(false);
```

#### Example in Java:

```java
var request = new RevokeCertificateRequestBuilder()
    .withRevocationReason(CertificateRevocationReason.KEYCOMPROMISE)
    .build();

ksefClient.revokeCertificate(request, certificateSerialNumber);
```

Revoked certificates cannot be reused. To continue use, a new certificate must be requested.


