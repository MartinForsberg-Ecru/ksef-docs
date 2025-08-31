# KSeF Number – Structure and Validation

The KSeF number is a unique invoice identifier assigned by the system. It always has a fixed length of **35 characters** and is globally unique – it unambiguously identifies each invoice in KSeF.

## General Number Structure

```
9999999999-RRRRMMDD-FFFFFFFFFFFF-FF  
```

Where:

* `9999999999` – Seller’s NIP (10 digits),
* `RRRRMMDD` – Date of invoice acceptance (year, month, day),
* `FFFFFFFFFFFF` – Technical part consisting of 12 alphanumeric characters,
* `FF` – Checksum (2 characters, CRC-8 in HEX format, uppercase).

## Example

```
5265877635-20250826-0100001AF629-AF
```

* `5265877635` – Seller’s NIP,
* `20250826` – Date of invoice acceptance for processing,
* `0100001AF629` – Technical part,
* `AF` – CRC-8 checksum.

## KSeF Number Validation

The validation process includes:

1. Verifying that the number is **exactly 35 characters** long.
2. Splitting the number into the data part (32 characters) and checksum (2 characters).
3. Calculating the checksum from the data part using the **CRC-8 algorithm**.
4. Comparing the calculated checksum to the one provided in the number.

## CRC-8 Algorithm

To calculate the checksum, the **CRC-8** algorithm is used with the following parameters:

* **Polynomial:** `0x07`
* **Initial value:** `0x00`
* **Output format:** 2-character hexadecimal (HEX, uppercase)

Example: if the calculated checksum is `0x46`, `"46"` will be appended to the KSeF number.

## Example in C#:

```csharp
using KSeF.Client.Core;

bool isValid = KsefNumberValidator.IsValid(ksefNumber, out string message);
```

## Example in Java:

```java
```
