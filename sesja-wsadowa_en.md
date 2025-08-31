## Batch Session

10.07.2025

Batch submission allows sending multiple invoices in a single ZIP file instead of submitting each one individually.

This method speeds up and simplifies the processing of large volumes of documents—especially useful for companies generating many invoices daily.

Each invoice must be prepared in XML format according to the current schema published by the Ministry of Finance:

* The ZIP package should be divided into parts no larger than 100 MB (before encryption), which are encrypted and uploaded separately.
* You must provide details of each part in the `fileParts` object.

### Prerequisites

To use batch submission, you must first complete the [authentication process](uwierzytelnianie_en.md) and obtain a valid access token (`accessToken`), authorizing access to protected KSeF API resources.

Before opening a session and sending invoices, you need to:

* Generate a 256-bit symmetric key and a 128-bit initialization vector (IV), prefixed to the ciphertext,
* Encrypt the document using AES-256-CBC with PKCS#7 padding,
* Encrypt the symmetric key using RSAES-OAEP (padding OAEP with MGF1 and SHA-256 hash), using the Ministry of Finance's public KSeF key.

These operations can be handled using the `CryptographyService` component available in the official KSeF client, which provides ready-made methods for key generation and encryption as per system requirements.

C# Example:

```csharp
EncryptionData encryptionData = cryptographyService.GetEncryptionData();
```

Java Example:

```java
var cryptographyService = new DefaultCryptographyService(ksefClient);
EncryptionData encryptionData = cryptographyService.getEncryptionData();
```

The generated data is used to encrypt the invoices.

### 1. Preparing the ZIP Package

Create a ZIP package containing all invoices to be submitted within a single session.

C# Example:

```csharp
// Load structured invoice (XML) files into memory
var files = invoices.Select(f => new { FileName = Path.GetFileName(f), Content = System.IO.File.ReadAllBytes(f) }).ToList();

// Create ZIP in memory
byte[] zipBytes;
using (var zipStream = new MemoryStream())
{
    using (var archive = new ZipArchive(zipStream, ZipArchiveMode.Create, true))
    {
        foreach (var file in files)
        {
            var entry = archive.CreateEntry(file.FileName, CompressionLevel.Optimal);
            using var entryStream = entry.Open();
            entryStream.Write(file.Content, 0, file.Content.Length);
        }
    }
    zipBytes = zipStream.ToArray();
}
```

Java Example:

```java
List<Path> invoices = ....;
byte[] zipBytes;
try (ByteArrayOutputStream zipStream = new ByteArrayOutputStream();
     ZipOutputStream archive = new ZipOutputStream(zipStream)) {

    for (Path file : invoices) {
        archive.putNextEntry(new ZipEntry(file.getFileName().toString()));
        byte[] fileContent = Files.readAllBytes(file);
        archive.write(fileContent);
        archive.closeEntry();
    }
    archive.finish();
    zipBytes = zipStream.toByteArray();
}
```

### 2. Binary Splitting of ZIP Package into Parts

Due to size limitations, the ZIP package should be binary-split into smaller parts for separate upload. Each part must have a unique name and order number.

C# Example:

```csharp
// Get metadata of ZIP (before encryption)
var zipMetadata = cryptographyService.GetMetaData(zipBytes);

// Split ZIP into 11 parts
int partCount = 11;
int partSize = (int)Math.Ceiling((double)zipBytes.Length / partCount);
var zipParts = new List<byte[]>();
for (int i = 0; i < partCount; i++)
{
    int start = i * partSize;
    int size = Math.Min(partSize, zipBytes.Length - start);
    if (size <= 0) break;
    var part = new byte[size];
    Array.Copy(zipBytes, start, part, 0, size);
    zipParts.Add(part);
}
```

Java Example:

```java
int numberOfParts = 11;
var zipMetadata = cryptographyService.getMetaData(zipBytes);
int partSize = (int) Math.ceil((double) zipBytes.length / numberOfParts);

List<byte[]> zipParts = new ArrayList<>();
for (int i = 0; i < numberOfParts; i++) {
    int start = i * partSize;
    int size = Math.min(partSize, zipBytes.length - start);
    if (size <= 0) break;
    byte[] part = Arrays.copyOfRange(zipBytes, start, start + size);
    zipParts.add(part);
}
```

### 3. Encrypting Package Parts

Each part must be encrypted using a newly generated AES-256-CBC key with PKCS#7 padding.

C# Example:

```csharp
var encryptedParts = new List<(byte[] Data, FileHash Metadata)>();
for (int i = 0; i < zipParts.Count; i++)
{
    var encrypted = cryptographyService.EncryptBytesWithAES256(zipParts[i], encryptionData.CipherKey, encryptionData.CipherIv);
    var metadata = cryptographyService.GetMetaData(encrypted);
    encryptedParts.Add((encrypted, metadata));

    // Save part to disk
    var partFileName = Path.Combine(BatchPartsDirectory, $"faktura_part{i + 1}.zip.aes");
    System.IO.File.WriteAllBytes(partFileName, encrypted);
}
```

Java Example:

```java
List<BatchPartSendingInfo> encryptedZipParts = new ArrayList<>();
for (int i = 0; i < zipParts.size(); i++) {
    byte[] encryptedZipPart = cryptographyService.encryptBytesWithAES256(
            zipParts.get(i),
            encryptionData.cipherKey(),
            encryptionData.cipherIv()
    );
    FileMetadata zipPartMetadata = cryptographyService.getMetaData(encryptedZipPart);
    encryptedZipParts.add(new BatchPartSendingInfo(encryptedZipPart, zipPartMetadata, (i + 1)));
}
```

### 4. Opening a Batch Session

Initialize a new batch session by providing:

* invoice schema version: [FA(2)](faktury/schemat-FA%282%29-v1-0E.xsd), [FA(3)](faktury/schemat-FA%283%29-v1-0E.xsd)<br>
* encrypted symmetric key<br>
* metadata of the ZIP package and its parts

POST [/sessions/batch](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/operation/batch.open)

The response contains a `referenceNumber` used to identify the batch session in later steps.

C# Example:

```csharp
var batchFileInfoBuilder = OpenBatchSessionRequestBuilder
    .Create()
    .WithFormCode(systemCode: "FA (2)", schemaVersion: "1-0E", value: "FA")
    .WithBatchFile("faktury.zip", zipMetadata.FileSize, zipMetadata.HashSHA.Value);

for (int i = 0; i < encryptedParts.Count; i++)
{
    batchFileInfoBuilder = batchFileInfoBuilder.AddBatchFilePart(
        i + 1,
        $"faktura_part{i + 1}.zip.aes",
        encryptedParts[i].Metadata.FileSize,
        encryptedParts[i].Metadata.HashSHA.Value);
}

var openBatchRequest = batchFileInfoBuilder.EndBatchFile()
    .WithEncryption(encryptionData.EncryptionInfo.EncryptedSymmetricKey, encryptionData.EncryptionInfo.InitializationVector)
    .Build();

var openBatchSessionResponse = await ksefClient.OpenBatchSessionAsync(openBatchRequest, accessToken, cancellationToken);
```

Java Example:

```java
var builder = OpenBatchSessionRequestBuilder.create()
    .withFormCode("FA (2)", "1-0E", "FA")
    .withOfflineMode(false)
    .withBatchFile(zipMetadata.getFileSize(), zipMetadata.getHashSHA());

for (int i = 0; i < encryptedZipParts.size(); i++) {
    var part = encryptedZipParts.get(i);
    builder = builder.addBatchFilePart(i + 1, "faktura_part" + (i + 1) + ".zip.aes",
            part.getMetadata().getFileSize(), part.getMetadata().getHashSHA());
}

OpenBatchSessionRequest request = builder.endBatchFile()
    .withEncryption(
            encryptionData.encryptionInfo().getEncryptedSymmetricKey(),
            encryptionData.encryptionInfo().getInitializationVector()
    )
    .build();

var openBatchSessionResponse = ksefClient.openBatchSession(request);
```

The method returns a list of parts and for each part, it includes the upload URL, required HTTP method, and all necessary headers.

### 5. Uploading the Declared Parts

Using the information returned during session opening (unique URL, HTTP method, required headers), upload each declared package part accordingly.

C# Example:

```csharp
await ksefClient.SendBatchPartsAsync(openBatchSessionResponse, BatchPartsDirectory);
```

Java Example:

```java
ksefClient.sendBatchParts(openBatchSessionResponse, encryptedZipParts);
```

### 6. Closing the Batch Session

Once all parts are uploaded, close the batch session to trigger asynchronous invoice processing ([verification details](faktury/weryfikacja-faktury_en.md)) and UPO generation.

POST [/sessions/batch/{referenceNumber}/close](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wysylka-wsadowa/paths/~1api~1v2~1sessions~1batch~1%7BreferenceNumber%7D~1close/post)

C# Example:

```csharp
var closeBatchSessionRequest = CloseBatchSessionRequestBuilder
    .Create()
    .WithReferenceNumber(openBatchSessionResponse.ReferenceNumber)
    .Build();
var closeBatchSessionResponse = await ksefClient.CloseBatchSessionAsync(closeBatchSessionRequest, accessToken);
```

Java Example:

```java
ksefClient.closeBatchSession(referenceNumber);
```

See also:

* [Session Status and UPO Download](faktury/sesja-sprawdzenie-stanu-i-pobranie-upo_en.md)
* [Invoice Verification](faktury/weryfikacja-faktury_en.md)
* [KSeF Number – Structure and Validation](numer-ksef_en.md)
