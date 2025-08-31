## Authentication

**10.07.2025**

## Introduction

Authentication in KSeF API 2.0 is a mandatory step required before accessing protected system resources. The process is based on obtaining an **access token (`accessToken`)** in `JWT` format, which is then used to authorize API operations.

The authentication process relies on two elements:

* **Login context** – defines the entity on whose behalf operations will be performed, such as a company identified by its NIP.
* **Authenticating entity** – indicates who is attempting to authenticate. The way this information is provided depends on the chosen authentication method.

**Available authentication methods:**

* **Using an XAdES signature**
  You submit an XML document (`AuthTokenRequest`) containing a digital signature in XAdES format. The authenticating entity is identified from the certificate used for the signature (e.g., via NIP, PESEL, or certificate fingerprint).
* **Using a KSeF token**
  You submit a JSON document containing a previously obtained system token (the so-called [KSeF token](tokeny-ksef_en.md)). The authenticating entity is determined based on the submitted [KSeF token](tokeny-ksef_en.md).

The authenticating entity is verified – the system checks whether it has at least one active permission in the selected context. Without such permissions, obtaining an access token and using the API is not possible.

The obtained token is valid only for a specified time and can be refreshed without re-authenticating. Tokens are automatically invalidated if permissions are revoked.

---

## Authentication Process

### 1. Obtaining the auth challenge

The process begins by retrieving an **auth challenge**, required to build the authentication request:

**POST** `/auth/challenge`

The challenge remains valid for 10 minutes.

**C# example:**

```csharp
var challengeResponse = await ksefClient.GetAuthChallengeAsync();
```

**Java example:**

```java
var challenge = ksefClient.getAuthChallenge();
```

The response includes the challenge and a timestamp.

---

## 2. Selecting Authentication Method

### 2.1. Authentication **using a qualified electronic signature (XAdES)**

#### a) Preparing the XML document (`AuthTokenRequest`)

After receiving the challenge, prepare an XML document compliant with the `AuthTokenRequest` schema. It includes:

| Field                      | Value                                                                   |
| -------------------------- | ----------------------------------------------------------------------- |
| Challenge                  | Value obtained from `POST /auth/challenge`                              |
| ContextIdentifier          | Identifier for authentication context (NIP, internal ID, or VAT UE)     |
| SubjectIdentifierType      | Identification method: `certificateSubject` or `certificateFingerprint` |
| (optional) IpAddressPolicy | IP validation rules for the issued access token                         |

Example XML references:

* auth/subject-identifier-type-certificate-subject\_en.md
* auth/subject-identifier-type-certificate-fingerprint\_en.md
* auth/context-identifier-nip\_en.md
* auth/context-identifier-internal-id\_en.md
* auth/context-identifier-nip-vat-ue\_en.md

Once prepared, the XML is signed using the entity’s certificate.

**Sample implementation table:**

| ContextIdentifier | SubjectIdentifierType           | Meaning                                                                          |
| ----------------- | ------------------------------- | -------------------------------------------------------------------------------- |
| `nip=1234567890`  | `certificateSubject` with NIP   | Auth on behalf of company with NIP 1234567890                                    |
| `nip=1234567890`  | `certificateSubject` with PESEL | Person with PESEL 88102341294 acting for that company                            |
| `nip=1234567890`  | `certificateFingerprint`        | Signature with a certificate for which permissions were granted for that company |

**C# sample:**

```csharp
var authTokenRequest = AuthTokenRequestBuilder
    .Create()
    .WithChallenge(challengeResponse.Challenge)
    .WithContext(ContextIdentifierType.Nip, contextIdentifier)
    .WithIdentifierType(SubjectIdentifierTypeEnum.CertificateSubject)
    .WithIpAddressPolicy(ipAddressPolicy)
    .Build();
```

**Java sample:**

```java
var authTokenRequest = new AuthTokenRequestBuilder()
    .withChallenge(challenge.getChallenge())
    .withContext(ContextIdentifierTypeEnum.NIP, context)
    .withSubjectType(SubjectIdentifierTypeEnum.CERTIFICATE_SUBJECT)
    .withIpAddressPolicy(IpChangePolicyEnum.IGNORE.value(), List.of(), List.of(), List.of())
    .build();
```

#### b) Signing the document (XAdES)

The `AuthTokenRequest` must be signed in XAdES format. This requires using:

* A qualified certificate with PESEL or NIP, or
* A qualified organizational seal (with NIP), or
* A Trusted Profile (ePUAP) via [gov.pl](https://www.gov.pl/web/gov/podpisz-dokument-elektronicznie-wykorzystaj-podpis-zaufany), or
* A **KSeF certificate** issued by KSeF (not a qualified certificate but accepted for authentication).

In test environments, you may use a self‑signed certificate as a stand-in.

Ksef.Client supports XAdES digital signature.

After signing, submit the XML to retrieve a temporary authentication token (`authenticationToken`).

See signature formats in detail [here](auth/podpis-xades_en.md).

**C# sample for signature generation:**

```csharp
var unsignedXml = AuthTokenRequestSerializer.SerializeToXmlString(authTokenRequest);
var signedXml = signatureService.SignAsync(unsignedXml, certificate);
```

**Java sample:**

```java
var unsignedXml = AuthTokenRequestSerializer.authTokenRequestSerializer(authTokenRequest);
var signedXml = new DefaultSignatureService().sign(xml.getBytes(), cert.certificate(), cert.getPrivateKey());
```

#### c) Submitting the signed XML

Submit the signed document via:

**POST** `/auth/xades-signature`

The response returns:

* `authenticationToken` (temporary JWT), and
* `referenceNumber`

These are used to check the authentication status and retrieve the access token (`accessToken`).

**C# example:**

```csharp
var authOperationInfo = await ksefClient.SubmitXadesAuthRequestAsync(signedXml);
```

**Java example:**

```java
var submitAuthTokenResponse = ksefClient.submitAuthTokenRequest(signedXml, false);
```

---

### 2.2. Authentication **using a KSeF token**

This variant involves sending an **encrypted string** combining the KSeF token and the challenge timestamp. The timestamp acts as a nonce to prevent replay attacks.

#### a) Preparing and encrypting the token

Create a string: `{tokenKSeF}|{challengeTimestamp}`
Encrypt it using KSeF public key (`RSA-OAEP` with `SHA-256 MGF1`) and encode with Base64.

**C# example:**

```csharp
var tokenWithTimestamp = $"{tokenKsef}|{timestampMs}";
var tokenBytes = Encoding.UTF8.GetBytes(tokenWithTimestamp);
var encryptedBytes = cryptographyService.EncryptKsefTokenWithRSAUsingPublicKey(tokenBytes);
var encryptedToken = Convert.ToBase64String(encryptedBytes);
```

**Java example:**

```java
var tokenWithTimestamp = ksefToken.getToken() + "|" + challengeTimeMs;
var encryptedToken = new DefaultCryptographyService(defaultKsefClient)
    .encryptKsefTokenWithRSAUsingPublicKey(tokenWithTimestamp.getBytes(StandardCharsets.UTF_8));
```

#### b) Submitting authentication request with KSeF token

Send the encrypted data along with the challenge and context (and optionally, IP policy) to:

**POST** `/auth/ksef-token`

**C# example:**

```csharp
var builder = AuthKsefTokenRequestBuilder
    .Create()
    .WithChallenge(challenge)
    .WithContext(contextIdentifierType, contextIdentifierValue)
    .WithEncryptedToken(encryptedToken);
...
var submissionRef = await ksefClient.SubmitKsefTokenAuthRequestAsync(authKsefTokenRequest, cancellationToken);
```

**Java example:**

```java
var authTokenRequest = new AuthKsefTokenRequestBuilder()
    .withChallenge(challenge.getChallenge())
    .withContextIdentifier(new ContextIdentifier(ContextIdentifierType.NIP, CONTEXT_NIP))
    .withEncryptedToken(Base64.getEncoder().encodeToString(encryptedToken))
    .build();
var response = defaultKsefClient.authorizeByKSeFToken(authTokenRequest);
```

The response returns `authenticationToken` and `referenceNumber`, used to check status and retrieve `accessToken`.

---

### 3. Checking Authentication Status

After sending `AuthTokenRequest` and receiving `authenticationToken` and `referenceNumber`, check the authentication status with:

**GET** `/auth/{referenceNumber}`
Use `Authorization: Bearer <authenticationToken>`

The response indicates the current state (e.g., *in progress*, *successful*).

**Caution:**
In production, authenticating with a qualified certificate may take longer due to external validation (OCSP/CRL).

**Error handling:**
Errors may indicate invalid signatures, missing permissions, or technical issues. Refer to endpoint documentation for codes.

**C# example:**

```csharp
var authorizationStatus = await ksefClient.GetAuthStatusAsync(referenceNumber, authenticationToken);
```

**Java example:**

```java
var authStatus = defaultKsefClient.getAuthStatus(response.getReferenceNumber());
```

---

### 4. Obtaining the access token (`accessToken`)

Upon successful authentication, retrieve your access and refresh tokens:

**POST** `/auth/token/redeem`

**C# example:**

```csharp
var accessTokenResponse = await ksefClient.GetAccessTokenAsync(authOperationInfo);
```

**Java example:**

```java
var accessTokenResponse = ksefClient.getAccessToken(submitAuthTokenResponse.getReferenceNumber());
```

Response includes:

* `accessToken` — JWT for API authorization (valid for a short duration, defined by `exp`)
* `refreshToken` — for renewing `accessToken` without re-authentication (valid up to 7 days)

**Note:**

1. Treat both tokens as confidential.
2. `accessToken` remains valid until `exp`, even if permissions are later revoked.

---

### 5. Refreshing the access token

To maintain uninterrupted API access, use the `refreshToken`:

**POST** `/auth/token/refresh`
Include in header:

```
Authorization: Bearer {refreshToken}
```

Response provides a new `accessToken` with updated permissions.

**C# example:**

```csharp
var refreshedAccessTokenResponse = await ksefClient.RefreshAccessTokenAsync(accessTokenResult.RefreshToken.Token);
```

**Java example:**

```java
var accessToken = ksefClient.refreshAccessToken(refreshToken);
```

---

### 6. Managing Authentication Sessions

For details on managing active authentication sessions, see the document [Authentication Sessions Management](auth/sesje_en.md).
