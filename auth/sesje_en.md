## Managing Authentication Sessions

10.07.2025

### Retrieving the List of Active Authentication Sessions

Returns a list of active authentication sessions.

GET [/auth/sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions/get)

**Example in C#:**

```csharp
const int pageSize = 20;
string? continuationToken = null;
var activeSessions = new List<Item>();
do
{
    var response = await ksefClient.GetActiveSessions(accessToken, pageSize, continuationToken, cancellationToken);
    continuationToken = response.ContinuationToken;
    activeSessions.AddRange(response.Items);
}
while (!string.IsNullOrWhiteSpace(continuationToken));
```

**Example in Java:**

```java
int pageSize = 10;
String continuationToken = null;
ksefClient.getActiveSessions(pageSize, continuationToken);
```

---

### Revoking the Current Session

DELETE [`/auth/sessions/current`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1current/delete)

Revokes the session associated with the token used in the request. After this operation:

* The associated `refreshToken` is revoked,
* Active `accessTokens` remain valid until their expiry.

**Example in C#:**

```csharp
await ksefClient.RevokeCurrentSessionAsync(token, cancellationToken);
```

**Example in Java:**

```java
ksefClient.revokeCurrentSession();
```

---

### Revoking a Specific Session

DELETE [`/auth/sessions/{referenceNumber}`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Aktywne-sesje/paths/~1api~1v2~1auth~1sessions~1%7BreferenceNumber%7D/delete)

Revokes the session identified by the given reference number. After this operation:

* The associated `refreshToken` is revoked,
* Active `accessTokens` remain valid until their expiry.

**Example in C#:**

```csharp
await ksefClient.RevokeSessionAsync(referenceNumber, accessToken, cancellationToken);
```

**Example in Java:**

```java
ksefClient.revokeSession(referenceNumber);
```
