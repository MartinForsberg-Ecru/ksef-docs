## Sample Scenarios

05.08.2025

### Scenario 1 – Bailiff

To use the KSeF system as a natural person with bailiff privileges in the test environment, you must add the person using the `/v2/testdata/person` endpoint, setting the *isBailiff* flag to **true**.

Example JSON:

```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "Komornik",
  "isBailiff": true
}
```

As a result, a person logging in under the given NIP using their PESEL or NIP will receive **Owner** and **EnforcementOperations** permissions, allowing them to operate in KSeF as a bailiff.

---

### Scenario 2 – Sole Proprietorship (JDG)

To use the KSeF system as a sole proprietor in the test environment, add the person using the `/v2/testdata/person` endpoint with the *isBailiff* flag set to **false**.

Example JSON:

```json
{
  "nip": "7980332920",
  "pesel": "30112206276",
  "description": "JDG",
  "isBailiff": false
}
```

As a result, the person logging in under the given NIP using their PESEL or NIP will receive **Owner** permissions, enabling them to operate in KSeF as a JDG entity.

---

### Scenario 3 – VAT Group

To simulate a VAT group structure and assign permissions to the group administrator and member administrators in the test environment, first create the entity structure using the `/v2/testdata/subject` endpoint by specifying the NIP of the parent and subunits.

Example JSON:

```json
{
  "subjectNip": "3755747347",
  "subjectType": "VatGroup",
  "description": "Grupa VAT",
  "subunits": [
    {
      "subjectNip": "4972530874",
      "description": "NIP 4972530874: member of VAT group for 3755747347"
    },
    {
      "subjectNip": "8225900795",
      "description": "NIP 8225900795: member of VAT group for 3755747347"
    }
  ]
}
```

This creates the entities and their relationships in the system. Next, assign permissions to a person in the context of the VAT group's NIP according to ZAW-FA rules using the `/v2/testdata/permissions` method.

Example JSON for a person authorized under the VAT group context:

```json
{
  "contextIdentifier": {
    "value": "3755747347",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "38092277125",
    "type": "pesel"
  },
  "permissions": [
    {
      "permissionType": "InvoiceRead",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "InvoiceWrite",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "Introspection",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsRead",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "CredentialsManage",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    },
    {
      "permissionType": "SubunitManage",
      "description": "working in context 3755747347: authorized PESEL: 38092277125, Adam Abacki"
    }
  ]
}
```

This operation can be performed for both the VAT group and its members. While this is the only way to assign initial permissions for the group itself, member administrators can alternatively be appointed using the standard `/v2/permissions/subunit/grants` endpoint.

Alternatively, you can use the test data endpoint shown above. Example JSON to assign `CredentialsManage` permission to a VAT group member administrator:

```json
{
  "contextIdentifier": {
    "value": "4972530874",
    "type": "nip"
  },
  "authorizedIdentifier": {
    "value": "3388912629",
    "type": "nip"
  },
  "permissions": [
    {
      "permissionType": "CredentialsManage",
      "description": "working in context 4972530874: authorized NIP: 3388912629, Bogdan Babacki"
    }
  ]
}
```

This enables a VAT group member's representative to manage permissions for themselves or others (e.g., employees) via the KSeF system.

### Scenario 4 – Enabling Invoice Attachments

In the test environment, you can simulate an entity with the ability to send invoices with attachments. Use the `/api/v2/testdata/attachment` endpoint.

```json
{
  "nip": "4972530874"
}
```

As a result, the entity with NIP 4972530874 will be able to send invoices with attachments.

### Scenario 5 – Disabling Invoice Attachments

To test a situation where an entity can no longer send invoices with attachments, use the `/api/v2/testdata/attachment/revoke` endpoint.

```json
{
  "nip": "4972530874"
}
```

As a result, the entity with NIP 4972530874 will lose the ability to send invoices containing attachments.
