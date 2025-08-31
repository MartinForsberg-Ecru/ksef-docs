## Offline Modes

**July 10, 2025**

### Overview

The KSeF system supports two primary invoicing modes:

* `online` – invoices are issued and submitted in real-time to KSeF,
* `offline` – invoices are issued electronically and submitted to KSeF at a later, legally defined time.

In offline mode, invoices must conform to the FA(3) structure. Key technical aspects include:

* When sending an invoice in either interactive or batch mode, set the parameter `offlineMode: true`.
* The issue date is taken from the `P_1` field of the invoice structure — not the time of submission.
* The receipt date is either the date a KSeF number is assigned or the actual receipt date if shared outside KSeF.
* After issuing an offline invoice, the client application must generate two [QR codes](kody-qr.md):

  * **Code I** – allows verification of the invoice in KSeF,
  * **Code II** – confirms the issuer’s identity.
* A correction invoice may only be submitted after the original invoice has received a KSeF number.
* If the offline invoice is rejected for technical reasons, you may use the [technical correction](offline/korekta-techniczna.md) mechanism.

---

### Comparison of Offline24, Offline, and Emergency Modes

| Mode          | Responsibility | Activation Conditions                                            | Deadline for Sending to KSeF                       | Legal Basis                                |
| ------------- | -------------- | ---------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------ |
| **offline24** | taxpayer       | No restrictions (at taxpayer's discretion)                       | by the next business day after the issue date      | Art. 106nda VAT Act (KSeF 2.0 draft)       |
| **offline**   | KSeF system    | System unavailability (announced via BIP and interface software) | by the next business day after unavailability ends | Art. 106nh VAT Act (effective Feb 1, 2026) |
| **emergency** | KSeF system    | Declared KSeF failure (announced via BIP and interface software) | within 7 business days after the end of the outage | Art. 106nf VAT Act (effective Feb 1, 2026) |

---

### Invoice Submission Deadlines with Overlapping Outages

If a KSeF outage is announced during the invoice submission window in either offline24 or offline modes, the submission deadline is reset — starting from the end of the latest declared outage, up to a maximum of 7 business days.

In emergency mode, if a new outage is declared during the 7-day period, the countdown resets and starts again from the end of the new outage.

If a **total system failure** is declared, submission of offline invoices is no longer required.

#### Example: offline24 Mode + Declared Outage

1. **2025-07-08 (Wednesday)**
   Taxpayer issues invoice in offline24 mode (`offlineMode = true`).
   Submission deadline is 2025-07-09 (next business day).
2. **2025-07-09 (Thursday)**
   Ministry of Finance declares a KSeF outage (via BIP/API).
   Deadline is postponed to the end of the outage.
3. **2025-07-12 (Saturday)**
   Outage ends — system restored.
   A 7-business-day period begins.
4. **2025-07-22 (Tuesday)**
   Deadline for submission. The invoice must be sent to KSeF with `offlineMode = true`.

---

### Total System Outage Mode

When a **total outage** is declared (via mass media like TV, radio, or press):

* Invoices may be issued in paper or electronic form, without FA(3) structure requirements.
* There is **no obligation** to submit the invoice to KSeF afterward.
* Delivery to the buyer can occur via any channel (in-person, email, etc.).
* The invoice issue date is the actual date written on it; the receipt date is the actual date of receipt.
* QR codes are **not required**.
* Correction invoices during the outage are issued outside KSeF, similarly with actual dates.

---

## Related Documents

* [Offline Invoice Technical Correction](offline/korekta-techniczna.md)
* [QR Codes](kody-qr.md)
