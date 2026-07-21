# ConditionalAccess-NamingConvention
A persona-based Conditional Access naming convention for Microsoft Entra ID, aligned to Microsoft CAF and Claus Jespersen's CA framework.

# Microsoft Entra ID — Conditional Access Naming Convention

A persona-based Conditional Access naming convention for Microsoft Entra ID, aligned to Microsoft's official CA planning guidance, the Cloud Adoption Framework (CAF), and Claus Jespersen's widely referenced CA framework.

---

## The problem

Conditional Access policies accumulate fast. Before long you end up with a mix of `CUSTOM-`, `OLD-`, `MFA-` and `BLOCK-` prefixes sitting alongside plain English names, all created at different points in time by different people. Without a naming convention you end up with multiple naming patterns which makes scanning the portal difficult. You have to click into every single policy just to understand what it does.

---

## Convention

```
CA{NNN}-{Action} | {Scope} for {Persona} [when {Condition}]
```

| Token | Required | Description |
|---|---|---|
| `{NNN}` | Yes | Zero-padded 3-digit sequence number within persona range |
| `{Action}` | Yes | `Block`, `Require`, `Enforce`, `Apply` |
| `{Scope}` | Yes | Target app or scope — `All Apps`, `O365`, `Admin Portals`, `Azure Management`, `Copilot` etc. |
| `{Persona}` | Yes | Identity group the policy targets |
| `[when {Condition}]` | Optional | Only include when the condition is the defining point of the policy |
| `[Report-only]` | Optional | Append when policy is in Report-only state. Remove when promoted to On. |

---

## Persona number ranges

Policies are grouped by identity type using number ranges. Related policies cluster together automatically when sorted in the Entra portal, and KQL queries in Sentinel can scope to a persona with a simple prefix match.

| Range | Persona | Who it covers |
|---|---|---|
| CA000–099 | Global | All identities — baseline controls that apply universally with no exceptions |
| CA100–199 | Admins | Privileged roles and elevated access accounts |
| CA200–299 | Staff | Internal employees |
| CA300–399 | Students / Externals | Non-staff identity types specific to your organisation |
| CA400–499 | Guests | B2B and external collaborators |
| CA500–599 | WorkloadIDs | Service principals and managed identities |

> **On Global:** Global means every identity in the tenant — staff, admins, guests and service principals. These are your non-negotiable baseline controls. Every other persona group gets Global policies plus their own persona-specific policies layered on top.

> **On "All Identities" vs "All Users":** Global policies use "All Identities" rather than "All Users" because Global policies apply to service principals and workload identities too, not just human accounts.

---

## Examples

| Current name | Proposed name |
|---|---|
| `Block legacy authentication` | `CA001-Block \| Legacy Authentication for All Identities` |
| `Sign-in Risk Policy` | `CA005-Block \| High Risk Sign-ins for All Identities when Sign-in Risk High` |
| `Token Protection Policy` | `CA011-Require \| Token Protection for All Identities [Report-only]` |
| `Require phishing-resistant MFA for admins` | `CA101-Require \| Phishing-Resistant MFA for Admins` |
| `Admin sign in frequency` | `CA104-Enforce \| Sign-in Frequency for Admins` |
| `CA - Require Device Compliance` | `CA106-Require \| Device Compliance for Admins [Report-only]` |
| `MFA for staff` | `CA201-Require \| MFA for Staff on All Apps` |
| `Block access from unmanaged devices` | `CA204-Block \| Unmanaged Device Access for Staff on O365 [Report-only]` |
| `MFA for Guests` | `CA401-Require \| MFA for Guests on All Apps` |
| `CA - Block Risky Workload Identities` | `CA501-Block \| Risky Workload Identities` |

---

## Why the pipe separator

The pipe `|` character is not restricted in Entra CA policy display names. It provides a clean visual break in the portal between the sequence/action token and the readable description, making policies significantly easier to scan at a glance. It is also consistent with common security group naming conventions (`SG-Device | All Corporate Devices`).

---

## Tips

- **Update Sentinel KQL before renaming.** Filter on `ConditionalAccessPolicies[*].id` rather than display name. Policy IDs are stable across renames, display names are not.
- **Rename via IaC not the portal.** Update `displayName` in your policy JSON, raise a PR and run through your pipeline.
- **One persona group per change window.** Global first, then Admins, then Staff etc. Smaller blast radius if something goes wrong.
- **Microsoft-created policies cannot be renamed.** Document their mapping in your register and move on.
- **Remove `[Report-only]` suffix** when a policy is promoted to On.

---

## References

- [Microsoft CA naming guidance](https://learn.microsoft.com/en-gb/entra/identity/conditional-access/plan-conditional-access#naming-conventions)
- [Microsoft CAF Conditional Access architecture](https://learn.microsoft.com/en-us/azure/architecture/guide/security/conditional-access-framework)
- [Claus Jespersen's CA framework for Zero Trust](https://github.com/microsoft/ConditionalAccessforZeroTrustResources)
- [Alf Løkken — Building Scalable Conditional Access (2025)](https://medium.com/@alf.lokken/building-scalable-conditional-access-a-policy-framework-for-zero-trust-3ac87175274c)

---
