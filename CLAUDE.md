# Configuration Drift Detection — ServiceNow Application

## What this is

A custom scoped ServiceNow application that detects configuration drift across enterprise
application infrastructure. Drift is detected in one of two modes:

1. **Peer comparison** — one environment's configuration compared against a peer environment
   (e.g. UAT vs Production).
2. **Baseline comparison** — current configuration compared against a versioned, approved
   baseline.

The solution is built for real enterprise CMDBs, including non-CSDM-conformant ones.
Configurability is a foundational requirement, not a later enhancement.

Full design: `docs/` (solution design document, executive summary deck).
CSDM 5 reference: `docs/CSDM_5.pdf`.

---

## Governing principle: reuse over rebuild

**Do not re-implement capability the platform already provides.** Before proposing a new table,
job, or UI, check whether ServiceNow already does it. In particular:

| Need | Use this — don't rebuild |
|---|---|
| CMDB data quality measurement | CMDB Health dashboards |
| CI identification / dedup / reconciliation | IRE and reconciliation rules |
| Ad-hoc CI graph queries | CMDB Query Builder |
| Trending, KPIs, scorecards | Performance Analytics |
| CI attribute change history | `sys_history_set` / CI attribute history |
| Topology discovery | Discovery, Service Mapping, MID Server |

If a suggestion drifts toward duplicating one of the above, stop and say so rather than building it.

---

## Non-negotiable design constraints

### Schema configurability
Relationship traversal paths must **never** be hard-coded to CSDM-conformant structures.
Hard-coded paths break against non-conformant CMDBs. Every application onboarded carries its own
schema mapping that defines how to walk from the application down to the CIs in scope.

### Findings lifecycle
Findings have explicit dispositions, and they are semantically distinct with different downstream
behaviour:

- **True Positive** — real drift, requires remediation.
- **False Positive** — the check or the data was wrong; feeds check tuning.
- **Accepted Deviation** — real difference, knowingly accepted; must not resurface as new drift.

### Auto-resolution integrity
A finding may only auto-resolve when:

- **Same-check guarantee** — resolution is driven by the *same* check that raised it, not a
  different or reconfigured check.
- **Flapping guard** — a value oscillating across runs must not produce repeated
  raise/resolve cycles.

### Architecture type drives drift semantics
What "drift" means, and how reliably it can be sourced, differs by architecture type:
on-prem physical, virtualised, IaaS, PaaS/serverless, SaaS. Rollout is phased by architecture type
for this reason. Do not assume a single comparison model covers all five.

---

## Design components

- **Onboarding & controls maturity assessment** — evaluates whether governance *controls exist*
  for an application. It does **not** re-measure data quality; CMDB Health already surfaces that.
- **Graph traversal & data model** — walks the CI graph from application to in-scope CIs using the
  per-application schema mapping.
- **Drift Watch List** — the configured set of CIs and attributes actually under surveillance.
- **Baseline & comparison engine** — versioned baselines and the peer/baseline comparison logic.
- **Findings lifecycle & remediation** — raise, disposition, remediate, resolve.
- **Reporting & aggregation** — roll-up of findings; built on Performance Analytics where possible.
- **CMDB / CSDM maturity requirements** — the preconditions an org must meet before onboarding.

---

## Domain vocabulary

CSDM 5 · Business Application · Application Service · `consumes` / `depends on` / `runs on`
relationships · IRE · reconciliation rules · MID Server · Service Mapping · SDLC Component ·
`cmdb_ci_service_auto` · `sys_history_set` · CI attribute history

---

## Working conventions

- Assume strong existing ServiceNow, CSDM, and CMDB knowledge. Skip introductory explanation of
  platform concepts.
- Prefer platform-native constructs over custom script where both would work.
- Flag over-engineering explicitly rather than quietly building it.
- When a design question is genuinely open, say so instead of inventing a resolution.

---

## Repository layout

<!-- TODO: fill in — update set XML, scoped app source, or other -->

## Open items

<!-- TODO: carry across the open technical validation items from the solution design document -->
