# VulnTrack — Business Rules

**Document status:** Draft  
**Version:** 0.1  
**Scope:** MVP  
**Last updated:** 2026-10-15

## 1. Purpose

This document defines the business rules that govern VulnTrack.

Business rules establish the constraints, permissions, validations, and conditions that the application must enforce regardless of the user interface or technical implementation.

These rules will guide the definition of use cases, the domain model, database design, API behavior, and automated tests.

## 2. Core Principles

- **Security first:** Access to organizational data must be restricted to authorized users.
- **Organization isolation:** One organization must never access another organization's private data.
- **Explicit permissions:** Every protected operation must be authorized by the backend.
- **Data integrity:** Assets, vulnerabilities, findings, users, and teams must maintain valid relationships.
- **Structured input:** Manually entered and imported data must follow the same applicable business rules.
- **Traceability:** Important security-related changes should be attributable to the user who performed them.
- **Controlled evolution:** New capabilities must not silently violate existing business rules.
- **Persistence independence:** Business logic must not depend directly on a specific database engine.

## 3. Organization Rules — BR-ORG

### BR-ORG-001 — Organizational ownership

Every organization-owned resource must belong to exactly one organization.

Organizational resources include organization memberships, teams, assets, vulnerabilities, findings, comments, and organization-specific settings.

Global resources are outside the MVP scope unless explicitly introduced in a future version.

### BR-ORG-002 — Organization isolation

A user must not access, modify, or delete private resources belonging to an organization to which they do not have authorized access.

This rule must be enforced by the backend, independently of frontend visibility.

### BR-ORG-003 — Organization-scoped operations

Every operation involving organizational resources must execute within an authorized organizational context.

The backend must validate the organization's scope rather than trusting an organization identifier supplied by the client.

### BR-ORG-004 — Cross-organization relationships

Relationships between organization-owned resources must not cross organizational boundaries.

For example, a finding must not reference an asset or vulnerability belonging to another organization.

### BR-ORG-005 — Organization configuration

Each organization may have its own users, teams, assets, vulnerabilities, findings, and organization-specific security-management settings, subject to the features supported by the MVP.

### BR-ORG-006 — Organization creation

Only a Platform Admin may create an organization in the MVP.

### BR-ORG-007 — Initial administrator

Every newly created organization must have an initial Organization Admin designated through the authorized platform administration process.

The initial administrator is responsible for managing the organization's users and roles.

### BR-ORG-008 — Organization administration

An Organization Admin may manage users, roles, teams, and organization settings within their own organization, subject to the permissions defined for the MVP.

Organization administrators must not administer another organization's resources.

### BR-ORG-009 — Organization lifecycle

Only a Platform Admin may suspend or reactivate an organization.

Organization deletion, data retention, and recovery policies remain to be defined before implementing those capabilities.

### BR-ORG-010 — Organizational independence

Each organization must operate independently from a business and data-access perspective.

The application must not rely on sharing private organizational data to provide its core functionality.

## 4. Platform Administration Rules — BR-PLATFORM

### BR-PLATFORM-001 — Platform-level role

PLATFORM_ADMIN is a platform-level role, distinct from the roles assigned within an organization.

### BR-PLATFORM-002 — Organization management

A Platform Admin may create, suspend, and reactivate organizations through authorized platform administration operations.

### BR-PLATFORM-003 — Initial administrator designation

A Platform Admin may designate the initial Organization Admin when creating an organization.

Routine management of the organization's users and roles is delegated to its Organization Admin.

### BR-PLATFORM-004 — Separation of administrative responsibilities

Platform administration does not automatically grant unrestricted access to an organization's private security data.

Any future platform-level access to customer data must be explicitly authorized, controlled, and auditable.

## 5. User and Role Rules — BR-AUTH

### BR-AUTH-001 — Authentication

A user must authenticate before accessing protected application resources.

### BR-AUTH-002 — User identity

A User represents an account that can authenticate with VulnTrack.

A User identity is distinct from the user's organization membership and the role assigned within that organization.

### BR-AUTH-003 — Organizational membership

An organization membership associates a User with an Organization and defines the User's role within that organization.

The membership determines the organizational context in which the user may operate.

### BR-AUTH-004 — Single organization membership in the MVP

In the MVP, each User account may belong to only one organization.

The conceptual model should not unnecessarily prevent supporting multiple organization memberships in a future version.

### BR-AUTH-005 — Platform role separation

PLATFORM_ADMIN is assigned at the platform level and is not an organization-level role.

The organization-level roles available in the MVP are:

- ADMIN
- SECURITY_MANAGER
- USER

### BR-AUTH-006 — Fixed role permissions

Role permissions are predefined in the MVP.

Organization administrators cannot create custom roles or modify the permission definitions.

### BR-AUTH-007 — One active organization role

Each organization membership has exactly one active organization-level role in the MVP.

A user cannot simultaneously hold multiple organization roles within the same membership.

### BR-AUTH-008 — Backend enforcement

The backend must validate permissions for every protected operation.

Hiding a button, page, or navigation item in the frontend does not constitute authorization.

### BR-AUTH-009 — Least privilege

Users must receive only the permissions required for their responsibilities.

Administrative capabilities must not be granted implicitly to ordinary users.

### BR-AUTH-010 — Administrative user management

Only an authorized Organization Admin may create, deactivate, and manage users belonging to their own organization, including assigning or changing their organization roles.

A Platform Admin may designate the initial administrator through the organization creation process.

### BR-AUTH-011 — Administrator limit

An organization may have a maximum of three active ADMIN accounts in the MVP.

Operations that would exceed this limit must be rejected.

### BR-AUTH-012 — Individual administrator accounts

Each administrator must have an individual account.

Sharing administrator credentials is not an intended operating model.

### BR-AUTH-013 — Last administrator protection

The system must not allow ordinary user-management operations to leave an organization without an active ADMIN.

Any exceptional administrator recovery procedure must be defined separately.

### BR-AUTH-014 — Organization-scoped administration

An Organization Admin must not manage users, roles, or settings belonging to another organization.

### BR-AUTH-015 — User deactivation

A deactivated User must not be able to authenticate or continue accessing protected resources merely because a previously issued session or token remains valid.

The implementation must enforce deactivation during authorization.

### BR-AUTH-016 — Role changes

Role changes must be authorized and enforced on subsequent protected operations.

The implementation must account for active sessions and previously issued tokens.

### BR-AUTH-017 — Resource-level authorization

Having a role that permits an operation does not automatically grant access to every resource.

The backend must also validate the user's organizational scope and any applicable resource-level restrictions.

## 6. Preliminary Role-Permission Matrix

The following matrix defines the intended MVP permissions. The details of each operation must be confirmed before its use case is finalized.

| Capability | PLATFORM_ADMIN | ADMIN | SECURITY_MANAGER | USER |
|---|---|---|---|---|
| Create organizations | Yes | No | No | No |
| Suspend/reactivate organizations | Yes | No | No | No |
| Designate initial organization admin | Yes | No | No | No |
| View organizational dashboard | No automatic access | Yes | Yes | Limited |
| Manage organizational users and roles | No | Yes | No | No |
| Manage teams | No | Yes | Yes, subject to defined scope | No |
| Create and manage assets | No | Yes | Yes | No |
| Create and manage vulnerabilities | No | Yes | Yes | No |
| Create findings | No | Yes | Yes | No |
| Assign findings | No | Yes | Yes | No |
| Manage finding priority | No | Yes | Yes | No |
| Work on assigned findings | No | Yes | Yes | Yes |
| Add comments | No | Yes | Yes | Yes, within permitted scope |
| Verify remediation | No | Yes | Yes | No |
| Accept organizational risk | No | Yes | Yes, subject to defined policy | No |
| Import JSON | No | Yes | Yes | No |
| Manage organization settings | No | Yes | No | No |

**Important:** This matrix describes intended MVP permissions. Platform administration and organization-level permissions are separate. The Platform Admin does not receive automatic access to private organizational resources.

All permissions must be enforced by the backend. Resource access, assignment restrictions, and applicable business rules must be validated independently of frontend controls.

## 7. Asset Rules — BR-AST

Status: Pending definition.

This section will define asset identity, ownership, required attributes, classification, lifecycle, duplication, and the handling of assets associated with existing findings.

## 8. Vulnerability Rules — BR-VUL

### BR-VUL-001 — Organizational ownership

Every Vulnerability record created in the MVP belongs to exactly one organization.

### BR-VUL-002 — No shared vulnerability records

Vulnerability records must not be shared between organizations in the MVP, even when they describe the same underlying technical weakness.

### BR-VUL-003 — Independent vulnerability records

Different organizations may independently register the same vulnerability.

Their records, findings, priorities, comments, and remediation information must remain separate.

### BR-VUL-004 — No global vulnerability catalog in the MVP

The MVP will not maintain a shared global vulnerability catalog.

A public reference catalog, such as one based on CVE data, may be considered in a future version without exposing private organizational information.

## 9. Finding Rules — BR-FND

Status: Pending definition.

This section will define finding identity, required attributes, relationships with assets and vulnerabilities, assignment, duplication, and the conditions required to create or modify a finding.

## 10. Finding Lifecycle Rules — BR-LIF

Status: Pending definition.

This section will define valid finding states, permitted transitions, transition authorization, remediation, verification, closure, reopening, and any restrictions on state changes.

## 11. Risk and Priority Rules — BR-RSK

Status: Pending definition.

This section will define the distinction between technical severity and business priority, the factors used to determine urgency, and the rules governing risk acceptance.

## 12. Import Rules — BR-IMP

Status: Pending definition.

This section will define the supported JSON contract, schema versioning, validation, authorization, duplicate handling, reference resolution, and transaction behavior when an import fails.

## 13. Audit and Traceability Rules — BR-AUD

Status: Pending definition.

This section will define which security-sensitive operations require audit records and the information those records must preserve, such as the actor, action, affected resource, and timestamp.

## 14. Architecture Constraints — ARCH-DATA

### ARCH-DATA-001 — Persistence independence

Domain rules and business logic must not depend directly on a specific database engine.

### ARCH-DATA-002 — MVP database

The MVP will use PostgreSQL as its database engine.

### ARCH-DATA-003 — Future database-per-organization support

The architecture should preserve the possibility of deploying different organizations on independent databases in the future.

The MVP is not required to support multiple database engines.

### ARCH-DATA-004 — Database-independent isolation invariant

Whether organizations share a database or use independent databases, organizational data isolation remains mandatory.

### ARCH-DATA-005 — Persistence boundary

Database access must be encapsulated behind persistence abstractions so database-specific concerns do not spread throughout domain and application logic.

### ARCH-DATA-006 — Future database compatibility

Future support for different database engines must preserve the application's domain rules and persistence contract.

Each supported engine must be explicitly tested before being considered compatible.

## 15. Open Decisions

The following decisions remain to be resolved before implementing the affected functionality:

- Whether organization deletion will be supported and what retention, recovery, and data-export policies will apply.
- The complete lifecycle and required attributes of assets.
- The exact finding creation, assignment, duplication, and modification rules.
- The complete finding lifecycle, including reopening and verification requirements.
- The policy for determining business priority and accepting risk.
- The exact JSON import contract, supported operations, duplicate handling, and failure behavior.
- Which operations require audit records and the retention and protection of those records.
- The detailed scope of permissions for each role, including team management and any resource-level restrictions.
- The authentication and organization-resolution architecture required if organizations eventually use independent databases or if users can belong to multiple organizations.

These decisions must be documented before the affected behavior becomes an implementation assumption.