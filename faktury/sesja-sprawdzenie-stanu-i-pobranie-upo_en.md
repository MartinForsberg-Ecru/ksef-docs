## Session – Status Check and UPO Retrieval
10.07.2025

This document describes operations for monitoring the status of a session (interactive or batch) and retrieving UPOs for invoices and the entire session.

### 1. Retrieving the List of Sessions
Returns a list of sessions matching the provided search criteria.

GET [sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions/get)

Returns the current session status along with aggregated data on the number of submitted, correctly and incorrectly processed invoices; after the session is closed, it additionally provides a list of references to the aggregate UPO.

Example in C#:
```csharp
// Retrieving batch sessions
var sessions = new List<Session>();
const int pageSize = 20;
string? continuationToken = null;
do
{
    var response = await ksefClient.GetSessionsAsync(SessionType.Batch, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
    continuationToken = response.ContinuationToken;
    sessions.AddRange(response.Sessions);
} while (!string.IsNullOrEmpty(continuationToken));

// Retrieving interactive sessions
var sessions = new List<Session>();
const int pageSize = 20;
string? continuationToken = null;
do
{
    var response = await ksefClient.GetSessionsAsync(SessionType.Online, accessToken, pageSize, continuationToken, sessionsFilter, cancellationToken);
    continuationToken = response.ContinuationToken;
    sessions.AddRange(response.Sessions);
} while (!string.IsNullOrEmpty(continuationToken));
````

Example in Java:

```java
int pageSize = 10;
String continuationToken = null;
SessionsQueryRequest request = new SessionsQueryRequest();
request.setSessionType(SessionType.ONLINE); // interactive session
//request.setSessionType(SessionType.BATCH); // batch session
SessionsQueryResponse sessionsQueryResponse = defaultKsefClient.getSessions(request, pageSize, continuationToken);
```

### 2. Checking Session Status

Checks the current status of a session.

GET [sessions/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get)

Returns the current status of the session along with aggregated data on the number of submitted, successfully and unsuccessfully processed invoices; after the session is closed, a list of references to the aggregate UPO is also provided.

Example in C#:

```csharp
var openSessionResult = await kSeFClient.GetSessionStatusAsync(referenceNumber, accessToken, cancellationToken);
var documentCount = openSessionResult.InvoiceCount;
var successfulInvoiceCount = openSessionResult.SuccessfulInvoiceCount;
var failedInvoiceCount = openSessionResult.FailedInvoiceCount;
```

Example in Java:

```java
var openSessionResult = ksefClient.getSessionStatus(referenceNumber);
var invoiceCount = openSessionResult.getInvoiceCount();
var successfulInvoiceCount = openSessionResult.getSuccessfulInvoiceCount();
var failedInvoiceCount = openSessionResult.getFailedInvoiceCount();
```

### 3. Retrieving Information About Submitted Invoices

GET [sessions/{referenceNumber}/invoices](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices/get)

Returns metadata for all submitted invoices in the session along with their statuses and the total number of invoices in the session.

Example in C#:

```csharp
const int pageSize = 50;
string continuationtoken = null;

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageOffset,
                                pageSize,
                                cancellationToken);

    foreach (var doc in getInvoicesResult.Invoices)
    {
        Console.WriteLine($"#{doc.InvoiceNumber}. Status: {doc.Status.Code}");
    }

    continuationtoken = sessionInvoices.ContinuationToken;
}
while (continuationtoken != null);
```

Example in Java:

```java
var sessionInvoices = ksefClient.getSessionInvoices(referenceNumber, 10, 0);
sessionInvoices.getInvoices().forEach(invoice -> {
    log.info(invoice.getInvoiceNumber() + " " + invoice.getKsefNumber() + " " + invoice.getStatus());
});
```

### 4. Retrieving Information About a Single Invoice

Allows retrieving detailed information about a single invoice in the session, including its status and metadata.

You must provide the session `referenceNumber` and the invoice `invoiceReferenceNumber`.

GET [sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D/get)

Example in C#:

```csharp
var invoice = await ksefClient
                .GetSessionInvoiceAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken);
```

Example in Java:

```java
SessionInvoice sessionInvoiceStatus = ksefClient.getSessionInvoiceStatus(referenceNumber, invoiceReferenceNumber);
```

### 5. Retrieving UPO for an Invoice

Allows retrieving a UPO for a single correctly accepted invoice.

#### 5.1 Based on the invoice reference number

GET [sessions/{referenceNumber}/invoices/{invoiceReferenceNumber}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1%7BinvoiceReferenceNumber%7D~1upo/get)

Example in C#:

```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByReferenceNumberAsync(
                referenceNumber,
                invoiceReferenceNumber,
                accessToken,
                cancellationToken)
```

Example in Java:

```java
var upo = ksefClient.getSessionInvoiceUpoByReferenceNumber(referenceNumber, invoiceReferenceNumber);
```

#### 5.2 Based on the KSeF invoice number

GET [sessions/{referenceNumber}/invoices/{ksefNumber}/upo](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1ksef~1%7BksefNumber%7D~1upo/get)

Example in C#:

```csharp
var upo = await ksefClient
                .GetSessionInvoiceUpoByKsefNumberAsync(
                referenceNumber,
                ksefNumber,
                accessToken,
                cancellationToken)
```

Example in Java:

```java
var upo = ksefClient.getSessionInvoiceUpoByKsefNumber(referenceNumber, ksefNumber);
```

The returned XML document is:

* signed in XADES format by the Ministry of Finance
* compliant with the [XSD schema](/faktury/upo-faktura.xsd) for single invoices.

### 6. Retrieving List of Rejected Invoices

GET [sessions/{referenceNumber}/invoices/failed](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1failed/get)

Returns the total number of rejected invoices in the session along with detailed information (status and error details) for each rejected invoice.

Example in C#:

```csharp
const int pageSize = 50;
string continuationToken = "";

do
{
    var sessionInvoices = await ksefClient
                                .GetSessionFailedInvoicesAsync(
                                referenceNumber,
                                accessToken,
                                pageSize,
                                continuationToken,
                                cancellationToken);

    continuationToken = failedResult.Invoices.ContinuationToken

}
while (!string.IsNullOrEmpty(continuationToken));
```

Example in Java:

```java
SessionInvoicesResponse sessionFailedInvoices = ksefClient.getSessionFailedInvoices(referenceNumber, pageSize);
sessionFailedInvoices.getInvoices().forEach(invoice->{
    log.info(invoice.getInvoiceNumber() + " " + invoice.getStatus());
});
```

This endpoint allows selective retrieval of only rejected invoices, which simplifies error analysis in sessions containing a large number of invoices.

### 7. Retrieving Session UPO

A session UPO is an aggregate acknowledgment of receipt for all correctly submitted invoices in the session.

After the session is closed, the response to the [status check](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get) (step 2 – Checking Session Status) includes not only invoice statistics but also a list of references to session UPOs.

Each `upo.pages[]` array element contains a UPO reference number (`referenceNumber`) and a link (`downloadUrl`) to retrieve it:

```json
"upo": {
    "pages": [
        {
            "referenceNumber": "20250901-EU-47FDBE3000-5961A5D232-BF",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-47FDBE3000-5961A5D232-BF"
        },
        {
            "referenceNumber": "20250901-EU-48D8488000-59667BB54C-C8",
            "downloadUrl": "/api/v2/sessions/20250901-SB-47FA636000-5960B49115-9D/upo/20250901-EU-48D8488000-59667BB54C-C8"
        }        
    ]
}
```

Using this list, the API client can fetch each UPO individually via the `downloadUrl`:
GET [/sessions/{referenceNumber}/upo/{upoReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1upo~1%7BupoReferenceNumber%7D/get)

The returned XML is compliant with the [XSD schema](/faktury/upo-sesja.xsd) and may include up to 10,000 invoice entries.

Example in C#:

```csharp
var upo = await ksefClient.GetSessionUpoAsync(
            sessionReferenceNumber,
            upoReferenceNumber,
            accesToken,
            cancellationToken
        );
```

Example in Java:

```java
var upo = ksefClient.getSessionUpo(referenceNumber, upoReferenceNumber);
```

## Related Documents

* [KSeF Number – Structure and Validation](numer-ksef_en.md)

```
```