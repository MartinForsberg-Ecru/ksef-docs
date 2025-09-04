## KSeF API 2.0 Environments
01.09.2025

Below is a summary of the public environments.

| Code   | Environment                   | Description                                                                                                                                   | Address (API)                                                                                           | Availability Date         |
|--------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|---------------------------|
| **TE**  | Test (Release Candidate)      | RC environment for early verification of planned changes before production deployment. Allows testing via both the Taxpayer Application and custom API calls. | [https://ksef-test.mf.gov.pl/api/v2](https://ksef-test.mf.gov.pl/api/v2)                             | 30.09.2025                |
| **TR**  | Pre-production (Demo/Preprod) | Environment matching the production configuration, intended for final integration validation in conditions similar to production.            | [https://ksef-demo.mf.gov.pl/api/v2](https://ksef-demo.mf.gov.pl/api/v2)                             | 15.10.2025               |
| **PRD** | Production                    | Environment for issuing and receiving legally binding invoices, with guaranteed SLA and valid production data.                               | [https://ksef.mf.gov.pl/api/v2](https://ksef.mf.gov.pl/api/v2)                                       | 01.02.2026               |

Documentation for each environment is available at `/docs/v2`.
```