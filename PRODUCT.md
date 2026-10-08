# VulnTrack

## 1. Product

VulnTrack is a vulnerability management platform designed to help organizations identify, prioritize, assign, remediate, and track security vulnerabilities affecting their digital assets.

The platform centralizes assets, vulnerabilities, findings, responsibilities, remediation status, and risk decisions in a single system.

### Product objective

Provide a centralized system where a security or IT team can answer:

- What assets does the organization have?
- Which vulnerabilities affect those assets?
- How severe are those vulnerabilities?
- Which vulnerabilities require immediate attention?
- Who is responsible for remediating them?
- What is the current remediation status?
- Has the vulnerability actually been fixed?
- Which risks have been consciously accepted by the organization?

---

## 2. Problem

Organizations can have hundreds or thousands of digital assets:

- Servers
- Applications
- Databases
- APIs
- Containers
- Repositories
- Network devices
- End-user devices

Security scanners and vulnerability databases can generate large amounts of vulnerability information.

Without a centralized vulnerability management process, organizations may struggle to:

- Keep track of affected assets
- Prioritize vulnerabilities
- Assign responsibility
- Track remediation
- Verify whether vulnerabilities were actually fixed
- Maintain visibility over accepted risks
- Measure the organization's current security posture

VulnTrack aims to solve this problem by turning vulnerability information into actionable security work.

---

## 3. Target Users

### Organization Administrator

Responsible for configuring the organization and managing users and teams.

Main responsibilities:

- Manage users
- Manage teams
- Manage organization settings
- Manage assets
- View security information

### Security Manager

Responsible for vulnerability management and security operations.

Main responsibilities:

- Manage vulnerabilities
- Manage findings
- Prioritize findings
- Assign findings
- Review remediation
- Verify fixes
- Accept risks
- Monitor security posture

### Security / Development User

Responsible for investigating and remediating assigned findings.

Main responsibilities:

- View assigned findings
- Update finding status
- Add comments
- Perform remediation
- Mark findings as remediated

---

## 4. Roles

The initial MVP will implement a simplified role model:

### ADMIN

Can manage:

- Organization
- Users
- Teams
- Assets
- Vulnerabilities
- Findings
- Assignments
- Risk decisions

### USER

Can:

- View relevant assets
- View vulnerabilities
- View assigned findings
- Update assigned findings
- Add comments
- Mark findings as remediated

More granular roles may be introduced in future versions.

---

## 5. Core Concepts

### Organization

The company or entity using VulnTrack.

An organization owns users, teams, assets, and security findings.

### Asset

A concrete digital resource belonging to an organization.

Examples:

- Production server
- Web application
- API
- Database
- Container
- Repository

Example:

```text
Name: production-api-01
Type: SERVER
Environment: PRODUCTION
Criticality: CRITICAL
Owner: Backend Team
```

### Vulnerability

A known security weakness.

Examples:

- CVE
- Dependency vulnerability
- Configuration vulnerability
- Application vulnerability

A vulnerability represents the **security issue itself**, independently of a specific organization asset.

Example:

```text
CVE: CVE-2026-1234
Severity: CRITICAL
CVSS: 9.8
```

### Finding

A specific occurrence of a vulnerability affecting an organization's asset.

Example:

```text
Vulnerability:
CVE-2026-1234

Asset:
production-api-01

Finding:
#152
```

The Finding is the primary unit of work in VulnTrack.

It contains information such as:

- Status
- Priority
- Assignee
- Team
- Due date
- Comments
- Resolution
- Verification

---

## 6. Finding Lifecycle

A finding represents a vulnerability that must be managed through its lifecycle.

```text
DETECTED
    ↓
OPEN
    ↓
TRIAGED
    ↓
IN_PROGRESS
    ↓
REMEDIATED
    ↓
VERIFIED
    ↓
CLOSED
```

An alternative path is:

```text
TRIAGED
    ↓
RISK_ACCEPTED
    ↓
CLOSED
```

### Status definitions

**DETECTED**

The vulnerability has been discovered through an import, scanner, or manual entry.

**OPEN**

The finding requires attention.

**TRIAGED**

The finding has been reviewed and its severity, priority, impact, and ownership have been assessed.

**IN_PROGRESS**

Remediation work is currently being performed.

**REMEDIATED**

The responsible team reports that the vulnerability has been fixed.

**VERIFIED**

The fix has been independently verified, preferably through a new scan or validation process.

**CLOSED**

The finding lifecycle has been completed.

**RISK_ACCEPTED**

The organization consciously accepts the risk instead of remediating the vulnerability.

Risk acceptance must eventually include information such as:

- Reason
- Approver
- Expiration date

---

## 7. Core Business Rules

### Finding vs Vulnerability

A Vulnerability represents a known security issue.

A Finding represents that vulnerability affecting a specific asset.

Therefore:

```text
Vulnerability 1 ──── N Findings
Asset          1 ──── N Findings
```

A single vulnerability can affect many assets.

A single asset can have many findings.

---

### Remediated vs Verified

A finding marked as `REMEDIATED` is not automatically considered fixed.

The responsible user reports that the issue has been fixed.

The finding should only become `VERIFIED` after the fix has been validated.

This distinction is fundamental to the product.

---

### Technical Severity vs Business Priority

Vulnerability severity and business priority are not necessarily the same.

For example:

```text
Vulnerability A
CVSS: 9.8
Asset: Development Server
```

may be less urgent than:

```text
Vulnerability B
CVSS: 7.5
Asset: Internet-facing Production Payment API
```

Therefore VulnTrack will eventually consider additional factors such as:

- Asset criticality
- Internet exposure
- Exploitability
- Business impact
- Technical severity

The first MVP may use a simplified priority model.

---

## 8. Vulnerability Sources

Vulnerabilities may originate from:

### Manual creation

A security user manually creates a vulnerability or finding.

### Import

Vulnerability information can be imported from structured data such as JSON.

### External scanners

Future versions will allow external security scanners to send findings to VulnTrack through an API.

Example:

```text
Security Scanner
       ↓
Scan Results
       ↓
VulnTrack API
       ↓
Finding
```

The scanner itself is outside the scope of the initial VulnTrack MVP.

---

## 9. MVP Scope

The initial MVP will provide the minimum functionality required to manage vulnerabilities from discovery to resolution.

### Authentication

- User registration
- User login
- Authentication
- Authorization
- Logout / token invalidation strategy

### Organization

- Create organization
- View organization
- Update organization

### Users

- Create users
- View users
- Assign roles
- Add users to teams

### Teams

- Create teams
- View teams
- Add/remove users from teams

### Assets

- Create asset
- View asset
- Update asset
- Deactivate asset
- List assets

### Vulnerabilities

- Create vulnerability
- View vulnerability
- Update vulnerability
- List vulnerabilities

### Findings

- Create finding
- View finding
- List findings
- Assign finding to team
- Assign finding to user
- Change finding priority
- Change finding status
- Add comments
- Mark finding as remediated
- Verify finding
- Close finding
- Accept risk

### Dashboard

The MVP dashboard should provide basic visibility into:

- Total assets
- Total open findings
- Critical findings
- High-priority findings
- Findings by status
- Findings by severity
- Findings by asset

---

## 10. MVP Use Cases

### Authentication

- UC-A01 — Register
- UC-A02 — Login
- UC-A03 — Logout

### Organization

- UC-O01 — Create organization
- UC-O02 — View organization
- UC-O03 — Update organization

### Users

- UC-U01 — Create user
- UC-U02 — View users
- UC-U03 — Assign role
- UC-U04 — Add user to team
- UC-U05 — Remove user from team

### Teams

- UC-T01 — Create team
- UC-T02 — View teams
- UC-T03 — Add user to team
- UC-T04 — Remove user from team

### Assets

- UC-AS01 — Create asset
- UC-AS02 — View asset
- UC-AS03 — Update asset
- UC-AS04 — Deactivate asset
- UC-AS05 — List assets

### Vulnerabilities

- UC-V01 — Create vulnerability
- UC-V02 — View vulnerability
- UC-V03 — Update vulnerability
- UC-V04 — List vulnerabilities

### Findings

- UC-F01 — Create finding
- UC-F02 — View finding
- UC-F03 — List findings
- UC-F04 — Assign finding to team
- UC-F05 — Assign finding to user
- UC-F06 — Change finding priority
- UC-F07 — Change finding status
- UC-F08 — Add comment
- UC-F09 — Mark finding as remediated
- UC-F10 — Verify finding
- UC-F11 — Close finding
- UC-F12 — Accept risk

### Dashboard

- UC-D01 — View security dashboard

---

## 11. MVP Out of Scope

The following features will not be implemented initially:

- Vulnerability scanner
- Automatic CVE synchronization
- GitHub App integration
- Container scanning
- Dependency scanning
- Network scanning
- Advanced risk scoring
- Complex RBAC
- SSO
- Multi-factor authentication
- Notifications
- Email integration
- SIEM integration
- Microservices architecture
- Kubernetes deployment
- Advanced reporting
- Machine learning
- AI-assisted vulnerability prioritization

These may become future features.

---

## 12. Future Integrations

VulnTrack is intended to eventually receive vulnerability data from external security tools.

Potential integrations:

```text
Dependency Scanner
        │
Container Scanner
        │
Network Scanner
        │
Repository Scanner
        │
        ▼
   VulnTrack API
        │
        ▼
     Findings
```

The Software Supply Chain Security Scanner will eventually be developed as a separate project and may integrate with VulnTrack.

---

## 13. Future Architecture

The final product may evolve into a platform composed of several components:

```text
                    ┌──────────────────┐
                    │   VulnTrack UI   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   VulnTrack API  │
                    └────────┬─────────┘
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Assets          Findings         Users
                             ▲
                             │
                       Scan Results
                             ▲
                             │
                    ┌────────┴─────────┐
                    │ External Scanners│
                    └──────────────────┘
```

The initial implementation will use a modular monolith.

```text
React
   ↓
Spring Boot REST API
   ↓
Service Layer
   ↓
Repository Layer
   ↓
PostgreSQL
```

Microservices will not be introduced unless a concrete requirement justifies them.

---

## 14. Business Model

VulnTrack is designed as a potential B2B SaaS product.

Possible future model:

### Free / Community

For small teams or individual security researchers.

Possible limitations:

- Number of assets
- Number of users
- Number of findings
- Limited integrations

### Professional

For small and medium-sized organizations.

Possible features:

- More assets
- More users
- Scanner integrations
- Advanced dashboards
- Reports
- Notifications

### Enterprise

For larger organizations.

Possible features:

- SSO
- Advanced RBAC
- Audit logs
- Multiple organizations
- Advanced integrations
- API access
- SLA/support

The business model is conceptual for the portfolio project and is not part of the initial technical MVP.

---

## 15. MVP Success Criteria

The MVP will be considered functional when an organization can:

1. Register and authenticate a user.
2. Create an organization.
3. Create users and teams.
4. Register digital assets.
5. Register vulnerabilities.
6. Create findings linking vulnerabilities to assets.
7. Assign findings to teams or users.
8. Prioritize findings.
9. Move findings through their lifecycle.
10. Add remediation comments.
11. Mark findings as remediated.
12. Verify and close findings.
13. Accept risks.
14. View the organization's current vulnerability status through a dashboard.

The MVP should demonstrate a complete vulnerability-management workflow rather than a collection of independent CRUD operations.