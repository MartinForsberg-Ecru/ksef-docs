# Technical Correction of Offline Invoice

20.08.2025

## Functionality Description

A technical correction allows for the resubmission of an invoice issued in [offline mode](../tryby-offline_en.md), which was **rejected** by the KSeF system due to technical errors, such as:

* schema non-compliance,
* exceeding the permissible file size,
* duplicate invoice,
* other **technical validation errors** preventing assignment of a `KSeF number`.

> **Note**!

1. A technical correction **does not apply** to situations involving lack of authorization of the entities appearing on the invoice (e.g., self-invoicing, validation of relationships for local governments or VAT groups).
2. This mode **does not permit** correcting the content of the invoice – the technical correction applies only to technical issues preventing its acceptance by the KSeF system.
3. A technical correction can only be submitted in an [interactive session](../sesja-interaktywna_en.md), but it may concern offline invoices rejected in both [interactive sessions](../sesja-interaktywna_en.md) and [batch sessions](../sesja-wsadowa_en.md).
4. It is not allowed to technically correct an offline invoice for which another valid correction has already been accepted.

## Example Flow of Technical Correction of Offline Invoice

1. **The seller issues an invoice in offline mode.**

   * The invoice contains two QR codes:

     * **QR Code I** – enables invoice verification in the KSeF system,
     * **QR Code II** – allows for confirmation of the issuer’s authenticity based on the [KSeF certificate](../certyfikaty-KSeF_en.md).

2. **The customer receives a visualization of the invoice (e.g., as a printout).**

   * After scanning **QR Code I**, the customer is informed that the invoice **has not yet been submitted to the KSeF system**.
   * After scanning **QR Code II**, the customer receives information about the KSeF certificate confirming the issuer’s authenticity.

3. **The seller submits the offline invoice to the KSeF system.**

   * The KSeF system verifies the document.
   * The invoice is **rejected** due to a technical error (e.g., invalid XSD schema compliance).

4. **The seller updates their software** and regenerates the invoice with the same content, but in compliance with the schema.

   * Since the XML content differs from the original version, the **SHA-256 hash of the invoice file is different**.

5. **The seller sends the corrected invoice as a technical correction.**

   * The `hashOfCorrectedInvoice` field includes the SHA-256 hash of the original rejected offline invoice.
   * The `offlineMode` parameter is set to `true`.

6. **The KSeF system successfully accepts the invoice.**

   * The document receives a KSeF number.
   * The invoice is **linked to the original offline invoice**, whose hash was provided in the `hashOfCorrectedInvoice` field.
   * This enables redirecting the client from the “old” QR Code I to the corrected invoice.

7. **The customer uses QR Code I placed on the original invoice.**

   * The KSeF system informs that the **original invoice was technically corrected**.
   * The customer receives metadata of the new, correctly processed invoice and can download it from the system.

## Sending the Correction

The correction is submitted in accordance with the principles described in the [interactive session](../sesja-interaktywna_en.md) document, with additional settings:

* `offlineMode: true`,
* `hashOfCorrectedInvoice` – the hash of the original invoice.

Example in C#:

```csharp
var sendOnlineInvoiceRequest = SendInvoiceOnlineSessionRequestBuilder
    .Create()
    .WithInvoiceHash(invoiceMetadata.HashSHA, invoiceMetadata.FileSize)
    .WithEncryptedDocumentHash(
        encryptedInvoiceMetadata.HashSHA, encryptedInvoiceMetadata.FileSize)
    .WithEncryptedDocumentContent(Convert.ToBase64String(encryptedInvoice))
    .WithOfflineMode(true)
    .WithHashOfCorrectedInvoice(hashOfCorrectedInvoice)    
    .Build();
```

Example in Java:

```java
```

## Related Documents

* [Offline Modes](tryby-offline_en.md)
* [QR Codes](kody-qr_en.md)
