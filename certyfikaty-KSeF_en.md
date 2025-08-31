## KSeF Certificates

21.08.2025

### Introduction

A KSeF certificate is a digital proof of entity identity, issued by the KSeF system upon user request.

There are two types of certificates available:

| Type               | Description                                                                                                                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Authentication** | Certificate intended for authentication in the KSeF system.                                                                                                                                  |
| **Offline**        | Certificate intended exclusively for issuing offline invoices. Used to confirm issuer authenticity and invoice integrity via [QR Code II](kody-qr_en.md). Cannot be used for authentication. |

#### Certificate Issuance Process

The application process for a certificate consists of several steps:

1. Checking available limits,
2. Retrieving certificate request data,
3. Submitting the application,
4. Retrieving the issued certificate.

### 1. Checking Limits

Before submitting a new certificate request, it's recommended to verify the certificate limits.

The API provides information about:

* the maximum number of allowed certificates,
* the number of currently active certificates,
* whether a new request can be submitted.

GET [/certificates/limits](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1limits/get)

C# Example:

```csharp
var limits = await ksefClient.GetCertificateLimitsAsync(accessToken, cancellationToken);
```

Java Example:

```java
var limits = ksefClient.getCertificateLimits();
```

### 2. Retrieving Certificate Enrollment Data

To begin the certificate application process, retrieve the identification data used to generate the Certificate Signing Request (CSR).

GET [/certificates/enrollments/data](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1data/get)

This data is returned based on the certificate used for authentication and reflects the identity of the applicant entity.

**Note**: Retrieving certificate data is only possible using signature-based authentication (XAdES). Authentication with a KSeF system token does not allow certificate application.

C# Example:

```csharp
var enrollmentData = await ksefClient
    .GetCertificateEnrollmentDataAsync(accessToken, cancellationToken)
    .ConfigureAwait(false);
```

Java Example:

```java
CertificateEnrollmentsInfoResponse enrollmentData = ksefClient.getCertificateEnrollmentInfo();
```

The following table shows all returned fields with OIDs:

| OID      | Name (EN)              | Description                         |
| -------- | ---------------------- | ----------------------------------- |
| 2.5.4.3  | commonName             | Common name                         |
| 2.5.4.4  | surname                | Surname                             |
| 2.5.4.5  | serialNumber           | Serial number (e.g., PESEL, NIP)    |
| 2.5.4.6  | countryName            | Country code (e.g., PL)             |
| 2.5.4.10 | organizationName       | Organization name                   |
| 2.5.4.42 | givenName              | First name(s)                       |
| 2.5.4.45 | uniqueIdentifier       | Unique identifier (optional)        |
| 2.5.4.97 | organizationIdentifier | Organization identifier (e.g., NIP) |

The `givenName` attribute may appear multiple times and is returned as a list of values.

### 3. Preparing CSR (Certificate Signing Request)

To apply for a KSeF certificate, you must prepare a Certificate Signing Request (CSR) in PKCS#10 format, containing:

* subject identification data (DN – Distinguished Name),
* the public key to be bound to the certificate.

Private key requirements:

* Allowed types:

  * RSA (OID: 1.2.840.113549.1.1.1), minimum length: 2048 bits,
  * EC (Elliptic Curve, OID: 1.2.840.10045.2.1), minimum length: 256 bits.
* EC keys are recommended.

All X.509 attributes must exactly match the values returned by the system in the previous step (/certificates/enrollment/data). Any modification will cause the request to be rejected.

C# Example using `ICryptographyService`:

```csharp
(var csrBase64encoded, var privateKeyBase64Encoded) = cryptographyService.GenerateCsr(enrollmentData);
```

Java Example:

```java
CsrResult csr = cryptographyService.generateCsr(enrollmentData);
```

* `csrBase64Encoded` – Base64-encoded CSR, ready to send to KSeF.
* `privateKeyBase64Encoded` – Base64-encoded private key linked to the generated CSR. Needed for signing and authentication.

**Note**: The private key must be stored securely according to the organization's security policy.

### 4. Submitting the Certificate Request

After preparing the CSR, submit it to KSeF using:

POST [/certificates/enrollments](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post)

Request must include:

* **certificate name** – visible in metadata, helps identify the certificate,
* **certificate type** – `Authentication` or `Offline`,
* **CSR** in PKCS#10 (DER), Base64-encoded,
* (optional) **validFrom** – certificate validity start date. If omitted, the certificate is valid from the issuance moment.

Ensure the CSR contains exactly the data returned from /certificates/enrollments/data.

C# Example:

```csharp
var request = SendCertificateEnrollmentRequestBuilder.Create()
    .WithCertificateName("Testowy certyfikat")
    .WithCertificateType(CertificateType.Authentication)
    .WithCsr(csrBase64encoded)
    .WithValidFrom(DateTimeOffset.UtcNow.AddDays(1))
    .Build();

var certificateEnrollmentResponse = await ksefClient.SendCertificateEnrollmentAsync(request, accessToken, cancellationToken);
```

Java Example:

```java
var request = new SendCertificateEnrollmentRequestBuilder()
    .withCertificateName("certificateName")
    .withCsr(csr.csr())
    .withValidFrom(OffsetDateTime.now())
    .build();

return ksefClient.sendCertificateEnrollment(request);
```

The response returns a `referenceNumber` used to check request status and later retrieve the certificate.

### 5. Checking the Request Status

Certificate issuance is asynchronous. The system does not return the certificate immediately but allows for later retrieval after processing.

Status should be checked periodically using the `referenceNumber` returned from the submission.

GET [/certificates/enrollments/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments~1%7BreferenceNumber%7D/get)

If the request is rejected, the response will contain an error message.

C# Example:

```csharp
await ksefClient.GetCertificateEnrollmentStatusAsync(certificateEnrollmentResponse.ReferenceNumber, accessToken, cancellationToken);
```

Java Example:

```java
ksefClient.getCertificateEnrollmentStatus(referenceNumber);
```

After obtaining the `certificateSerialNumber`, you can retrieve its content and metadata in the next steps.

## 6. Retrieving Certificates List

KSeF allows retrieving the content of previously issued internal certificates using their serial numbers. Each certificate is returned in DER format, Base64-encoded.

POST [/certificates/retrieve](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1retrieve/post)

C# Example:

```csharp
var certificates = await ksefClient.GetCertificateListAsync(new CertificateListRequest { CertificateSerialNumbers = certificateSerialNumbers }, accessToken, cancellationToken);
```

Java Example:

```java
var certificates = ksefClient.getCertificateList(new CertificateListRequest(List.of(certificateSerialNumber)))
        .getCertificates();
```

Response structure:
Each item contains:

| Field                     | Description                                    |
| ------------------------- | ---------------------------------------------- |
| `certificateSerialNumber` | Serial number of the certificate               |
| `certificateName`         | Name assigned during registration              |
| `certificate`             | Base64-encoded certificate (DER)               |
| `certificateType`         | Certificate type (`Authentication`, `Offline`) |

These certificates can be used for [authentication](uwierzytelnianie_en.md) in KSeF.

## 7. Retrieving Certificate Metadata

You can retrieve metadata of internal certificates submitted by the entity, including both active and historical records with status, validity, and identifiers.

GET [/certificates/query](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1query/post)

#### Optional filtering parameters:

* `status` – certificate status (`Active`, `Blocked`, `Revoked`, `Expired`)
* `expiresAfter` – expiry date (optional)
* `name` – certificate name (optional)
* `type` – certificate type (`Authentication`, `Offline`) (optional)
* `certificateSerialNumber` – serial number (optional)
* `pageSize` – page size (default 10)
* `pageOffset` – page offset (default 0)

C# Example:

```csharp
var request = GetCertificateMetadataListRequestBuilder
    .Create()
    .WithCertificateSerialNumber(serialNumber)
    .WithName(name)
    .Build();

var metadataList = await kSeFClient.GetCertificateMetadataListAsync(accessToken, request, 20, 0, cancellationToken);
```

Java Example:

```java
var request = new CertificateMetadataListRequestBuilder()
        .withName("name")
        .withCertificateSerialNumber("certificateSerialNumber")
        .withStatus(CertificateListItemStatus.ACTIVE)
        .build();

ksefClient.getCertificateMetadataList(request, 10, 0)
        .getCertificates();
```

Sample response metadata:

| Field                     | Example Value          |
| ------------------------- | ---------------------- |
| `certificateSerialNumber` | `00035434535`          |
| `name`                    | `Certyfikat 1`         |
| `type`                    | `Authentication`       |
| `commonName`              | `Jan Kowalski`         |
| `status`                  | `Active`               |
| `subjectIdentifier`       | `1234445678`           |
| `subjectIdentifierType`   | `Nip`                  |
| `validFrom`               | `2019-08-24T14:15:22Z` |
| `validTo`                 | `2019-08-24T14:15:22Z` |
| `lastUseDate`             | `2019-08-24T14:15:22Z` |

## 8. Revoking a Certificate

A KSeF certificate can only be revoked by its owner in case of private key compromise, end of use, or organizational change. Once revoked, it cannot be used for authentication or operations in KSeF.

Revocation is based on the certificate's `certificateSerialNumber` and an optional revocation reason.

POST [/certificates/{certificateSerialNumber}/revoke](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1%7BcertificateSerialNumber%7D~1revoke/post)

C# Example:

```csharp
var request = RevokeCertificateRequestBuilder.Create()
    .WithRevocationReason(CertificateRevocationReason.KeyCompromise) // optional
    .Build();

await ksefClient.RevokeCertificateAsync(request, certificateSerialNumber, accessToken, cancellationToken)
     .ConfigureAwait(false);
```

Java Example:

```java
var request = new RevokeCertificateRequestBuilder()
        .withRevocationReason(CertificateRevocationReason.KEYCOMPROMISE)
        .build();

ksefClient.revokeCertificate(request, certificateSerialNumber);
```

Once revoked, the certificate cannot be reused. To continue using KSeF, a new certificate must be obtained.
