
# BUSINESS RULES — VulnTrack

**Status:** Draft  
**Version:** 0.2  
**Scope:** MVP  
**Last updated:** 2026-10-26

## 1. Purpose

This document defines the business rules that govern VulnTrack's MVP. These rules are the source of truth for use cases, domain modeling, validation, authorization, and tests. Technical implementation details may evolve, but business behavior must remain consistent with these rules.

## 2. Core Principles

1. **Organization isolation:** organization-owned data is accessible only to authorized members of that organization.
2. **No scanner in the MVP:** assets, vulnerabilities, and findings are entered manually or imported through the defined JSON import process. Scanning is a possible future, separate project.
3. **Finding-centered workflow:** a Finding represents a specific Vulnerability affecting a specific Asset.
4. **Preserve history:** deactivation, remediation, closure, and reopening must not erase historical records.
5. **Backend enforcement:** authorization, lifecycle rules, relationship constraints, and validation must be enforced by the backend, not only by the UI.
6. **Controlled scope:** the MVP uses fixed lifecycle states, severity/priority levels, asset types, and criticality levels.
7. **No cross-organization relationships:** relationships between organization-owned records must remain within the same organization.
8. **Auditable changes:** relevant security and administrative actions are recorded by the backend.

## 3. Organization Rules

### BR-ORG-001 — Organization ownership

Every organization-owned record MUST belong to exactly one organization.

### BR-ORG-002 — Organization isolation

A user MUST NOT access, modify, or infer private records belonging to another organization unless a specific platform-level operation explicitly permits the action.

### BR-ORG-003 — Organization context

Every organization-scoped operation MUST execute within a validated organization context and MUST verify the user's membership and permissions.

### BR-ORG-004 — Same-organization relationships

A relationship between organization-owned records MUST connect records belonging to the same organization. The backend MUST reject cross-organization references.

### BR-ORG-005 — Organization status

An organization may be active or suspended. Suspended organizations MUST NOT permit ordinary organization-user operations.

### BR-ORG-006 — Suspended organization data

Suspending an organization MUST NOT delete its data or alter its historical records.

### BR-ORG-007 — Organization administration

An organization MUST have at least one active user with the `ADMIN` role during normal operation.

### BR-ORG-008 — Organization creation

Organizations are created by a `PLATFORM_ADMIN`. The platform administrator designates the initial organization administrator as part of setup.

### BR-ORG-009 — Deletion, retention, and recovery

Organization deletion, data-retention periods, export, backup, and recovery policies are not implemented in the MVP. These policies MUST be defined before permanent organization deletion is introduced.

### BR-ORG-010 — No implicit platform access

Platform-level administration MUST NOT automatically grant access to an organization's private assets, vulnerabilities, findings, or other security details.

## 4. Platform Administration Rules

### BR-PLATFORM-001 — Platform administrator role

`PLATFORM_ADMIN` is a platform-level role, distinct from organization roles.

### BR-PLATFORM-002 — Organization lifecycle management

A `PLATFORM_ADMIN` MAY create, suspend, and reactivate organizations.

### BR-PLATFORM-003 — Initial organization administrator

When creating an organization, a `PLATFORM_ADMIN` MUST designate an initial `ADMIN` for that organization.

### BR-PLATFORM-004 — Separation of administrative scope

Platform administration does not imply organization-level authorization. Access to private organization data requires a separately defined, explicitly authorized operation; no such general access is included in the MVP.

## 5. Authentication and Authorization Rules

### BR-AUTH-001 — Authenticated access

Protected operations MUST require an authenticated user.

### BR-AUTH-002 — Authorization on the backend

The backend MUST verify authorization for every protected operation, regardless of frontend behavior.

### BR-AUTH-003 — Organization membership

A user MUST have an active membership in the organization to perform organization-scoped operations there.

### BR-AUTH-004 — MVP organization membership

For the MVP, each user account is associated with one organization. The design SHOULD avoid assumptions that would make future multi-organization membership impossible.

### BR-AUTH-005 — Organization roles

Organization roles are `ADMIN`, `SECURITY_MANAGER`, and `USER`.

### BR-AUTH-006 — One active organization role

A user has one active organization role per organization membership in the MVP.

### BR-AUTH-007 — Default role

A newly created organization user MUST receive the `USER` role unless an authorized administrator explicitly assigns a different permitted role.

### BR-AUTH-008 — User management

Only an organization `ADMIN` may create, modify, deactivate, or reactivate users within that organization, subject to the rule protecting the last active administrator.

### BR-AUTH-009 — Role assignment

Only an organization `ADMIN` may assign or change organization roles.

### BR-AUTH-010 — Maximum active administrators

An organization MUST NOT have more than three active users with the `ADMIN` role.

### BR-AUTH-011 — Protect the last administrator

The system MUST prevent an operation that would leave an organization with no active `ADMIN`.

### BR-AUTH-012 — Deactivated users

A deactivated user MUST NOT authenticate or perform organization operations. Historical records created or changed by that user MUST be preserved.

### BR-AUTH-013 — Assignment eligibility

A Finding's assigned individual MUST be an active user in the same organization as the Finding.

### BR-AUTH-014 — Role changes

A role change MUST take effect for subsequent authorization checks. Session and token invalidation details remain an implementation decision.

### BR-AUTH-015 — No privilege escalation

Users MUST NOT grant themselves roles or permissions they do not already have authority to assign.

### BR-AUTH-016 — Data minimization

Authentication secrets, passwords, access tokens, and refresh tokens MUST NOT be written to audit logs.

### BR-AUTH-017 — Explicit authorization

If a permission is not granted by the role-permission rules or another explicit rule, it MUST be denied by default.

## 6. Preliminary Role-Permission Matrix

This matrix describes intended MVP permissions. More specific business rules in this document take precedence where they narrow a permission.

| Operation | PLATFORM_ADMIN | ADMIN | SECURITY_MANAGER | USER |
|---|---|---|---|---|
| Create/suspend/reactivate organizations | Yes | No | No | No |
| Access an organization's private security data by platform role alone | No | No | No | No |
| Manage organization users and roles | No | Yes | No | No |
| Create teams | No | Yes | Pending scope decision | No |
| Modify/deactivate teams | No | Yes | Pending scope decision | No |
| Create assets | No | Yes | No | No |
| Modify existing asset fields and lifecycle, excluding responsible-team assignment | No | Yes | Yes | No |
| Assign/change an asset's responsible team | No | Yes | No | No |
| Create/manage vulnerabilities | No | Yes | Yes | No |
| Create findings | No | Yes | Yes | Yes |
| Triage findings and change severity/priority | No | Yes | Yes | No |
| Assign a finding's team and individual | No | Yes | Yes | No |
| Submit a remediation for verification | No | Assigned individual | Assigned individual | Assigned individual, if assigned |
| Verify remediation and mark REMEDIATED | No | Yes | Yes | No |
| Close a remediated finding | No | Yes | No | No |
| Reopen a closed finding | No | Yes | Yes | No |
| View audit events | No automatic access | Organization-wide | Relevant security events | No unrestricted access |
| Formally accept risk | No | No | No | No |

**Clarification:** the `SECURITY_MANAGER` may modify existing asset fields and change an Asset's active/inactive status, but may not assign or change its responsible Team. Asset creation is restricted to `ADMIN` so the required initial Team assignment can be performed by the authorized role. The exact scope of `SECURITY_MANAGER` permissions for Team management remains an open decision.

## 7. Asset Rules

### BR-AST-001 — Asset definition

An Asset represents an application, service, infrastructure component, repository, or other in-scope resource controlled by an organization.

### BR-AST-002 — Asset ownership

Every Asset MUST belong to exactly one organization.

### BR-AST-003 — Responsible Team

Every Asset MUST have exactly one responsible Team from the same organization. An Asset MUST NOT be considered operationally configured until this assignment exists.

### BR-AST-004 — Fixed asset types

The MVP supports these asset types only: `APPLICATION_API`, `SERVER_INFRASTRUCTURE`, `DATABASE`, `REPOSITORY`, `CLOUD_RESOURCE`, and `CONTAINER`. Organizations MUST NOT define custom asset types in the MVP.

### BR-AST-005 — Asset lifecycle

An authorized user may change an Asset's lifecycle status subject to role permissions and lifecycle restrictions. The MVP supports `ACTIVE` and `INACTIVE` only.

### BR-AST-006 — Team membership and assignment integrity

A Team MUST have at least one user. An active Team MUST be responsible for at least one Asset. Each Asset MUST have exactly one responsible Team. During initial setup, the organization administrator MUST create the necessary Team membership and Asset assignment before the organization is treated as operationally configured.

### BR-AST-007 — Duplicate asset prevention

The system MUST prevent duplicate Asset records when a reliable matching criterion is available. The exact matching criteria for each asset type remain an open decision. The system MUST NOT silently merge records solely because their display names match unless the defined matching rules establish that they are duplicates.

### BR-AST-008 — Asset criticality

Each Asset MUST have exactly one criticality level: `LOW`, `MEDIUM`, or `HIGH`. These levels are fixed globally for the MVP. Organizations may assign a level to each Asset but MUST NOT define custom levels.

### BR-AST-009 — Asset history

The system MUST preserve an Asset's historical information and its relationships to existing Findings when the Asset becomes inactive.

### BR-AST-010 — Asset lifecycle and Findings

New Findings MUST NOT be created for or associated with Assets whose status is `INACTIVE`.

This restriction MUST be enforced by the backend for both manual operations and JSON imports.

Existing Findings and their references to Vulnerabilities MUST be preserved and remain available for authorized historical consultation. Changing an Asset's status MUST NOT automatically delete, close, or mark existing Findings as remediated.

Reactivating an inactive Asset MAY allow new Findings to be created for it, subject to the application's normal authorization and validation rules, including the lifecycle status of the referenced Vulnerability.

### BR-AST-011 — Asset deletion

Assets with historical Findings MUST NOT be physically deleted through ordinary application operations. The MVP MUST preserve the history needed to understand past Findings.

### BR-AST-012 — Team deactivation

A Team MUST NOT be deactivated while it is responsible for one or more Assets.

Before deactivating a Team, an organization `ADMIN` MUST reassign all its Assets to another active Team within the same organization.

The system MUST prevent Team deactivation if the Team is still responsible for any Assets.

Team deactivation MUST NOT delete the Team's historical information or alter historical records associated with its previous Asset assignments.

## 8. Vulnerability Rules

### BR-VUL-001 — Vulnerability ownership

Every Vulnerability record MUST belong to exactly one organization.

### BR-VUL-002 — Vulnerability definition

A Vulnerability represents an identifiable security weakness or issue that may affect one or more Assets. It is distinct from a Finding, which records that the Vulnerability affects a particular Asset.

### BR-VUL-003 — Vulnerability lifecycle

A Vulnerability may be `ACTIVE` or `INACTIVE`. An inactive Vulnerability MUST NOT be associated with newly created Findings. Existing Findings referencing it MUST be preserved.

### BR-VUL-004 — No global catalog in MVP

The MVP MUST NOT expose a shared global Vulnerability catalog across organizations. Organization-owned Vulnerabilities and their relationships MUST remain isolated.

## 9. Finding Rules

### BR-FND-001 — Finding definition

A Finding represents one specific Vulnerability affecting one specific Asset within one organization.

### BR-FND-002 — Finding relationships

Every Finding MUST reference exactly one Asset and exactly one Vulnerability. The Asset, Vulnerability, and Finding MUST belong to the same organization.

### BR-FND-003 — Initial status

A newly created Finding MUST start in either `CREATED` or `OPEN`. A new Finding MUST NOT be created directly as `REMEDIATED` or `CLOSED`.

### BR-FND-004 — Independent Findings

If the same Vulnerability affects multiple Assets, the system MUST represent each affected Asset with a separate Finding. Each Finding has an independent status, assignment, severity, priority, and history.

### BR-FND-005 — Finding creation permissions

An active organization user with the `ADMIN`, `SECURITY_MANAGER`, or `USER` role MAY create a Finding, subject to all applicable validation rules.

### BR-FND-006 — Severity

Each Finding MUST have exactly one severity: `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL`. The levels are fixed for the MVP.

### BR-FND-007 — Priority

Each Finding MUST have exactly one priority: `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL`. Priority is distinct from severity and may reflect business context, including Asset criticality and exposure.

### BR-FND-008 — Required assignment before execution

A Finding MUST have exactly one responsible Team and exactly one assigned active individual from the same organization before it can enter `IN_PROGRESS`. The same assignment requirements apply before a Finding can enter `REMEDIATED` or `CLOSED`.

### BR-FND-009 — Finding status definitions

- `CREATED`: the Finding has been recorded but has not yet entered the active triage workflow.
- `OPEN`: the Finding is available for assessment or action.
- `TRIAGED`: the Finding has been assessed and is ready for planned handling.
- `IN_PROGRESS`: remediation work is underway.
- `REMEDIATED`: remediation has been completed and verified by an authorized `SECURITY_MANAGER` or `ADMIN`.
- `CLOSED`: the verified Finding has been formally closed by an authorized `ADMIN`.

### BR-FND-010 — Permitted status transitions

The following transitions are permitted in the MVP:

| Current status | Permitted next status | Who may perform the transition |
|---|---|---|
| `CREATED` | `OPEN` | `ADMIN`, `SECURITY_MANAGER` |
| `CREATED` | `TRIAGED` | `ADMIN`, `SECURITY_MANAGER` |
| `OPEN` | `TRIAGED` | `ADMIN`, `SECURITY_MANAGER` |
| `TRIAGED` | `OPEN` | `ADMIN`, `SECURITY_MANAGER` |
| `TRIAGED` | `IN_PROGRESS` | `ADMIN`, `SECURITY_MANAGER`, subject to assignment requirements |
| `IN_PROGRESS` | `REMEDIATED` | `ADMIN` or `SECURITY_MANAGER`, after verification |
| `REMEDIATED` | `CLOSED` | `ADMIN` only |
| `REMEDIATED` | `IN_PROGRESS` | `ADMIN` or `SECURITY_MANAGER`, when remediation is invalidated or more work is required |
| `CLOSED` | `OPEN` | `ADMIN` or `SECURITY_MANAGER` |

Any transition not listed above MUST be rejected. The backend MUST validate both the current status and the actor's permission for every transition.

### BR-FND-011 — Remediation submission

The assigned individual MAY submit remediation evidence or request verification. This action MUST NOT by itself change the Finding status to `REMEDIATED`.

### BR-FND-012 — Remediation verification

Only an `ADMIN` or `SECURITY_MANAGER` may verify remediation and transition a Finding from `IN_PROGRESS` to `REMEDIATED`. The person submitting remediation MUST NOT approve their own submission when acting only as the assigned individual; verification requires an authorized reviewer action.

### BR-FND-013 — Inactive Asset restrictions

New Findings MUST NOT be created for inactive Assets. Existing Findings MUST remain accessible to authorized users and MUST NOT be automatically deleted, closed, reopened, or marked remediated when an Asset's status changes. Reactivating an Asset allows new Findings only if all normal rules are satisfied.

### BR-FND-014 — Inactive Vulnerability restrictions

New Findings MUST NOT reference an inactive Vulnerability. Existing Findings referencing an inactive Vulnerability MUST remain available for historical consultation.

### BR-FND-015 — Finding history

Status changes, assignment changes, severity/priority changes, remediation submissions, verification, closure, and reopening MUST preserve relevant history and generate the applicable audit events.

## 10. Lifecycle Rules

### BR-LIF-001 — Controlled lifecycle

Lifecycle statuses MUST be represented by the defined values and transitions. The frontend MUST NOT be the only layer enforcing these rules.

### BR-LIF-002 — No arbitrary transitions

The backend MUST reject status transitions that are not explicitly permitted by the relevant entity's lifecycle rules.

### BR-LIF-003 — Historical preservation

The system MUST preserve relevant historical records and relationships when an entity becomes inactive, is remediated, is closed, or is reopened.

### BR-LIF-004 — Asset lifecycle states

Asset lifecycle states in the MVP are `ACTIVE` and `INACTIVE`. The system MUST NOT introduce a `RETIRED` Asset state in the MVP.

### BR-LIF-005 — Vulnerability lifecycle states

Vulnerability lifecycle states in the MVP are `ACTIVE` and `INACTIVE`.

### BR-LIF-006 — Finding lifecycle states

Finding lifecycle states are `CREATED`, `OPEN`, `TRIAGED`, `IN_PROGRESS`, `REMEDIATED`, and `CLOSED`.

### BR-LIF-007 — Deactivation restrictions

Deactivation MUST respect entity-specific rules. In particular, an inactive Asset cannot receive new Findings; an inactive Vulnerability cannot be referenced by new Findings; and a Team cannot be deactivated while it remains responsible for Assets.

### BR-LIF-008 — Reopening

Reopening a closed Finding MUST use the transition `CLOSED` to `OPEN`, MUST be performed by an `ADMIN` or `SECURITY_MANAGER`, and MUST preserve the previous closure and audit history.

### BR-LIF-009 — No deletion through status changes

Changing an entity's lifecycle status MUST NOT be used as a substitute for deleting its history or relationships. Permanent deletion and retention behavior remain subject to the unresolved policies in this document.

## 11. Risk and Prioritization Rules

### BR-RSK-001 — Severity and priority are distinct

Severity describes the seriousness of the Vulnerability as it affects the Asset. Priority describes the urgency with which the organization should address the Finding.

### BR-RSK-002 — Fixed severity levels

The only severity levels in the MVP are `LOW`, `MEDIUM`, `HIGH`, and `CRITICAL`.

### BR-RSK-003 — Fixed priority levels

The only priority levels in the MVP are `LOW`, `MEDIUM`, `HIGH`, and `CRITICAL`.

### BR-RSK-004 — Risk context

Asset criticality and relevant exposure/business context SHOULD inform the manual assessment of a Finding's priority.

### BR-RSK-005 — Critical priority

A `CRITICAL` priority indicates that the Finding requires immediate attention and urgent response appropriate to the risk. It does not guarantee that remediation can or will be completed immediately.

### BR-RSK-006 — Risk acceptance out of scope

Formal risk acceptance, risk exceptions, and risk-acceptance approval workflows are out of scope for the MVP.

### BR-RSK-007 — No automatic risk score

The MVP MUST NOT calculate or claim to provide an automated risk score. Severity and priority are assigned by authorized users.

### BR-RSK-008 — Permission to change severity and priority

Only an `ADMIN` or `SECURITY_MANAGER` may set or change a Finding's severity and priority.

## 12. JSON Import Rules

### BR-IMP-001 — Supported import format

The MVP supports structured JSON import only. Arbitrary file uploads and unrestricted file formats are out of scope.

### BR-IMP-002 — Schema validation

An import payload MUST conform to the supported JSON schema and required field definitions. The schema versioning and final payload shape remain open decisions.

### BR-IMP-003 — Authorization

Only authenticated users with permission to create the relevant records may execute an import. The import MUST run within a validated organization context.

### BR-IMP-004 — Organization isolation

Every imported record and reference MUST belong to the importing organization. Cross-organization references MUST cause the import to fail.

### BR-IMP-005 — Atomic import

An import MUST be atomic: if any record or relationship fails validation or persistence, the entire import MUST be rejected and none of its records may be partially committed.

### BR-IMP-006 — Business-rule validation

Imports MUST respect all lifecycle, authorization, assignment, and relationship rules. An import MUST be rejected if it attempts to create a Finding for an inactive Asset, associate a new Finding with an inactive Vulnerability, bypass required assignments, or perform a prohibited lifecycle transition.

New Findings created through JSON import MUST be initialized in either the `CREATED` or `OPEN` state.

JSON imports MUST NOT bypass permitted lifecycle transitions or be used to directly mark a newly imported Finding as `REMEDIATED` or `CLOSED`.

### BR-IMP-007 — No silent coercion

Invalid values MUST be rejected rather than silently coerced into a different status, severity, priority, asset type, or criticality.

### BR-IMP-008 — Import error reporting

The system SHOULD report actionable validation errors identifying the relevant record and field where feasible. Error responses MUST NOT disclose private data from another organization.

### BR-IMP-009 — Duplicate handling

Imports MUST apply the same duplicate-prevention rules as manual creation. The system MUST NOT silently create duplicates when a reliable matching rule exists.

### BR-IMP-010 — Audit

Import attempts and their outcomes MUST generate appropriate audit events. Audit events MUST NOT contain passwords, tokens, or other authentication secrets.

## 13. Audit Rules

### BR-AUD-001 — Backend-generated audit events

Audit events MUST be generated by the backend. Clients MUST NOT be trusted to submit authoritative audit records.

### BR-AUD-002 — Relevant actions

The system MUST audit relevant authentication/security events, user and role administration, organization administration, Asset and Team changes, Vulnerability changes, Finding creation and changes, imports, remediation verification, closure, reopening, and access-control failures where appropriate.

### BR-AUD-003 — Organization scope

Organization audit events MUST be associated with the organization to which they belong.

### BR-AUD-004 — Actor attribution

Where an authenticated actor initiates an action, the audit event SHOULD identify that actor. System-generated actions SHOULD identify the system as the actor.

### BR-AUD-005 — Event details

Audit events SHOULD record the event type, timestamp, actor, target entity, and relevant change context, while avoiding unnecessary sensitive data.

### BR-AUD-006 — No secrets in logs

Audit events MUST NOT include passwords, access tokens, refresh tokens, or other authentication secrets.

### BR-AUD-007 — Append-only through the application

Audit records MUST NOT be editable or deletable through ordinary application operations.

### BR-AUD-008 — Audit access

An organization `ADMIN` may consult organization-wide audit events. A `SECURITY_MANAGER` may consult relevant security events. A `USER` MUST NOT have unrestricted access to audit logs. A `PLATFORM_ADMIN` has no automatic access to private organization audit data.

### BR-AUD-009 — Audit retention

Audit records MUST be retained while the organization remains registered. Permanent deletion, long-term retention, export, and recovery policies remain open decisions and MUST be defined before permanent organization deletion is implemented.

### BR-AUD-010 — Audit and business-operation consistency

For critical state changes, the system SHOULD ensure that the business operation and its required audit event persist consistently. The system MUST NOT report a critical operation as successful if its required audit event cannot be persisted.

### BR-AUD-011 — Audit implementation details

The persistence mechanism, transaction boundaries, and recovery behavior for audit records remain implementation decisions. These details MUST preserve the guarantees established by the rules above.

## 14. Data and Architecture Constraints

### ARCH-DATA-001 — Relational integrity

The data model MUST enforce required relationships and referential integrity for organization-owned entities.

### ARCH-DATA-002 — Organization-scoped references

The backend MUST validate organization ownership for references, even when database identifiers are globally unique.

### ARCH-DATA-003 — Finding as the linking entity

A Finding MUST link one Asset and one Vulnerability. A Vulnerability MUST NOT be treated as the same record as a Finding.

### ARCH-DATA-004 — History preservation

The data model MUST support preservation of relevant historical Finding states and audit events.

### ARCH-DATA-005 — MVP architecture

The MVP is intended to use a modular monolith. Microservices, Kafka, Kubernetes, and distributed architecture are not required for the MVP.

### ARCH-DATA-006 — Database

PostgreSQL is the intended relational database for the MVP.

## 15. Open Decisions

The following decisions remain open and MUST be resolved before the affected behavior is treated as final:

1. **Organization deletion and data policies:** define deletion, retention periods, export, backup, and recovery before implementing permanent organization deletion.
2. **Administrator recovery:** define the recovery process if an organization has no active `ADMIN`, without allowing privilege escalation.
3. **Security Manager Team permissions:** define whether `SECURITY_MANAGER` may create, modify, or deactivate Teams. Until decided, those permissions remain pending; `ADMIN` retains full Team administration.
4. **Authentication/session invalidation:** define token/session invalidation after role changes, user deactivation, or organization suspension, and define organization-context resolution if multi-organization membership is introduced.
5. **Audit persistence:** define transaction boundaries and failure handling so critical business changes and audit events remain consistent.
6. **JSON schema and versioning:** define the exact import payload structure, supported schema version, and compatibility behavior.
7. **Duplicate detection:** define reliable matching criteria for each Asset type and other entity types where duplicate prevention is required.
8. **Team membership edge cases:** define the behavior when a Team's last active user is deactivated or removed. Until resolved, the rule that a Team must have at least one user remains in force.

### Decisions established for the MVP

- Asset lifecycle states are `ACTIVE` and `INACTIVE`; `RETIRED` is out of scope.
- Vulnerability lifecycle states are `ACTIVE` and `INACTIVE`.
- Finding lifecycle states and permitted transitions are defined in this document.
- Severity, priority, asset criticality, and asset types use fixed values.
- Formal risk acceptance and automated risk scoring are out of scope.
- The scanner is out of scope; records are entered manually or through validated JSON import.
- JSON imports are atomic and cannot bypass business rules.
- Audit events are generated by the backend and retained while the organization remains registered.
- PostgreSQL is the intended database.
- The MVP uses a modular monolith.
