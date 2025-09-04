```markdown
# KSeF Number – Structure and Validation

The KSeF number is a unique invoice identifier assigned by the system. It is always **35 characters** long and globally unique – it unambiguously identifies each invoice in KSeF.

## General Number Structure
```

9999999999-YYYYMMDD-FFFFFFFFFFFF-FF

```
Where:
- `9999999999` – Seller's NIP (10 digits),
- `YYYYMMDD` – Date the invoice was accepted for processing (year, month, day),
- `FFFFFFFFFFFF` – Technical part consisting of 12 hexadecimal characters, only [0–9 A–F], uppercase,
- `FF` – CRC-8 checksum – 2 hexadecimal characters, only [0–9 A–F], uppercase.

## Example
```

5265877635-20250826-0100001AF629-AF

````
- `5265877635` – Seller's NIP,
- `20250826` – Date of invoice acceptance,
- `0100001AF629` – Technical part,
- `AF` – CRC-8 checksum.

## KSeF Number Validation

The validation process includes:
1. Checking if the number has **exactly 35 characters**.  
2. Splitting the data part (32 characters) and the checksum (2 characters).  
3. Calculating the checksum from the data part using the **CRC-8 algorithm**.  
4. Comparing the calculated checksum with the value in the number.

## CRC-8 Algorithm

The **CRC-8** algorithm is used to compute the checksum with the following parameters:

- **Polynomial:** `0x07`  
- **Initial value:** `0x00`  
- **Output format:** 2-character hexadecimal string (HEX, uppercase)

Example: If the calculated checksum is `0x46`, the KSeF number will end with `"46"`.

## Example in C#:
```csharp
using KSeF.Client.Core;

bool isValid = KsefNumberValidator.IsValid(ksefNumber, out string message);
````

## Example in Java:

```java
// Implementation not provided
```

```
```