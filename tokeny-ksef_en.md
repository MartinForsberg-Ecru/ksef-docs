## Managing KSeF Tokens

**June 29, 2025**

A KSeF token is a unique authentication identifier that — alongside a [qualified electronic signature](uwierzytelnianie_en.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym) — allows you to [authenticate](uwierzytelnianie_en.md#22-uwierzytelnianie-tokenem-ksef) with the KSeF API.

A `KSeF token` is issued with a fixed set of permissions specified at creation. Any changes to permissions require generating a new token.

> **Important:** <br>
> `KSeF tokens` act as **confidential secrets** and must be stored securely in a trusted vault.

---

### Prerequisites

To generate a `KSeF token`, a one-time authentication using a [qualified electronic signature (XAdES)](uwierzytelnianie_en.md#21-uwierzytelnianie-kwalifikowanym-podpisem-elektronicznym) is required.

---

### 1. Generating a Token

Create a token using the endpoint:
POST [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/post)

Request body includes a list of permissions and a description.

| Field       | Sample Value                                                              | Description                   |
| ----------- | ------------------------------------------------------------------------- | ----------------------------- |
| Permissions | `["InvoiceRead", "InvoiceWrite", "CredentialsRead", "CredentialsManage"]` | Permissions assigned to token |
| Description | `"Token for invoice read/write and account data"`                         | Token description             |

**C# Example:**

```csharp
var tokenRequest = new KsefTokenRequest
{
    Permissions = [
        KsefTokenPermissionType.InvoiceRead,
        KsefTokenPermissionType.InvoiceWrite
    ],
    Description = "Demo token",
};
var token = await ksefClient.GenerateKsefTokenAsync(tokenRequest, accessToken, cancellationToken);
```

**Java Example:**

```java
GenerateTokenRequest request = new GenerateTokenRequestBuilder()
    .withDescription("test description")
    .withPermissions(List.of(
        TokenPermissionType.INVOICEREAD,
        TokenPermissionType.INVOICEWRITE,
        TokenPermissionType.CREDENTIALSREAD))
    .build();
var token = ksefClient.generateKsefToken(request);
```

---

### 2. Filtering Tokens

Retrieve and filter token metadata via:
GET [/tokens](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens/get)

| Parameter | Sample Value        | Description                                   |
| --------- | ------------------- | --------------------------------------------- |
| status    | `pending`, `active` | Filter by status (can be used multiple times) |
| pageSize  | `20`                | Number of results per page (pagination)       |

**C# Example:**

```csharp
var result = new List<AuthenticationKsefToken>();
const int pageSize = 20;
var status = AuthenticationKsefTokenStatus.Active;
var continuationToken = string.Empty;

do
{
    var tokens = await ksefClient.QueryKsefTokensAsync(accessToken, [status], continuationToken, pageSize, cancellationToken);
    result.AddRange(tokens.Tokens);
    continuationToken = tokens.ContinuationToken;
} while (!string.IsNullOrEmpty(continuationToken));
```

**Java Example:**

```java
List<AuthenticationTokenStatus> statuses = List.of(AuthenticationTokenStatus.ACTIVE);
Integer pageSize = 10;
var tokens = ksefClient.queryKsefTokens(statuses, StringUtils.EMPTY, pageSize);
```

Returned metadata includes the issuer, context, and assigned permissions.

---

### 3. Retrieving a Specific Token

Get details of a specific token via:
GET [/tokens/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/get)

`referenceNumber` is the unique ID obtained during token creation or from token listing.

**C# Example:**

```csharp
var token = await ksefClient.GetKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```

**Java Example:**

```java
var token = ksefClient.getKsefToken(referenceNumber);
```

---

### 4. Revoking a Token

Revoke a token via:
DELETE [/tokens/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Tokeny/paths/~1api~1v2~1tokens~1%7BreferenceNumber%7D/delete)

**C# Example:**

```csharp
await ksefClient.RevokeKsefTokenAsync(referenceNumber, accessToken, cancellationToken);
```

**Java Example:**

```java
ksefClient.revokeKsefToken(referenceNumber);
```
