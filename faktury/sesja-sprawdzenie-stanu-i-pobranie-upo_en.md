## Session – Status Check and UPO Retrieval

10.07.2025

This document describes operations for monitoring the status of a session (interactive or batch) and retrieving the UPO for invoices and the entire session.

### 1. Retrieve the list of sessions

Returns a list of sessions matching the given search criteria.

GET [sessions](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions/get)

Returns the current session status along with aggregated data on the number of submitted, successfully and unsuccessfully processed invoices; after session closure, it additionally provides a list of references to the summary UPO.

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
```

Example in Java:

```java
int pageSize = 10;
String continuationToken = null;
SessionsQueryRequest request = new SessionsQueryRequest();
request.setSessionType(SessionType.ONLINE); // interactive session
//request.setSessionType(SessionType.BATCH); // batch session
SessionsQueryResponse sessionsQueryResponse = defaultKsefClient.getSessions(request, pageSize, continuationToken);
```

### 2. Check session status

Checks the current status of a session.

GET [sessions/{referenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D/get)

Returns the current session status along with aggregated data on the number of submitted, successfully and unsuccessfully processed invoices; after session closure, it additionally provides a list of references to the summary UPO.

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

### 3. Retrieve sent invoice information

GET [sessions/{referenceNumber}/invoices](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices/get)

Returns a list of metadata for all submitted invoices along with their statuses and the total number of invoices in the session.

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

### 4. Retrieve information about a single invoice

Allows retrieval of detailed information about a single invoice in a session, including its status and metadata.

You must provide the session reference number `referenceNumber` and the invoice reference number `invoiceReferenceNumber`.

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

### 5. Retrieve the UPO for an invoice

Allows retrieval of the UPO for a single successfully accepted invoice.

#### 5.1 Using the invoice reference number

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

#### 5.2 Using the KSeF number

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

* digitally signed in XADES format by the Ministry of Finance,
* compliant with the [XSD schema](/faktury/upo-faktura_en.xsd) for a single invoice.

### 6. Retrieve the list of rejected invoices

GET [sessions/{referenceNumber}/invoices/failed](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1invoices~1failed/get)

Returns the total number of rejected invoices in the session and detailed information (status and error details) for each incorrectly processed invoice.

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

### 7. Retrieve session UPO

Provides a summary UPO confirming the acceptance of all invoices submitted in the given session.

GET [/sessions/{referenceNumber}/upo/{upoReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Status-wysylki-i-UPO/paths/~1api~1v2~1sessions~1%7BreferenceNumber%7D~1upo~1%7BupoReferenceNumber%7D/get)

The returned XML document complies with the [XSD schema](/faktury/upo-sesja_en.xsd).

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
