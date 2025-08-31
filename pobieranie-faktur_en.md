## Invoice Retrieval

### Retrieve Invoice by KSeF Number

21.08.2025

Returns the invoice with the specified KSeF number.

GET [/invoices/ksef/{ksefReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1ksef~1%7BksefNumber%7D/get)

C# Example:

```csharp
var invoice = await ksefClient.GetInvoiceAsync(ksefReferenceNumber, accessToken, cancellationToken);
```

Java Example:

```java
var document = ksefClient.getInvoice(ksefReferenceNumber);
```

### Retrieve Invoice Metadata List

Returns a list of invoice metadata matching the provided search criteria.

POST [/invoices/query/metadata](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1query~1metadata/post)

C# Example:

```csharp
var pagedInvoicesResponse = await ksefClient.QueryInvoiceMetadataAsync(body, accessToken, pageOffset, pageSize, cancellationToken);
```

Java Example:

```java
var queryInvoicesReponse = ksefClient.queryInvoices(pageOffset, pageSize, invoicesQueryRequest);
```

### Initialize Asynchronous Invoice Retrieval

Starts an asynchronous process for searching invoices in the KSeF system based on provided filters. The request must include encryption information in the `Encryption` field, which is used to encrypt the generated invoice packages.

POST [/invoices/exports](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports/post)

C# Example:

```csharp
// Add usage code here when available
```

Java Example:

```java
// Add usage code here when available
```

### Check Asynchronous Invoice Retrieval Status

Retrieves the status of a previously initialized asynchronous invoice retrieval operation using the operation reference number. Allows tracking the processing progress and downloading the result packages when ready.

GET [/invoices/async-query/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Pobieranie-faktur/paths/~1api~1v2~1invoices~1exports~1%7BoperationReferenceNumber%7D/get)

C# Example:

```csharp
// Add usage code here when available
```

Java Example:

```java
// Add usage code here when available
```
