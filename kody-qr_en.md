## QR Verification Codes

21.08.2025

A QR code (Quick Response) is a graphical representation of text, typically a URL. In the KSeF context, it is an encoded link containing invoice-identifying data — this format enables fast information retrieval using end-user devices (smartphones or optical scanners). The link can be scanned and redirected directly to the KSeF system resource responsible for invoice or certificate visualization and verification.

QR codes are intended for situations where the invoice reaches the recipient through channels other than direct KSeF API retrieval (e.g., PDF, printout, or email attachment). In such cases, anyone can:

* verify whether the invoice exists in the KSeF system and has not been altered,
* download its structured version (XML file) without contacting the issuer,
* confirm the issuer’s authenticity (for offline invoices).

Code generation (for both online and offline invoices) is done locally in the client application based on data from the issued invoice. The QR code must comply with ISO/IEC 18004:2015. If embedding the code in the invoice is not possible (e.g., due to format constraints), it must be delivered as a separate image file or link.

Depending on the issuance mode (online or offline), the invoice visualization includes:

* in **online** mode — one QR code (CODE I), enabling invoice verification and retrieval from KSeF,
* in **offline** mode — two QR codes:

  * **CODE I** for verification after sending the invoice to KSeF,
  * **CODE II** for verifying issuer authenticity using the [KSeF certificate](certyfikaty-KSeF_en.md).

### 1. CODE I – Invoice Verification and Retrieval

`CODE I` contains a link for verifying and reading the invoice in KSeF.
Upon scanning the QR code or clicking the link, the user sees a simplified invoice data view and confirmation of its presence in the system. Full access (e.g., XML download) requires additional input.

#### Link Generation

The link is composed of:

* URL base: `https://ksef-test.mf.gov.pl/client-app/invoice`,
* invoice issue date (`P_1`) in format DD-MM-YYYY,
* seller’s NIP,
* invoice hash (SHA-256, Base64URL-encoded).

Example:

* issue date: "01-02-2026",
* seller NIP: "1111111111",
* SHA-256 hash (Base64URL): "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE"

Resulting link:

```
https://ksef-test.mf.gov.pl/client-app/invoice/1111111111/01-02-2026/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

C# Example:

```csharp
var url = linkSvc.BuildInvoiceVerificationUrl(nip, issueDate, invoiceHash);
```

Java Example:

```java
String url = linkSvc.buildInvoiceVerificationUrl(nip, issueDate, xml);
```

#### QR Code Generation

C# Example:

```csharp
var qrCode = qrSvc.GenerateQrCode(url);
```

Java Example:

```java
byte[] qrCode = qrSvc.generateQrCode(url);
```

#### QR Labeling

Invoice acceptance by KSeF is usually immediate — the KSeF number is assigned promptly. In rare cases (e.g., system overload), there may be a slight delay.

* **If KSeF number is known**: display the KSeF number below the QR code (applies to online and sent offline invoices).

![QR KSeF](qr/qr-ksef.png)

* **If KSeF number is unknown**: label the QR code with **OFFLINE** (for offline invoices not yet sent, or online invoices awaiting a number).

![QR Offline](qr/qr-offline.png)

C# Example:

```csharp
var labeledQr = qrSvc.AddLabelToQrCode(qrCode, ksefNumber);
```

Java Example:

```java
byte[] labeledQr = qrSvc.addLabelToQrCode(qrCode, ksefNumber);
```

### 2. CODE II – Certificate Verification

`CODE II` is generated only for invoices issued in offline mode (offline24, offline-unavailability, emergency mode) and serves to verify the issuer’s authenticity and the invoice’s integrity. It requires an active [Offline-type KSeF certificate](certyfikaty-KSeF_en.md) — the link contains a cryptographic signature created using the private key from the Offline certificate, preventing forgery by unauthorized parties.

> **Note**: An `Authentication` certificate **must not** be used to generate CODE II. Its purpose is strictly for API authentication.

Offline certificates can be obtained via [`/certificates`](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Certyfikaty/paths/~1api~1v2~1certificates~1enrollments/post).

#### Link Generation

The link includes:

* base URL: `https://ksef-test.mf.gov.pl/client-app/certificate`,
* context identifier type: "Nip", "InternalId", "NipVatUe", "PeppolId",
* context identifier value,
* seller NIP,
* KSeF certificate serial number,
* SHA-256 invoice hash (Base64URL),
* cryptographic signature of the link (Base64URL).

**Signature format**
The signed portion excludes the protocol (`https://`) and trailing slash, e.g.:

```
ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE
```

**Signing algorithm:**

* RSA (RSASSA‑PSS): sign the string with the KSeF private key using RSA-OAEP and SHA-256 (MGF1). The resulting ciphertext must be Base64URL-encoded.

Example values:

* context type: "Nip",
* context ID: "1111111111",
* seller NIP: "1111111111",
* certificate serial: "01F20A5D352AE590",
* hash: "UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE",
* signature: "KFoN1Z91HvySb2uqP2ZDFKubKftzWtWsQOjqTdLFXA3ZEoC1PB8sixNi1LLwgnndwL-MkDcpaCEZxBuVtjWcBpAUJEYlnmDQFOMQs-ueSsp5uuVbmNb-d8\_yRTAQvSUdHNuIYfpy7Wpj1jTY0yghgOmdTQzpV5MrcHEReLKjpGONHj8lZtq7RpY43LlCfvctHbUWbqSYAC0C7zmNKz5ROK1LFE-QIi5qGqrRIdw1PCEEPW9tKcB94G3qAHXizNyp5TO6V\_0qZ9eC1DXtftkCocQD\_CEI3MY-nu7yk6LC7hF7yQ1t8GGRikhQP6w4OwBR-z048IdhCtWd-RYNnxiU6w"

Generated link:

```
https://ksef-test.mf.gov.pl/client-app/certificate/Nip/1111111111/1111111111/01F20A5D352AE590/UtQp9Gpc51y-u3xApZjIjgkpZ01js-J8KflSPW8WzIE/KFoN1Z91HvySb2uqP2ZDFKubKftzWtWsQOjqTdLFXA3ZEoC1PB8sixNi1LLwgnndwL-MkDcpaCEZxBuVtjWcBpAUJEYlnmDQFOMQs-ueSsp5uuVbmNb-d8_yRTAQvSUdHNuIYfpy7Wpj1jTY0yghgOmdTQzpV5MrcHEReLKjpGONHj8lZtq7RpY43LlCfvctHbUWbqSYAC0C7zmNKz5ROK1LFE-QIi5qGqrRIdw1PCEEPW9tKcB94G3qAHXizNyp5TO6V_0qZ9eC1DXtftkCocQD_CEI3MY-nu7yk6LC7hF7yQ1t8GGRikhQP6w4OwBR-z048IdhCtWd-RYNnxiU6w
```

C# Example:

```csharp
var cert = new X509Certificate2(Convert.FromBase64String(certbase64));
var url = linkSvc.BuildCertificateVerificationUrl(nip, certSerial, invoiceHash, cert, privateKey);
```

Java Example:

```java
String pem = privateKeyPemBase64.replaceAll("\\s+", "");
byte[] keyBytes = java.util.Base64.getDecoder().decode(pem);
PrivateKey privateKey = new DefaultCryptographyService(ksefClient).parsePrivateKeyFromPem(keyBytes);

String url = linkSvc.buildCertificateVerificationUrl(nip, certSerial, xml, privateKey);
```

#### QR Code Generation

C# Example:

```csharp
var qrCode = qrSvc.GenerateQrCode(url);
```

Java Example:

```java
byte[] qrCode = qrSvc.generateQrCode(url);
```

#### QR Labeling

The label **CERTYFIKAT** should appear under the QR code to indicate certificate verification functionality.

C# Example:

```csharp
var labeledQr = qrSvc.AddLabelToQrCode(qrCode, "CERTYFIKAT");
```

Java Example:

```java
byte[] labeledQr = qrSvc.addLabelToQrCode(qrCode, "CERTYFIKAT");
```

![QR Certificate](qr/qr-cert.png)
