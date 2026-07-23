

# Microsoft Entra ID - Conditional Access Naming Convention

A persona-based Conditional Access naming convention for Microsoft Entra ID, aligned to Microsoft's official CA planning guidance, and Claus Jespersen's widely referenced CA framework.

---

## The problem

Conditional Access policies accumulate fast. Before long you end up with a mix of `CUSTOM-`, `OLD-`, `MFA-` and `BLOCK-` prefixes sitting alongside plain English names, all created at different points in time by different people. Without a naming convention you end up with multiple naming patterns which makes scanning the portal difficult. You have to click into every single policy just to understand what it does.


---

## The goal

A good CA policy name should answer three questions without opening the policy:

1. **Who** it applies to
2. **What** it does
3. **When** it applies (if a condition is the point)

If you have to click into a policy to understand its purpose, the name has failed.

---

## What I considered

I looked at three approaches before landing on a standard.

**Option 1: Pure Microsoft Learn style**

Microsoft's CA planning guidance recommends:
```
CA{NNN} - {Cloud app}: {Response} For {Principal} When {Conditions}
```
Readable, but the freetext "When" clause diverges across admins over time. Also puts the app first which means all your admin policies and staff policies get scattered when you sort alphabetically.

**Option 2: Microsoft CAF architecture pattern**

The Cloud Adoption Framework recommends a more structured approach:
```
CA{NNN}-{Persona}-{PolicyType}-{App}-{Platform}-{GrantControl}
```
Excellent for machine readability and Sentinel filtering. Too long and hard to read at a glance in the portal, especially when troubleshooting at speed.

**Option 3: Pure pipe-based readable format**

Something like `CA-Block | Legacy Authentication` — clean and instantly readable. But no sequence number, no persona grouping, and freetext that diverges over time.

---

## What I landed on

I combined the best of all three, drawing on:

- **Microsoft's official CA naming guidance** — for the format structure
- **Claus Jespersen's CA framework** (hosted under the Microsoft GitHub org at `github.com/microsoft/ConditionalAccessforZeroTrustResources`) — for the persona-based design and number ranges, which has become the de facto standard referenced by modern identity architects
- **The pipe separator** — for readability in the Entra portal. Microsoft place no restriction on pipe, hyphen or underscore in CA policy display names

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

These ranges are a guide — you can adjust them, add more personas, or restructure the numbering to fit your organisation's needs.

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

| Old (Default) | New |
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

## What the portal looks like after

Sorting alphabetically gives you clean persona clusters:

- `CA001` onwards — all Global baseline controls together
- `CA101` onwards — all Admin policies together
- `CA201` onwards — all Staff policies together
- `CA401` onwards — Guest policies
- `CA501` onwards — WorkloadIDs policies

No more scrolling past `CUSTOM -` and `OLD -` prefixes mixed in with everything else. One glance and you know exactly what persona you are looking at and what the policy does.

---

## References

- [Microsoft Conditional Access architecture and Naming](https://learn.microsoft.com/en-us/azure/architecture/guide/security/conditional-access-framework)
- [Claus Jespersen's CA framework for Zero Trust](https://github.com/microsoft/ConditionalAccessforZeroTrustResources/blob/main/ConditionalAccessGovernanceAndPrinciplesforZeroTrust%20October%202023.pdf)
- [Alf Løkken — Building Scalable Conditional Access (2025)](https://medium.com/@alf.lokken/building-scalable-conditional-access-a-policy-framework-for-zero-trust-3ac87175274c)
---
