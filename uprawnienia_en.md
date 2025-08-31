## Permissions

**10.07.2025**

### Introduction – Business Context

The KSeF system introduces an advanced permission management mechanism, which is foundational for secure and regulation-compliant usage by various entities. Permissions determine who can perform specific operations in KSeF—such as issuing invoices, reviewing documents, granting further permissions, or managing subordinate units.

### Objectives of Permission Management:

* **Data security** – limiting access to information only to persons and entities formally authorized.
* **Regulatory compliance** – ensuring operations are performed by appropriate entities in accordance with legal requirements (e.g., VAT Act).
* **Auditability** – every operation related to granting or revoking permissions is logged and can be analyzed.

### Who Can Grant Permissions?

Permissions may be granted by:

* the **Owner** of an entity,
* an **administrator of a subordinate entity**,
* an **administrator of a subunit**,
* an **administrator of an EU entity**,
* an **administrator of the entity**, i.e., another entity or person with the `CredentialsManage` permission.

In practice, this means that each organization must manage its employees’ permissions—for example, assigning permissions to an accounting staff member when onboarding a new employee or revoking permissions when the employee leaves.

### When Are Permissions Granted?

#### Examples:

* at the start of a new employee's cooperation,
* when a company collaborates with an accounting office and needs to grant invoice read access so the office can process invoices,
* due to changes in relationships between entities.

### Structure of Granted Permissions:

Permissions are granted to:

1. **Individuals** identified by PESEL, NIP, or certificate fingerprint—working in KSeF:

   * in the context of the granting entity (direct permissions), or
   * in the context of another or multiple entities:

     * subordinate entity identified by NIP (e.g., a municipal unit or VAT group member),
     * subunit identified by an internal identifier,
     * complex NIP–VAT UE context linking a Polish entity to an EU self‑billing entity,
     * a specific entity (by NIP) – the grant recipient's client (selective, indirect permission),
     * all entities—clients of the granting entity (general, indirect permissions).
2. **Other entities** identified by NIP,

   * as final recipients of invoice issuance or viewing permissions,
   * as intermediaries with permission delegation enabled, allowing further indirect permission granting (see points iv and v above).
3. **Entities acting in their own context on behalf of the granting entity** (entity-level permissions):

   * tax representatives,
   * self‑invoicing entities,
   * VAT RR invoice issuers.

Access to system functions depends on the context of authentication and the scope of permissions granted to the authenticated entity/person in that context.

---

## Glossary (KSeF Permissions Context)

| Term                                         | Definition                                                                                                  |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Permission**                               | Authorization to perform specific operations in KSeF, e.g., `InvoiceWrite`, `CredentialsManage`.            |
| **Owner**                                    | The owner of an entity—with full access in the entity’s context (defined by matching NIP or linked PESEL).  |
| **Administrator of a subordinate entity**    | Person with `CredentialsManage` in a subordinate entity context; can grant permissions like `InvoiceWrite`. |
| **Subunit administrator**                    | Person with `CredentialsManage` in a subunit context; can grant permissions like `InvoiceWrite`.            |
| **EU entity administrator**                  | Person with `CredentialsManage` in a `NipVatUe` complex context; can grant permissions like `InvoiceRead`.  |
| **Intermediary entity**                      | Entity with `canDelegate = true` that can pass on `InvoiceRead` and `InvoiceWrite` indirectly.              |
| **Target entity**                            | Entity in whose context the permission applies (e.g., a company serviced by an accounting office).          |
| **Granted directly**                         | Permission assigned directly by owner or administrator.                                                     |
| **Granted indirectly**                       | Permission assigned via an intermediary for `InvoiceRead` and `InvoiceWrite`.                               |
| **`canDelegate`**                            | A flag (`true`/`false`) that allows granting `InvoiceRead` or `InvoiceWrite` indirectly.                    |
| **`subjectIdentifier`**                      | Identifier of the permission recipient: `Nip`, `Pesel`, or `Fingerprint`.                                   |
| **`targetIdentifier` / `contextIdentifier`** | Identifier of the area in which the permission operates: client NIP or subunit identifier.                  |
| **Fingerprint**                              | SHA‑256 hash of a qualified certificate, used when no NIP or PESEL is present.                              |
| **InternalId**                               | Internal identifier of a subunit: a composite of NIP and 5 digits (`nip-5_digits`).                         |
| **NipVatUe**                                 | Composite identifier: Polish entity's NIP plus EU VAT number (`nip-vat_ue`).                                |
| **Revocation**                               | The act of removing previously granted permission.                                                          |
| **`permissionId`**                           | Technical identifier for a granted permission, used in revocation requests.                                 |
| **`operationReferenceNumber`**               | Identifier for grant/revoke operations, returned by the API to track status.                                |
| **Operation status**                         | Current state of the grant/revoke process: `100`, `200`, `400`, etc.                                        |

---

## Roles & Permissions Model (Permissions Matrix)

KSeF enables fine-grained permission assignments, both direct and indirect, allowing delegation of access appropriately.

### Role Examples Represented via Permissions:

| Role/Entity                         | Description                                                             | Possible Permissions                                           |
| ----------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------- |
| **Owner of an entity**              | Automatically full access in entity’s context (matching NIP/PESEL)      | All invoice and administrative rights, except `VatUeManage`.   |
| **Entity administrator**            | Person who can grant/revoke permissions and manage sub-entity admins    | `CredentialsManage`, `SubunitManage`, `Introspection`.         |
| **Operator (accounting/factoring)** | Person issuing or reviewing invoices                                    | `InvoiceWrite`, `InvoiceRead`.                                 |
| **Authorized entity**               | Entity permitted to issue invoices on behalf of another (e.g., tax rep) | `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`.           |
| **Intermediary entity**             | Entity with delegation rights (`canDelegate`)                           | `InvoiceRead`, `InvoiceWrite` with delegation true.            |
| **EU entity admin**                 | Cert-based admin for EU context                                         | `InvoiceWrite`, `InvoiceRead`, `VatUeManage`, `Introspection`. |
| **EU entity representative**        | Cert-based user acting for an EU entity                                 | `InvoiceWrite`, `InvoiceRead`.                                 |
| **Subunit administrator**           | Manager of sub-entities                                                 | `CredentialsManage`.                                           |

---

### Classification of Permissions:

| Type of Permission  | Examples                                            | Indirect Grant Allowed  | Operational Description                                 |
| ------------------- | --------------------------------------------------- | ----------------------- | ------------------------------------------------------- |
| **Invoice-related** | `InvoiceWrite`, `InvoiceRead`                       | (if `canDelegate=true`) | Invoice sending and retrieval operations.               |
| **Administrative**  | `CredentialsManage`, `SubunitManage`, `VatUeManage` |                         | Managing permissions and subunit structures.            |
| **Entity-level**    | `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative` |                         | Authorizing other entities to invoice in entity’s name. |
| **Technical**       | `Introspection`                                     |                         | Access to history of operations and sessions.           |

---

## General vs. Selective Permissions

KSeF supports assigning **general** or **selective** permissions, enabling flexible access control across business partners.

### Selective (Individual) Permissions

Granting permissions in the context of a **specific target entity**. Common for an accounting office servicing a single client—allowing access only in that client’s context.

* Requires `targetIdentifier` (e.g., client NIP).
* Recipient operates only within the specified context.
* No access to other clients of the intermediary.

### General Permissions

Granting without specifying a partner—allowing access for **all entities serviced by an intermediary**.

* `targetIdentifier` is omitted.
* Access spans all clients of the intermediary.
* Useful for mass integrations or shared service centers.
* Use with caution in production due to broad scope.

---

### Technical Notes & Limitations

* Indirect delegation applies only to `InvoiceRead` and `InvoiceWrite`.
* The distinction between selective and general is the presence or absence of `targetIdentifier` in the request payload for `POST /permissions/indirect/grants`.
* Cannot combine general and selective grants in one request—requires separate operations.
* General permissions should be used carefully, especially in production systems.

---

### Permission Granting Flow

1. **Direct Grant**: Admin grants `InvoiceWrite` to a user in the entity context.
2. **Grant with Delegation**: Admin grants `InvoiceRead` with `canDelegate=true` to intermediary.
3. **Indirect Grant**: Intermediary, using `POST /permissions/indirect/grants`, grants permissions to another party on behalf of the original entity.

---

### Example Permissions Matrix:

| User / Entity                 | InvoiceWrite  | InvoiceRead   | CredentialsManage | SubunitManage | TaxRepresentative |
| ----------------------------- | ------------- | ------------- | ----------------- | ------------- | ----------------- |
| Anna Kowalska (PESEL)         | ✅             | ✅             | ❌                 | ❌             | ❌                 |
| Accounting Office XYZ (NIP)   | ✅ (delegated) | ✅ (delegated) | ❌                 | ❌             | ❌                 |
| Jan Nowak (cert-based)        | ✅             | ✅             | ❌                 | ❌             | ❌                 |
| Accounting Dept Admin (PESEL) | ❌             | ❌             | ✅                 | ✅             | ❌                 |
| Parent Company (NIP)          | ✅             | ✅             | ✅                 | ✅             | ✅                 |
| VAT Group Admin (PESEL)       | ❌             | ❌             | ❌                 | ✅             | ❌                 |
| Tax Representative (NIP)      | ❌             | ❌             | ❌                 | ❌             | ✅                 |

---

## Required Roles or Permissions for Granting

| Granting Context                      | Required Role or Permission    |
| ------------------------------------- | ------------------------------ |
| To a person for KSeF access           | `Owner` or `CredentialsManage` |
| To an entity for invoice operations   | `Owner` or `CredentialsManage` |
| Entity-level permissions              | `Owner` or `CredentialsManage` |
| Indirect grant for invoice operations | `Owner` or `CredentialsManage` |
| Subunit administrator                 | `SubunitManage`                |
| EU entity administrator               | `Owner` or `CredentialsManage` |
| EU entity representative              | `VatUeManage`                  |

---

## Identifier Limitations (`subjectIdentifier`, `contextIdentifier`)

| Identifier Type | Identifies                    | Notes                                                        |
| --------------- | ----------------------------- | ------------------------------------------------------------ |
| `Nip`           | Domestic entity or individual | For entities registered in Poland or individuals.            |
| `Pesel`         | Individual                    | Required for staff using trusted profile or cert with PESEL. |
| `Fingerprint`   | Certificate holder            | Used when cert lacks NIP/PESEL or for foreign entity admins. |
| `NipVatUe`      | Linked EU entity              | For granting to EU admins or reps.                           |
| `InternalId`    | Subunit within structure      | Used in complex hierarchical entities.                       |

---

## API Functional Constraints

* Duplicate permissions cannot be granted—API may return an error or ignore duplicates.
* Granting is **asynchronous**, not immediate—must check operation status for completion.

---

## Time Constraints

* Permissions remain active until revoked.
* Time-limited functionality requires client-side logic (e.g., scheduling revocation).

---

## Granting Permissions

### Granting to Individuals for KSeF Access

Permissions can be assigned to individuals (e.g., accountants or IT staff) based on identifiers (PESEL, NIP, Fingerprint). Permissions may include operational and administrative roles. Below is how to grant via the API and required permissions:

**POST** [/permissions/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1persons~1grants/post)

| Field               | Value                                                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `subjectIdentifier` | `"Nip"`, `"Pesel"`, or `"Fingerprint"`                                                                                                         |
| `permissions`       | `"CredentialsManage"`, `"CredentialsRead"`, `"InvoiceWrite"`, `"InvoiceRead"`, `"Introspection"`, `"SubunitManage"`, `"EnforcementOperations"` |
| `description`       | Text description                                                                                                                               |

**Permissible permissions for individuals:**

* `CredentialsManage` – manage permissions
* `CredentialsRead` – view permissions
* `InvoiceWrite` – issue invoices
* `InvoiceRead` – view invoices
* `Introspection` – view session history
* `SubunitManage` – manage subunits
* `EnforcementOperations` – execute enforcement operations

**C# Example:**

```csharp
var request = GrantPersonPermissionsRequestBuilder
    .Create()
    .WithSubject(subjectIdentifier)
    .WithPermissions(StandardPermissionType.InvoiceRead, StandardPermissionType.InvoiceWrite)
    .WithDescription("Access for quarterly review")
    .Build();

return await ksefClient.GrantsPermissionPersonAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
var request = new PersonPermissionsGrantRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withPermissions(List.of(PersonPermissionType.INVOICEREAD, PersonPermissionType.INVOICEWRITE))
    .withDescription("Access for quarterly review")
    .build();

return ksefClient.grantsPermissionPerson(request);
```

Grantors must be:

* Owner
* Holder of `CredentialsManage`
* Subunit administrator (`SubunitManage`)
* EU entity administrator (`VatUeManage`)

---

### Granting Invoice Permissions to Entities

Entities (e.g., accounting offices, shared services) can be granted `InvoiceRead` and `InvoiceWrite`, optionally with `canDelegate=true`. Granting requires `CredentialsManage` or Owner role.

**POST** [/permissions/entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1entities~1grants/post)

| Field               | Value                             |
| ------------------- | --------------------------------- |
| `subjectIdentifier` | Entity identifier (e.g., `"Nip"`) |
| `permissions`       | `"InvoiceWrite"`, `"InvoiceRead"` |
| `description`       | Description text                  |

**C# Example:**

```csharp
var request = GrantEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subjectIdentifier)
    .WithPermissions(
        Permission.New(StandardPermissionType.InvoiceRead, true),
        Permission.New(StandardPermissionType.InvoiceWrite, false)
    )
    .WithDescription("Access for quarterly review")
    .Build();

return await ksefClient.GrantsPermissionEntityAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
var request = new EntityPermissionsGrantRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withPermissions(List.of(
        new EntityPermission(EntityPermissionType.INVOICEREAD, true),
        new EntityPermission(EntityPermissionType.INVOICEREAD, false)
    ))
    .withDescription("Access for quarterly review")
    .build();

return ksefClient.grantsPermissionEntity(request);
```

---

### Granting Entity-Level Permissions

Entity-level permissions (`TaxRepresentative`, `SelfInvoicing`, `RRInvoicing`) are granted only by Owner or `CredentialsManage`.

**POST** [/permissions/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants/post)

| Field               | Value                                                     |
| ------------------- | --------------------------------------------------------- |
| `subjectIdentifier` | `"Nip"`                                                   |
| `permissions`       | `"SelfInvoicing"`, `"RRInvoicing"`, `"TaxRepresentative"` |
| `description`       | Description text                                          |

**C# Example:**

```csharp
var request = GrantProxyEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(subjectIdentifier)
    .WithPermission(StandardPermissionType.TaxRepresentative)
    .WithDescription("Access for quarterly review")
    .Build();

return await ksefClient.GrantsPermissionProxyEntityAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
GrantPermissionsProxyEntityRequest request = new GrantPermissionsProxyEntityRequest();
request.setSubjectIdentifier(subjectIdentifier);
request.setPermissions(List.of(ProxyEntityPermissionType.TAXREPRESENTATIVE));
request.setDescription("Access for quarterly review");

return ksefClient.grantsPermissionsProxyEntity(request);
```

---

### Granting Permissions Indirectly

Intermediaries can grant `InvoiceRead` and `InvoiceWrite` (selective or general) via delegation.

**POST** [/permissions/indirect/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1indirect~1grants/post)

| Field               | Value                                  |
| ------------------- | -------------------------------------- |
| `subjectIdentifier` | `"Nip"`, `"Pesel"`, or `"Fingerprint"` |
| `targetIdentifier`  | `"Nip"` or `null`                      |
| `permissions`       | `"InvoiceRead"`, `"InvoiceWrite"`      |
| `description`       | Description text                       |

**C# Example:**

```csharp
var request = GrantIndirectEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(grantPermissionsRequest.SubjectIdentifier)
    .WithContext(grantPermissionsRequest.TargetIdentifier)
    .WithPermissions(grantPermissionsRequest.Permissions.ToArray())
    .WithDescription(grantPermissionsRequest.Description)
    .Build();

return await ksefClient.GrantsPermissionIndirectEntityAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
IndirectPermissionsGrantRequest request = new IndirectPermissionsGrantRequest();
request.setSubjectIdentifier(subjectIdentifier);
request.setPermissions(List.of(IndirectPermissionType.INVOICEREAD, IndirectPermissionType.INVOICEWRITE));
request.setDescription("Access for quarterly review");

return ksefClient.grantsPermissionIndirectEntity(request);
```

---

### Granting Subunit Administrator Permissions

Manage permissions in hierarchical structures (subunits).

**POST** [/permissions/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1subunits~1grants/post)

**C# Example:**

```csharp
var request = GrantSubUnitPermissionsRequestBuilder
    .Create()
    .WithSubject(grantPermissionsRequest.SubjectIdentifier)
    .WithContext(grantPermissionsRequest.ContextIdentifier)
    .WithDescription(grantPermissionsRequest.Description)
    .Build();

return await ksefClient.GrantsPermissionSubUnitAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
var request = new SubunitPermissionsGrantRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withContextIdentifier(subjectIdentifier)
    .withDescription("Sub-entity access")
    .build();

return ksefClient.grantsPermissionSubUnit(request);
```

---

### Granting EU Entity Administrator Permissions

Assign administrator rights in EU-linked contexts (`nip-vat_ue`), enabling login and permission management for EU representatives.

**POST** [/permissions/eu-entities/administration/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1administration~1grants/post)

**C# Example:**

```csharp
var request = GrantEUEntityPermissionsRequestBuilder
    .Create()
    .WithSubject(grantPermissionsRequest.SubjectIdentifier)
    .WithContext(grantPermissionsRequest.ContextIdentifier)
    .WithDescription("Access for quarterly review")
    .Build();

return await ksefClient.GrantsPermissionEUEntityAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
var request = new EuEntityPermissionsGrantRequestBuilder()
    .withSubject(body.subjectIdentifier())
    .withContext(body.contextIdentifier())
    .withDescription("Access for quarterly review")
    .build();

return ksefClient.grantsPermissionEUEntity(request);
```

---

### Granting EU Entity Representative Permissions

Representatives acting for EU entities can be granted access only by a VAT UE administrator.

**POST** [/permissions/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Nadawanie-uprawnien/paths/~1api~1v2~1permissions~1eu-entities~1grants/post)

**C# Example:**

```csharp
var request = GrantEUEntityRepresentativePermissionsRequestBuilder
    .Create()
    .WithSubject(subjectIdentifier)
    .WithPermissions(StandardPermissionType.InvoiceRead, StandardPermissionType.InvoiceWrite)
    .WithDescription("Representative access")
    .Build();

return await ksefClient.GrantsPermissionEUEntityRepresentativeAsync(request, accessToken, cancellationToken);
```

**Java Example:**

```java
var request = new EuEntityPermissionsGrantRepresentativeRequestBuilder()
    .withSubjectIdentifier(subjectIdentifier)
    .withPermissions(List.of(EuEntityPermissionType.INVOICEREAD, EuEntityPermissionType.INVOICEWRITE))
    .withDescription("Representative access")
    .build();

return ksefClient.grantsPermissionEUEntityRepresentative(request);
```

---

### Revoking Permissions

Revoking permissions is essential for access control, and can be done for individuals, entities, subunits, EU representatives, or administrators.

#### Standard Revocation

**DELETE** [/permissions/common/grants/{permissionId}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1common~1grants~1%7BpermissionId%7D/delete)

**C# Example:**

```csharp
await ksefClient.RevokeCommonPermissionAsync(permissionId, accessToken, cancellationToken);
```

**Java Example:**

```java
ksefClient.revokeCommonPermission(permissionId);
```

Used for revoking permissions granted to individuals, entities, intermediaries, subunit admins, EU administrators, or representatives.

---

#### Revoking Entity-Level Permissions

**DELETE** [/permissions/authorizations/grants/{permissionId}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Odbieranie-uprawnien/paths/~1api~1v2~1permissions~1authorizations~1grants~1%7BpermissionId%7D/delete)

**C# Example:**

```csharp
await ksefClient.RevokeAuthorizationsPermissionAsync(permissionId, accessToken, cancellationToken);
```

**Java Example:**

```java
ksefClient.revokeAuthorizationsPermission(permissionId);
```

Used for `SelfInvoicing`, `RRInvoicing`, `TaxRepresentative`.

---

## Searching Granted Permissions

KSeF provides endpoints to query granted permissions for auditing, interface administration, and access review.

### List of Permissions Granted to Individuals or Entities

**POST** [/permissions/query/persons/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1persons~1grants/post)

| Field                  | Description                                                        |
| ---------------------- | ------------------------------------------------------------------ |
| `authorIdentifier`     | Granting entity identifier: `Nip`, `Pesel`, `Fingerprint`          |
| `authorizedIdentifier` | Recipient identifier: `Nip`, `Pesel`, `Fingerprint`                |
| `targetIdentifier`     | Target entity identifier for indirect grants: `Nip`                |
| `permissionTypes`      | Filter by permissions like `CredentialManage`, `InvoiceRead`, etc. |
| `permissionState`      | `Active` / `Inactive`                                              |

**C# Example:**

```csharp
await ksefClient.SearchGrantedPersonPermissionsAsync(request, accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchGrantedPersonPermissions(request, pageOffset, pageSize);
```

---

### Search Subunit Administrators

**POST** [/permissions/query/subunits/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subunits~1grants/post)

| Field               | Description                               |
| ------------------- | ----------------------------------------- |
| `subjectIdentifier` | Subunit identifier: `InternalId` or `Nip` |

**C# Example:**

```csharp
await ksefClient.SearchSubunitAdminPermissionsAsync(request, accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchSubunitAdminPermissions(request, pageOffset, pageSize);
```

---

### Retrieve Entity Roles

**GET** [/permissions/query/entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1entities~1roles/get)

**C# Example:**

```csharp
await ksefClient.SearchEntityInvoiceRolesAsync(accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchEntityInvoiceRoles(pageOffset, pageSize);
```

---

### Retrieve Subordinate Entities

**POST** [/permissions/query/subordinate-entities/roles](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1subordinate-entities~1roles/post)

| Field                         | Description                                      |
| ----------------------------- | ------------------------------------------------ |
| `subordinateEntityIdentifier` | Identifier of the entity with permissions: `Nip` |

**C# Example:**

```csharp
await ksefClient.SearchSubordinateEntityInvoiceRolesAsync(request, accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchSubordinateEntityInvoiceRoles(request, pageOffset, pageSize);
```

---

### Retrieve Entity-Level Invoice Permissions

**POST** [/permissions/query/authorizations/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1authorizations~1grants/post)

| Field                   | Description                            |
| ----------------------- | -------------------------------------- |
| `authorizingIdentifier` | Granting entity’s identifier: `Nip`    |
| `authorizedIdentifier`  | Recipient entity’s identifier: `Nip`   |
| `queryType`             | `Granted` or `Received`                |
| `permissionTypes`       | Permissions like `SelfInvoicing`, etc. |

**C# Example:**

```csharp
await ksefClient.SearchEntityAuthorizationGrantsAsync(request, accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchEntityAuthorizationGrants(request, pageOffset, pageSize);
```

---

### Retrieve EU Entity Permissions

**POST** [/permissions/query/eu-entities/grants](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Wyszukiwanie-nadanych-uprawnien/paths/~1api~1v2~1permissions~1query~1eu-entities~1grants/post)

| Field                             | Description                                         |
| --------------------------------- | --------------------------------------------------- |
| `vatUeIdentifier`                 | EU VAT identifier                                   |
| `authorizedFingerprintIdentifier` | Certificate fingerprint of authorized party         |
| `permissionTypes`                 | `VatUeManage`, `InvoiceWrite/Read`, `Introspection` |

**C# Example:**

```csharp
await ksefClient.SearchGrantedEuEntityPermissionsAsync(request, accessToken, pageOffset, pageSize);
```

**Java Example:**

```java
ksefClient.searchGrantedEuEntityPermissions(request, pageOffset, pageSize);
```

---

## Operations

KSeF allows tracking the status of asynchronous grant and revoke operations using a unique `operationReferenceNumber`. This enables automation, error handling, and admin reporting.

### Retrieve Operation Status

**GET** [/permissions/operations/{operationReferenceNumber}](https://ksef-test.mf.gov.pl/docs/v2/index.html#tag/Operacje/paths/~1api~1v2~1permissions~1operations~1%7BoperationReferenceNumber%7D/get)

**C# Example:**

```csharp
var operationStatus = await ksefClient.OperationsStatusAsync(referenceNumber, accessToken, cancellationToken);
```

**Java Example:**

```java
var permissionStatusInfo = ksefClient.permissionOperationStatus(referenceNumber);
```
