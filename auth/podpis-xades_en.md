## XAdES Signature

[https://www.w3.org/TR/XAdES/](https://www.w3.org/TR/XAdES/)

### Allowed Signature Formats

* Enveloping
* Enveloped

> **Detached signatures are not accepted.**

### Allowed Transforms in XAdES Signature

* `http://www.w3.org/TR/1999/REC-xpath-19991116` – `not(ancestor-or-self::ds:Signature)`
* `http://www.w3.org/2002/06/xmldsig-filter2`
* `http://www.w3.org/2000/09/xmldsig#enveloped-signature`
* `http://www.w3.org/2000/09/xmldsig#base64`
* `http://www.w3.org/2006/12/xml-c14n11`
* `http://www.w3.org/2006/12/xml-c14n11#WithComments`
* `http://www.w3.org/2001/10/xml-exc-c14n#`
* `http://www.w3.org/2001/10/xml-exc-c14n#WithComments`
* `http://www.w3.org/TR/2001/REC-xml-c14n-20010315`
* `http://www.w3.org/TR/2001/REC-xml-c14n-20010315#WithComments`

---

### Allowed Certificate Types for XAdES Signature

* **Qualified certificate for individuals** – must include PESEL or NIP of a person authorized to act on behalf of a company
* **Qualified certificate for organizations** (seal certificate) – must include the company’s NIP
* **Trusted Profile (ePUAP)** – used by individuals to sign documents via [gov.pl](https://www.gov.pl/web/gov/podpisz-dokument-elektronicznie-wykorzystaj-podpis-zaufany)
* **Internal KSeF certificate** – issued by KSeF system (not a qualified certificate but accepted for authentication)

> **Qualified certificate** – A certificate issued by a qualified trust service provider listed in the [EU Trusted List (EUTL)](https://eidas.ec.europa.eu/efda/trust-services/browse/eidas/tls) in compliance with the eIDAS Regulation. Certificates from both Poland and other EU member states are accepted in KSeF.

---

### Required Attributes for Qualified Certificates

#### For Personal Qualified Certificates (Individuals)

| OID      | Name         | Description                           |
| -------- | ------------ | ------------------------------------- |
| 2.5.4.42 | givenName    | First name                            |
| 2.5.4.4  | surname      | Last name                             |
| 2.5.4.5  | serialNumber | Serial number (includes PESEL or NIP) |
| 2.5.4.3  | commonName   | Common name of certificate holder     |
| 2.5.4.6  | countryName  | Country code (ISO 3166 format)        |

**Recognized patterns in `serialNumber` field:**

* `(PNOPL|PESEL).*?(?<number>\d{11})`
* `(TINPL|NIP).*?(?<number>\d{10})`

---

#### For Organizational Qualified Certificates (Seals)

| OID      | Name                   | Description                         |
| -------- | ---------------------- | ----------------------------------- |
| 2.5.4.10 | organizationName       | Full legal name of the organization |
| 2.5.4.97 | organizationIdentifier | Organization identifier (e.g., NIP) |
| 2.5.4.3  | commonName             | Common name of the organization     |
| 2.5.4.6  | countryName            | Country code (ISO 3166 format)      |

**Prohibited attributes for organization certificates:**

| OID      | Name      | Description |
| -------- | --------- | ----------- |
| 2.5.4.42 | givenName | First name  |
| 2.5.4.4  | surname   | Last name   |

**Recognized patterns in `organizationIdentifier` field:**

* `(VATPL).*?(?<number>\d{10})`

---

### Certificate Fingerprint Authentication

If a qualified certificate lacks proper identifiers in the `OID.2.5.4.5` field, it may still be used for authentication after permissions are granted based on its **SHA-256 fingerprint**.
