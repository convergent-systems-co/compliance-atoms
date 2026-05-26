# XAIP: Framework Composition Schema

**Atom type:** `compliance-framework`
**Version:** 0.1
**Audience:** Compliance engineers, identity architects, platform integrators

---

## 1. Purpose

A compliance framework composition assembles `control-family`, `control`, `evidence-type`, and `audit-requirement` atoms into a complete, machine-readable compliance framework definition. The composition is the authoritative artifact from which audit tooling, policy engines, and identity-binding systems derive their rules.

---

## 2. Composition Structure

A framework composition is a JSON document at the path `frameworks/<framework_id>/v<version>/framework.json` within a catalog.

```json
{
  "framework_id": "iso27001-2022",
  "version": "2022",
  "display_name": "ISO/IEC 27001:2022",
  "control_families": [
    {
      "family_ref": "control-family:iso27001-2022/organizational",
      "display_name": "Organizational Controls"
    }
  ],
  "controls": [
    {
      "control_ref": "control:iso27001-2022/A.5.15",
      "family_ref": "control-family:iso27001-2022/organizational",
      "title": "Access control",
      "evidence_requirements": [
        {
          "evidence_type_ref": "evidence-type:access-review-log",
          "frequency": "quarterly",
          "required": true
        },
        {
          "evidence_type_ref": "evidence-type:policy-document",
          "frequency": "annual",
          "required": true
        }
      ],
      "audit_requirements": [
        {
          "audit_requirement_ref": "audit-requirement:iso27001-2022/A.5.15/annual-review"
        }
      ]
    }
  ],
  "metadata": {
    "issuing_body": "ISO",
    "effective_date": "2022-10-25",
    "catalog_ref": "compliance-atoms.convergent-systems.co"
  }
}
```

### 2.1 Required fields

| Field | Type | Description |
|---|---|---|
| `framework_id` | string | Stable, slug-form identifier (e.g., `soc2-2023`, `iso27001-2022`) |
| `version` | string | Framework edition/year |
| `controls` | array | Ordered list of control objects (see §2.2) |
| `metadata.catalog_ref` | string | The compliance-atoms catalog serving this framework |

### 2.2 Control object fields

| Field | Type | Required | Description |
|---|---|---|---|
| `control_ref` | URI | yes | Stable reference to a `control` atom |
| `family_ref` | URI | yes | Parent control-family atom |
| `evidence_requirements` | array | yes | Evidence types required for this control (see §4) |
| `audit_requirements` | array | no | Linked audit-requirement atoms |

---

## 3. Cross-Framework Equivalence

Many organizations operate under multiple frameworks simultaneously (ISO 27001 + SOC 2, PCI-DSS + NIST CSF). The composition schema supports equivalence mappings so that a single piece of evidence can satisfy controls across frameworks.

### 3.1 Equivalence declaration

Equivalences are declared at the composition level in an `equivalences` array:

```json
{
  "equivalences": [
    {
      "equivalence_id": "access-control-iso-soc2",
      "controls": [
        "control:iso27001-2022/A.5.15",
        "control:soc2-2023/CC6.1"
      ],
      "shared_evidence_types": [
        "evidence-type:access-review-log",
        "evidence-type:policy-document"
      ],
      "confidence": "full",
      "rationale": "Both controls require documented access review at minimum quarterly frequency with retained logs."
    }
  ]
}
```

### 3.2 Example: ISO 27001 A.5.15 ↔ SOC 2 CC6.1

| Attribute | ISO 27001 A.5.15 | SOC 2 CC6.1 |
|---|---|---|
| Domain | Access control | Logical and physical access |
| Core requirement | Documented access policy + review | Access restriction to authorized users |
| Evidence: access review | quarterly, required | quarterly, required |
| Evidence: policy document | annual, required | annual, required |
| Equivalence confidence | full | full |

The `confidence` field takes one of: `full`, `partial`, `directional`.

- `full` — evidence satisfying one control fully satisfies the other.
- `partial` — evidence satisfies some but not all requirements of the mapped control; a gap list is required.
- `directional` — equivalence holds in one direction only (A satisfies B, but B does not satisfy A).

---

## 4. Equivalence Resolution Algorithm

When an audit tool evaluates whether a piece of evidence satisfies a control, it runs the following resolution steps:

```
1. Locate the control reference in the target framework's controls array.
2. Retrieve the control's evidence_requirements list.
3. For each required evidence_type_ref:
   a. Check if submitted evidence carries that evidence_type_ref directly.
   b. If not, consult the framework's equivalences array for cross-framework
      mappings that include this control.
   c. For each matched equivalence with confidence == "full":
      - Accept evidence submitted under the equivalent control's type.
   d. For confidence == "partial":
      - Accept evidence for shared_evidence_types only.
      - Flag the gap list as outstanding.
   e. For confidence == "directional":
      - Accept only if the source control is the declared direction's origin.
4. Aggregate: a control is satisfied when all required evidence_type_refs
   are covered, either directly or via full/partial equivalence with no
   outstanding gaps.
5. Emit a compliance_result atom: { control_ref, satisfied: bool, gaps: [] }
```

---

## 5. Integration with identity-atoms

Compliance frameworks impose obligations on identities. The composition links to identity-atoms to declare which identities must satisfy which controls.

### 5.1 Identity compliance binding

```json
{
  "identity_compliance_bindings": [
    {
      "identity_scope": "identity:convergent-systems-co/org-member",
      "required_frameworks": ["iso27001-2022", "soc2-2023"],
      "enforcement": "block-access-on-gap",
      "review_cycle_days": 90
    },
    {
      "identity_scope": "identity:convergent-systems-co/contractor",
      "required_frameworks": ["soc2-2023"],
      "enforcement": "notify-on-gap",
      "review_cycle_days": 30
    }
  ]
}
```

### 5.2 Enforcement modes

| Mode | Behavior |
|---|---|
| `block-access-on-gap` | Access is denied to controlled resources until all gaps are resolved |
| `notify-on-gap` | Gap is surfaced to the identity's manager; access continues under escalated monitoring |
| `audit-only` | Gap is recorded but no enforcement action is taken |

### 5.3 Cross-catalog reference pattern

When a compliance framework composition references identity atoms from a different catalog:

```
identity-atoms.convergent-systems.co → identity atom URI
compliance-atoms.convergent-systems.co → control + evidence URIs
```

The binding is expressed using fully-qualified URIs:

```json
{
  "identity_scope": "https://identity-atoms.convergent-systems.co/atoms/org-member/v1/atom.json"
}
```

---

## 6. Catalog Conventions

| Convention | Value |
|---|---|
| Framework file path | `frameworks/<framework_id>/v<version>/framework.json` |
| Control atom path | `atoms/controls/<framework_id>/<control_id>/v<version>/atom.json` |
| Evidence type path | `atoms/evidence-types/<slug>/v<version>/atom.json` |
| Audit requirement path | `atoms/audit-requirements/<framework_id>/<slug>/v<version>/atom.json` |
| Equivalence map path | `frameworks/equivalences/<slug>/v<version>/equivalence.json` |

---

## 7. Related Atoms and Docs

- `control-family` atom — groups controls by domain within a framework
- `control` atom — individual requirement with evidence and audit linkage
- `evidence-type` atom — schema for a category of audit evidence
- `audit-requirement` atom — temporal and procedural constraints on evidence collection
- identity-atoms: `xaip-identity-composition.md` — how identities bind to compliance frameworks
- policy-atoms: `xaip-policy-composition.md` — how policy stacks enforce compliance obligations
