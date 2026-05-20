# compliance-atoms — Goals

> SOC2, HIPAA, ISO27001, PCI, GDPR mapped once with shared control families, evidence types, audit requirements, and cross-framework equivalence — ending the industry-wide redundancy of duplicate control mappings.

*This document is derived from `aish/ARCHITECTURE.md` (now `xdao/xdao/ARCHITECTURE.md` §The *-Atoms Catalogs). Sections marked **Generated** are pattern-based and are intended as a starting point for revision, not as decided plan.*

---

## What this catalog makes civilization-grade

Every enterprise duplicates this work — mapping the same controls across SOC2, HIPAA, ISO27001, PCI, GDPR. Massive industry-wide redundancy. Auditors and security teams rebuild the same matrices every year for every client.

By cataloging the primitives, `compliance-atoms` turns this domain from opaque-and-ephemeral to typed, versioned, composable, machine-readable, and open — the civilization-grade properties the ecosystem requires.

## What it catalogs

### Atom types

- **`control-family`** — Logical grouping (access-control, change-management, incident-response).
- **`control`** — Individual control statement (e.g., 'all production changes require peer review').
- **`evidence-type`** — What proves the control is operating (log, screenshot, signed attestation, ticket reference).
- **`audit-requirement`** — What an auditor needs to verify compliance (sample size, retention, freshness).

### Compositions: `frameworks`

A framework composition is a complete compliance framework mapping — SOC2 Type II, HIPAA Security Rule, ISO27001 Annex A, PCI DSS, GDPR Art. 32. Each maps control families to controls to evidence types to audit requirements.

### Rule types

- **`framework-requirement`** — What controls a framework demands (e.g., SOC2 CC6.1 requires access control to production).
- **`evidence-sufficiency`** — What evidence satisfies a control (e.g., quarterly access review report ≥ 1 page with sign-off).
- **`cross-framework-equivalence`** — Which controls in framework A satisfy controls in framework B (SOC2 CC6.1 ≡ ISO27001 A.9.1).

## Runtime consumers

- **olympus** — Governance panels collect compliance evidence on every emission. Aegis attestations roll up into framework-aligned audit packets.
- **aish** — History engine produces signed audit trails that satisfy 'change-management' and 'access-review' evidence types.

## Status & priority

**Current status:** `proposed`

**Priority tier:** Tier 3 — Build when supporting runtimes mature

**Trigger / activation condition:** Olympus governance maturation. Customer asks for SOC2-ready evidence.

## Roadmap *(Generated — milestone shapes mirror aish's roadmap pattern; revise as actual work begins)*

### v0.1 — Bootstrap & spec acceptance

**Goal:** SOC2 Type II framework cataloged. Evidence-collection schema accepted.

**Success criterion:** Olympus generates SOC2 evidence packets that pass a mock audit.

**Kill criterion:** Mock audit reveals catalog gaps too large to close without per-customer customization.

**Work:**

- [ ] XAIP: framework composition schema with cross-framework equivalence
- [ ] Catalog SOC2 Type II as first complete framework
- [ ] Define evidence-sufficiency rules
- [ ] Olympus evidence-collection integration
- [ ] Mock audit dry-run

### v0.2 — Adoption & expansion

**Goal:** Add HIPAA, ISO27001, PCI. Cross-framework equivalence working.

**Work:**

- [ ] Catalog HIPAA Security Rule
- [ ] Catalog ISO27001 Annex A
- [ ] Cross-framework equivalence resolver

### v1.0 — Operational

**Goal:** Audit-ready evidence packets generated on-demand from any combination of cataloged frameworks.

## Concrete atom example *(Generated — illustrative, not seed content)*

```yaml
frameworks/soc2-type-2/definition.yml
---
id: soc2-type-2
type: composition
version: 1.0.0
families:
  - { ref: atoms/control-family/cc6-access-control }
  - { ref: atoms/control-family/cc7-system-operations }
controls:
  - { ref: atoms/control/cc6.1, evidence: [access-review, mfa-attestation] }
  - { ref: atoms/control/cc6.2, evidence: [provisioning-log] }
audit_requirements:
  - { ref: atoms/audit-requirement/quarterly-access-review }
```

## Adoption strategy *(Generated)*

Driven by Olympus customers needing compliance evidence. First framework (SOC2) anchors the schema; subsequent frameworks reuse it.

## Civilization-grade property checklist

Every catalog must satisfy these before v1.0. Failing any blocks a release.

| Property | Mechanism in this catalog |
|---|---|
| Typed | JSON Schema in `schemas/` validates every atom, composition, rule |
| Versioned | Every atom has a semver `version` field; compositions reference atoms by version-pinned ID |
| Machine-readable | `exports/catalog.json` published on every release |
| Composable | Compositions reference atoms by ID; CI verifies references resolve and no circular dependencies |
| Open | Apache-2.0 licensed; LICENSE file present |
| Durable | No external dependencies for primary content (no remote image URLs, no vendor APIs in the hot path) |

## Related

- **Spec:** [atoms-spec](https://github.com/convergent-systems-co/atoms-spec) — the canonical structure every catalog conforms to
- **Tools:** [atoms-tools](https://github.com/convergent-systems-co/atoms-tools) — CLI for validate / export / bootstrap / resolve
- **Federation:** [xdao](https://github.com/convergent-systems-co/xdao) — ecosystem directory and discovery
- **Umbrella:** [atoms](https://github.com/convergent-systems-co/atoms) — every catalog as a git submodule
- **Manifest:** [`ATOMS.yml`](./ATOMS.yml) — this catalog's machine-readable manifest
- **Standard:** [`README.md`](./README.md) — catalog overview and contribution flow
