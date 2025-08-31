## Interactive Session

10.07.2025

The interactive session is used to submit individual structured invoices to the KSeF API. Each invoice must be prepared in XML format in accordance with the current schema published by the Ministry of Finance.

### Prerequisites

To use the interactive submission method, you must first complete the [authentication process](uwierzytelnianie_en.md) and obtain a valid access token (`accessToken`), authorizing access to protected KSeF API resources.

Before opening a session and sending invoices, the following steps are required:

* Generate a 256-bit symmetric key and a 128-bit initialization vector (IV), which is prefixed to the ciphertext,
* Encrypt the document using AES-256-CBC with PKCS#7 padding,
* Encrypt the symmetric key using RSAES-OAEP (padding OAEP with MGF1 and SHA-256 hash), using the Ministry of Finance's public KSeF key.

These operations can be performed using the `CryptographyService` component available in the KSeF client.

C# Example:

```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```

Java Example:

```java
var cryptographyService = new DefaultCryptographyService(ksefClient);
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

### 1. Opening a Session

Start a new interactive session by providing:

* the invoice schema version: [FA(2)](faktury/schemat-FA%282%29-v1-0E.xsd), [FA(3)](faktury/schemat-FA%283%29-v1-0E.xsd)<br>
  This determines which XSD version will be used for validating the submitted invoices.
* the encrypted symmetric key<br>
  The symmetric key for encrypting XML files, encrypted with the Ministry's public key. It is recommended to use a new key for each session.

Opening a session is lightweight and synchronous — multiple sessions can be maintained concurrently under one authentication.

POST [sessions/online](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/operation/onlineSession.open)

The response includes:

* `referenceNumber` – a unique ID of the interactive session to be used in all subsequent API calls.
* `validUntil` – Session expiry time. Sessions remain active for 12 hours from creation.

C# Example:

```csharp
 var request = OpenOnlineSessionRequestBuilder
         .Create()
         .WithFormCode(systemCode: "FA (2)", schemaVersion: "1-0E", value: "FA")
         .WithEncryption(
             encryptedSymmetricKey: encryptionData.EncryptionInfo.EncryptedSymmetricKey,
             initializationVector: encryptionData.EncryptionInfo.InitializationVector)
         .Build();

 var openSessionResponse = await ksefClient.OpenOnlineSessionAsync(request, accessToken, cancellationToken);
```

Java Example:

```java
var request = new OpenOnlineSessionRequestBuilder()
        .withFormCode(new FormCode("FA (2)","FA","1-0E"))
        .withEncryptionInfo(encryptionData.encryptionInfo())
        .build();

var response = ksefClient.openOnlineSession(request);
```

### 2. Sending an Invoice

Send the encrypted invoice using:

POST [sessions/online/{referenceNumber}/invoices/](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1invoices/post)

The response includes a `referenceNumber` used to identify the invoice in further operations (e.g., listing documents).

After successful submission, asynchronous invoice verification begins ([details here](faktury/weryfikacja-faktury_en.md)).

C# Example:

```csharp
var encryptedInvoice = cryptographyService.EncryptBytesWithAES256(invoice, encryptionData.CipherKey, encryptionData.CipherIv);

var invoiceMetadata = cryptographyService.GetMetaData(invoice);
var encryptedInvoiceMetadata = cryptographyService.GetMetaData(encryptedInvoice);
var sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA.Value, invoiceMetadata.FileSize)
    .WithEncryptedInvoiceHash(
       encryptedInvoiceMetadata.HashSHA.Value, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedInvoiceContent(Convert.ToBase64String(encryptedInvoice))
    .Build();
var sendInvoiceResponse = await ksefClient.SendOnlineSessionInvoiceAsync(sendOnlineInvoiceRequest, referenceNumber, accesToken, cancellationToken);
```

Java Example:

```java
byte[] invoice = ...;
var encryptedInvoice = cryptographyService.encryptBytesWithAES256(invoice,
        encryptionData.cipherKey(),
        encryptionData.cipherIv());

var invoiceMetadata = cryptographyService.getMetaData(invoice);
var encryptedInvoiceMetadata = cryptographyService.getMetaData(encryptedInvoice);

var sendInvoiceRequest = new SendInvoiceRequestBuilder()
        .withInvoiceHash(invoiceMetadata.getHashSHA())
        .withInvoiceSize(invoiceMetadata.getFileSize())
        .withEncryptedInvoiceHash(encryptedInvoiceMetadata.getHashSHA())
        .withEncryptedInvoiceSize(encryptedInvoiceMetadata.getFileSize())
        .withEncryptedInvoiceContent(Base64.getEncoder().encodeToString(encryptedInvoice))
        .build();

SendInvoiceResponse sendInvoiceResponse = defaultKsefClient.onlineSessionSendInvoice(referenceNumber, sendInvoiceRequest);
```

### 3. Closing the Session

After all invoices are submitted, the session should be closed to trigger asynchronous UPO (confirmation) generation.

POST [/sessions/online/{referenceNumber}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-interaktywna/paths/~1api~1v2~1sessions~1online~1%7BreferenceNumber%7D~1close/post)

The UPO will be available after checking session status.

C# Example:

```csharp
 var closeOnlineSessionResponse = await ksefClient.CloseOnlineSessionAsync(
            sessionReferenceNumber,
            cancellationToken
        );    
```

Java Example:

```java
ksefClient.closeOnlineSession(referenceNumber);
```

Related documents:

* [Session Status and UPO Download](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo_en.md)
* [Invoice Verification](faktury/weryfikacja-faktury_en.md)
* [KSeF Number – Structure and Validation](numer-ksef_en.md)
