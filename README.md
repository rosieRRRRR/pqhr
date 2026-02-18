# PQHR -- Human-Readable Policy Interface

**Deterministic human rendering of cryptographic policy and enforcement state.**

* **Specification Version:** 1.0.0
* **Status:** Implementation Ready
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea
* **PQ Ecosystem:** CORE — The PQ Ecosystem is a post-quantum security framework built on deterministic enforcement, fail-closed semantics, and refusal-driven authority. Bitcoin is the reference deployment. It is not the scope.

---

## Summary

The PQ ecosystem produces cryptographically signed, CBOR-encoded,
tick-bounded artefacts that govern authority, consent, enforcement,
and identity. These artefacts are designed for machine consumption.
Humans cannot read them.

This specification defines the rendering contract: how conformant
systems MUST present policy state to human holders in a way that
is accurate, complete, and not misleading.

PQHR does not grant authority. It does not modify enforcement
semantics. It defines how enforcement state is made visible to the
humans that enforcement is supposed to protect.

**Key Properties:**
Accuracy obligation | No rendering authority | Completeness
requirement | Omission is misrepresentation | Holder-inspectable

---

## Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## Explicit Dependencies

Implementations claiming conformance with PQHR MUST meet the minimum versions below:

| Specification | Minimum Version | Rendering Purpose |
|---------------|-----------------|-------------------|
| PQSF | ≥ 2.0.3 | Canonical encoding, artefact hashing, signature envelope expectations, evidence classification |
| PQSEC | ≥ 2.0.3 | EnforcementOutcome semantics, refusal codes, predicate result fields |
| Epoch Clock | ≥ 2.1.0 | Tick interpretation conventions and freshness state semantics |
| PQAI | ≥ 1.2.0 | Drift state fields and classification enums (if rendered) |
| PQPS | ≥ 1.0.0 | Persistent state artefacts, drift/update receipts, holder edit/delete flows, verifiable deletion |

If a rendered artefact originates from a specification below the minimum version, the renderer MUST still display it, but MUST prominently mark the artefact as "UNVERSIONED/UNKNOWN FORMAT" and require Detail View and Raw View access where available.

---

## Index

- [0. Scope and Purpose](#0-scope-and-purpose)
  - [0.1 Problem Statement](#01-problem-statement)
  - [0.2 Scope](#02-scope)
  - [0.3 Design Principles](#03-design-principles)
- [1. Core Invariant (Normative)](#1-core-invariant-normative)
- [2. Rendering Fidelity Levels (Normative)](#2-rendering-fidelity-levels-normative)
  - [2.1 Summary View](#21-summary-view)
  - [2.2 Detail View](#22-detail-view)
  - [2.3 Raw View](#23-raw-view)
- [3. Artefact Rendering Requirements (Normative)](#3-artefact-rendering-requirements-normative)
  - [3.1 Policy Profile](#31-policy-profile)
  - [3.2 Consent Proof](#32-consent-proof)
  - [3.3 Enforcement Outcome](#33-enforcement-outcome)
  - [3.4 Boundary Receipt (Bridge Mode)](#34-boundary-receipt-bridge-mode)
  - [3.5 Drift Measurement (PQAI)](#35-drift-measurement-pqai)
  - [3.6 RevocationEvidence](#36-revocationevidence)
  - [3.7 Freshness State (Epoch Clock §9A)](#37-freshness-state-epoch-clock-9a)
  - [3.8 Persistent State (PQPS)](#38-persistent-state-pqps)
- [4. Prohibited Rendering Patterns (Normative)](#4-prohibited-rendering-patterns-normative)
  - [4.1 Omission](#41-omission)
  - [4.2 Minimisation](#42-minimisation)
  - [4.3 False Reassurance](#43-false-reassurance)
  - [4.4 Authority Implication](#44-authority-implication)
  - [4.5 Selective Disclosure](#45-selective-disclosure)
  - [4.6 Rendering-Dependent Enforcement](#46-rendering-dependent-enforcement)
  - [4.7 Selective Emphasis](#47-selective-emphasis)
  - [4.8 Structural Refusals MUST NOT Be Collapsed](#48-structural-refusals-must-not-be-collapsed)
  - [4.9 PQPS Deletion Rendering Obligation](#49-pqps-deletion-rendering-obligation-normative)
- [5. Holder Inspection Rights (Normative)](#5-holder-inspection-rights-normative)
  - [5.1 Right to Inspect](#51-right-to-inspect)
  - [5.2 Right to Detail](#52-right-to-detail)
  - [5.3 Right to Raw](#53-right-to-raw)
  - [5.4 Right to Export](#54-right-to-export)
  - [5.5 Right to Audit History](#55-right-to-audit-history)
- [6. Rendering Integrity (Normative)](#6-rendering-integrity-normative)
  - [6.1 Source of Truth](#61-source-of-truth)
  - [6.2 Rendering Verification](#62-rendering-verification)
  - [6.3 No Rendering Cache Authority](#63-no-rendering-cache-authority)
- [7. Cross-Specification Alignment (Normative)](#7-cross-specification-alignment-normative)
  - [7.1 Dependencies](#71-dependencies)
  - [7.2 Authority Boundary](#72-authority-boundary)
- [8. Implementation Notes (Non-Normative)](#8-implementation-notes-non-normative)
- [9. Conformance Checklist (Normative)](#9-conformance-checklist-normative)
- [10. Accessibility Requirements (Normative)](#10-accessibility-requirements-normative)
- [11. Internationalisation Requirements (Normative)](#11-internationalisation-requirements-normative)
- [Annex A -- Rendering Threat Model and Anti-Manipulation Controls (Normative)](#annex-a--rendering-threat-model-and-anti-manipulation-controls-normative)
  - [A.1 Purpose](#a1-purpose)
  - [A.2 Threat Model](#a2-threat-model)
  - [A.3 Mechanical Prohibitions](#a3-mechanical-prohibitions)
  - [A.4 Environment Isolation](#a4-environment-isolation)
  - [A.5 RenderAttestation (Optional / Profile-Scoped)](#a5-renderattestation-optional--profile-scoped)
  - [A.6 Unknown Field Handling](#a6-unknown-field-handling)
- [Annex B -- Render Model and Snapshot Testing (Normative)](#annex-b--render-model-and-snapshot-testing-normative)
  - [B.1 Canonical JSON Requirement](#b1-canonical-json-requirement)
  - [B.2 RenderModel Object](#b2-rendermodel-object)
  - [B.3 Snapshot Conformance](#b3-snapshot-conformance)
- [Changelog](#changelog)

---

## 0. Scope and Purpose

### 0.1 Problem Statement

1. Every artefact in the PQ ecosystem is binary, signed, and
   opaque to human inspection
2. Policy profiles, consent proofs, drift thresholds, boundary
   policies, and enforcement outcomes are all machine-readable only
3. A human holder cannot inspect their own governance state without
   a rendering layer
4. A rendering layer that omits, simplifies, or misrepresents
   governance state undermines the sovereignty the ecosystem exists
   to protect
5. Without a rendering specification, every implementation invents
   its own presentation, creating inconsistency and potential
   misrepresentation

### 0.2 Scope

PQHR defines:

- Mandatory rendering obligations for conformant implementations
- Required information disclosures for each artefact class
- Accuracy and completeness requirements
- Prohibited rendering patterns
- Rendering fidelity levels
- Holder inspection rights

PQHR does not define:

- Visual design, typography, or layout
- Specific UI frameworks or rendering technologies
- Accessibility standards (implementations SHOULD follow
  platform-appropriate accessibility guidelines)
- Natural language translation (implementations MAY present
  in any language provided accuracy is maintained)

### 0.3 Design Principles

1. **Omission is misrepresentation.** If a governance state exists,
   the holder MUST be able to see it
2. **Rendering has no authority.** The rendered view MUST NOT
   influence enforcement. Enforcement operates on artefacts,
   never on rendered representations
3. **Accuracy over simplicity.** A correct but complex
   rendering is preferable to a simple but misleading one
4. **Holder is the audience.** Rendering serves the human
   holder, not the system operator, not the AI, not the
   counterparty
5. **Inspectability is non-negotiable.** The holder MUST be
   able to reach the full artefact from any rendered summary

---

## 1. Core Invariant (Normative)

The rendered representation MUST be a faithful projection of
the underlying artefact.

No rendered view may omit, obscure, minimise, or misrepresent
any field that affects the holder's authority, consent, exposure,
or rights.

If an artefact contains fields unknown to the implementation, the renderer MUST:

1. Surface an "UNKNOWN FIELDS PRESENT" warning in Summary View.
2. Display the unknown fields in Detail View (name + raw value).
3. Preserve the fields in Raw View and Export.

Unknown fields MUST NOT be omitted or silently discarded.

---

## 2. Rendering Fidelity Levels (Normative)

Implementations MUST support at least two fidelity levels.

### 2.1 Summary View

A concise, human-readable presentation of the artefact's
essential meaning.

Requirements:

1. MUST present: artefact type, current state, time bounds,
   active constraints, and any active suspensions or
   degradations
2. MUST indicate when the summary omits detail
3. MUST provide a mechanism to reach the Detail View

Summary View is the default presentation.

### 2.2 Detail View

A complete, field-by-field presentation of the artefact.

Requirements:

1. MUST present every field in the artefact
2. MUST present field values accurately
3. MUST distinguish between normative fields and
   informative annotations
4. MUST present tick values with both the raw tick number
   and a human-readable time interpretation (per Epoch Clock
   informative note conventions)
5. MUST present hash values in full (truncation permitted
   only if the full value is accessible on interaction)
6. MUST present signature verification status, including:
   - VERIFIED / FAILED / NOT_PRESENT / NOT_SUPPORTED
   - Verification key identity (profile or key id) when available
   - Failure reason code when FAILED

### 2.3 Raw View

Implementations SHOULD additionally support a Raw View that
presents the canonical encoding (CBOR hex or diagnostic
notation) of the artefact.

The Raw View enables independent verification by technical
holders.

---

## 3. Artefact Rendering Requirements (Normative)

### 3.1 Policy Profile

**Summary View MUST present:**

- Policy profile identifier
- Current enforcement mode (Advisory / Bridge / Strict, as defined by PQSEC Annex AU)
- Active predicates and their current state
- Any active degradations (STALE_WARN, STALE_HARD, OFFLINE)
- Consent requirements by operation class
- Drift thresholds (as percentages for human readability, with
  fixed-point values accessible in Detail View)
- Time bounds (issued tick, expiry tick)

**The holder MUST be able to answer from the Summary View:**

- "What can happen without my consent?"
- "What requires my consent?"
- "What is currently refused?"
- "When does this policy expire?"

### 3.2 Consent Proof

**Summary View MUST present:**

- What was consented to (operation description)
- When consent was given (tick, with human-readable time)
- When consent expires
- Whether consent has been revoked
- Scope of consent (what operations it covers)

**The holder MUST be able to answer:**

- "What did I agree to?"
- "Is this consent still active?"
- "How do I revoke this?"

### 3.3 Enforcement Outcome

**Summary View MUST present:**

- Operation that was evaluated
- Decision (ALLOW / DENY / FAIL_CLOSED_LOCKED)
- If denied: the reason (refusal code in human-readable form)
- If denied: whether retryable (per PQSEC refusal code taxonomy)
- Predicates that were evaluated and their results
- Any predicates that were suspended (Bridge mode)

**The holder MUST be able to answer:**

- "What happened?"
- "Why was it allowed or denied?"
- "What can I do about it?"

### 3.4 Boundary Receipt (Bridge Mode)

BoundaryReceipt is defined by PQSEC Annex AU.

**Summary View MUST present:**

- That this operation crossed a boundary to a non-conformant
  or partially conformant system
- Which predicates were suspended
- Which predicates were preserved and their results
- The boundary class
- The counterparty's conformance level (without identifying
  the counterparty beyond what is necessary)

**The holder MUST be able to answer:**

- "Was this operation fully protected?"
- "What guarantees were suspended?"
- "Why were they suspended?"

### 3.5 Drift Measurement (PQAI)

**Summary View MUST present:**

- Current drift state (NONE / WARNING / CRITICAL)
- Drift score as a human-readable percentage
- Which model is being measured
- When the measurement was taken
- What the drift thresholds are

**The holder MUST be able to answer:**

- "Is the AI behaving consistently?"
- "How far has it drifted?"
- "What happens if it drifts further?"

### 3.6 RevocationEvidence

**Summary View MUST present:**

- What was revoked (artefact type and identifier)
- When the revocation was issued
- The reason code in human-readable form
- The effect (which predicate now evaluates FALSE)

**The holder MUST be able to answer:**

- "What changed?"
- "Why?"
- "What does this affect?"

### 3.7 Freshness State (Epoch Clock §9A)

**Summary View MUST present:**

- Current freshness state (FRESH / STALE_WARN / STALE_HARD / OFFLINE)
- If degraded: what operations are affected
- If degraded: what the holder can still do
- If degraded: what is needed to restore normal operation
- Time since last verified tick (in human-readable form)

**The holder MUST be able to answer:**

- "Is the system operating normally?"
- "What can I still do?"
- "How do I fix this?"

### 3.8 Persistent State (PQPS)

PQPS defines holder-controlled persistent state. Rendering MUST make state inspectable, editable, and revocable without implying authority.

#### 3.8.1 Summary View (Required)

Summary View MUST show:

1. Whether AI-side state access is currently:
   - AVAILABLE
   - UNAVAILABLE (stale/lease invalid/absent)
2. Last authorised tick for AI-side state access
3. Drift status summary (if present):
   - NONE, LOW, MODERATE, HIGH, CRITICAL (as defined by PQPS/PQAI integration)
4. Count of:
   - facets present
   - categories present
   - entries present
5. A prominent indicator if any delete or revocation is pending reconciliation.

#### 3.8.2 Detail View (Required)

Detail View MUST support:

A. Facet rendering:
- Show facet name, entry count, last mutation tick, and drift deltas (if measured).
- Provide a per-facet "Inspect entries" action.

B. Entry rendering:
- Each entry MUST show:
  - entry_id (short form plus full-copy affordance)
  - last_edit_tick
  - provenance class: HOLDER_EDIT, AI_PROPOSED, MERGED (if your PQPS defines these)
  - whether the entry is currently active, deleted, or tombstoned

C. Drift visualisation:
- If drift receipts exist, render:
  - drift state classification
  - measurement tick
  - the minimum fields necessary to understand what changed
- Render MUST avoid false precision. If a value is uncertain, it MUST be marked as such.

D. Edit and delete confirmations:
- Any state mutation action MUST present:
  - exactly what will change (diff-style)
  - the receipt type that will be emitted
  - the refusal condition if authorisation fails
- Delete MUST be a two-step confirmation and MUST surface the "irreversibility" semantics clearly.

#### 3.8.3 Raw View (Required)

Raw View MUST provide:

1. Canonical CBOR bytes (hex) for the PQPS artefact or receipt.
2. ReceiptEnvelope fields where applicable.
3. Signature verification status and suite_profile.

#### 3.8.4 Prohibited Patterns (Extension)

The renderer MUST NOT:
- claim "memory deleted" unless a deletion receipt exists and is validated
- auto-collapse or hide facets or categories without explicit user action
- present AI-side state availability as a "quality" metric

Refusal code: `E_RENDER_PQPS_INCOMPLETE`

#### 3.8.5 Unknown PQPS Version Fallback

If the renderer encounters a PQPS artefact or receipt with a version or schema it does not recognise, the renderer MUST display the artefact in Raw View with a visible `UNKNOWN FORMAT` marker. The renderer MUST NOT silently discard or ignore unrecognised PQPS versions.

---

## 4. Prohibited Rendering Patterns (Normative)

The following rendering patterns are non-conformant:

### 4.1 Omission

Rendering MUST NOT omit artefacts or fields that affect the
holder's authority, consent, exposure, or rights.

A system that renders a policy profile without showing active
suspensions is non-conformant.

### 4.2 Minimisation

Rendering MUST NOT minimise the significance of enforcement
states through visual de-emphasis, buried placement, or
dismissive language.

A system that renders STALE_HARD as "minor connectivity issue"
is non-conformant.

### 4.3 False Reassurance

Rendering MUST NOT present uncertain states as certain.

A system that renders UNAVAILABLE predicates as "everything
is fine" is non-conformant.

### 4.4 Authority Implication

Rendering MUST NOT imply that the rendered view itself
constitutes authority or consent.

"By viewing this summary, you agree to..." is non-conformant.

### 4.5 Selective Disclosure

Rendering MUST NOT show different governance state to
different viewers of the same artefact.

The holder and any authorised auditor MUST see the same
rendering for the same artefact at the same fidelity level.

### 4.6 Rendering-Dependent Enforcement

Enforcement MUST NOT depend on whether or how an artefact
was rendered.

An artefact that is valid when rendered but invalid when not
rendered is non-conformant. Rendering is observation, not
activation.

### 4.7 Selective Emphasis

Rendering implementations MUST NOT highlight, prioritise, or
visually emphasise specific predicates or states while omitting
or de-emphasising others in a manner that alters the operator's
perception of overall system state.

All required predicates for the operation class MUST be presented
with equal structural prominence.

Structural prominence refers to visual hierarchy tier, ordering,
and default visibility. It does not prohibit neutral grouping,
but grouping MUST NOT alter perceived severity or conceal failure.

### 4.8 Structural Refusals MUST NOT Be Collapsed

If enforcement refuses due to a structural incompatibility or unsupported extension (e.g. unknown artefact type, unknown predicate class, unsupported OPTIONAL extension field, or other non-base construct), renderers MUST:

- display the refusal class or code (if available),
- identify the incompatible construct at minimum by its type identifier, and
- state that the cause is unsupported or unrecognised capability.

Renderers MUST NOT collapse such refusals into generic messages (e.g. "Invalid request", "Something went wrong", or "Not authorised") without indicating that the refusal was caused by an unsupported or unknown construct.

### 4.9 PQPS Deletion Rendering Obligation (Normative)

When rendering state governed by PQPS (Persistent States):

1. A renderer MUST NOT display "deleted" status for PQPS-managed state unless a `pqps.delete_receipt` is validated, including verification of the `destroyed_commitment` field.
2. Presence of a PQSF `MediaDeleteReceipt` (Section 22X.4) alone is NOT sufficient to render "deleted" for PQPS-managed state. The PQPS `DeleteReceipt` takes precedence for state-level deletion claims.
3. If PQPS deletion is claimed but no valid `pqps.delete_receipt` exists, the renderer MUST display a state inconsistency indicator (e.g. "deletion unconfirmed" or "deletion receipt missing") rather than displaying "deleted".
4. This rule applies only to PQPS-managed state. For non-PQPS media, `MediaDeleteReceipt` rendering is governed by the producing specification's rules.
5. Renderers MUST NOT suppress the inconsistency indicator regardless of user preferences or theme settings. Deletion state accuracy is a holder sovereignty requirement.

---

## 5. Holder Inspection Rights (Normative)

### 5.1 Right to Inspect

The holder MUST be able to inspect any artefact that
governs their authority, consent, or exposure at any time.

"At any time" includes during degraded operation,
during enforcement, and after the fact.

### 5.2 Right to Detail

The holder MUST be able to reach the Detail View of any
artefact from any Summary View.

Implementations MUST NOT require additional authentication,
payment, or consent to access the Detail View of the
holder's own artefacts.

### 5.3 Right to Raw

Implementations SHOULD provide Raw View access. If Raw View
is not implemented, the implementation MUST disclose this
limitation.

### 5.4 Right to Export

The holder MUST be able to export any artefact that governs
their authority, consent, or exposure in a machine-readable
format suitable for independent verification.

Export MUST include the canonical encoding and signature.

### 5.5 Right to Audit History

The holder MUST be able to inspect historical enforcement
outcomes, consent records, and mode transitions that affected
their governance state.

The depth of history available MAY be limited by retention
policy, but the existence of a retention limit MUST be
disclosed.

---

## 6. Rendering Integrity (Normative)

### 6.1 Source of Truth

The canonical artefact is always the source of truth.
The rendered view is always a projection.

If the rendered view and the canonical artefact disagree,
the canonical artefact is authoritative.

### 6.2 Rendering Verification

Implementations SHOULD provide a mechanism for the holder
to verify that the rendered view accurately reflects the
canonical artefact.

This MAY be as simple as showing the artefact hash alongside
the rendering and providing a tool to independently compute
the hash.

### 6.3 No Rendering Cache Authority

Cached renderings MUST NOT be treated as authoritative.

If a cached rendering is displayed, it MUST be marked as
potentially stale and the holder MUST be able to request
a fresh rendering from the canonical artefact.

### 6.4 Rendering of Files and BlobRef (Normative)

When rendering a file or BlobRef for human approval:

1. The UI MUST display the hash (full or truncated), exact size in bytes, and media_type.
2. The UI MUST explicitly state whether the displayed preview represents the exact canonical bytes bound by the BlobRef.
3. If the rendered preview transforms content (e.g., scaling an image, truncating text, decoding binary), this MUST be disclosed.
4. Approval MUST bind to BlobRef.hash, not to a rendered view.

Failure to bind approval to BlobRef.hash constitutes non-conformant rendering.

### 6.5 Refusal Code Surface Rule (Normative)

Any refusal code emitted outside the PQHR component boundary (including over APIs, in receipts, or in cross-specification documents) MUST be a PQSEC Annex AE code. PQHR-internal debug reasons MUST NOT be surfaced as interoperable refusal codes.

---

## 7. Cross-Specification Alignment (Normative)

### 7.1 Dependencies

| Specification | Rendering Requirement |
|---------------|----------------------|
| PQSEC | Enforcement outcomes, predicate results, refusal codes |
| PQSF | Canonical encoding for Raw View |
| Epoch Clock | Tick-to-time interpretation, freshness states |
| PQAI | Drift states, model identity, fingerprint summaries |
| PQPS | Human-side state, AI-side state, facet structure |

### 7.2 Authority Boundary

PQHR defines rendering obligations only.

It does not grant authority, modify enforcement semantics,
create new predicates, or alter artefact validity.

All enforcement decisions remain exclusively within PQSEC.
PQHR makes those decisions visible. It does not make them.

### 7.3 Rendering for Agent and Gateway Operations (Normative)

Conformant renderers MUST support rendering of the following ReceiptEnvelope
types when present in the local audit store.

#### 7.3.1 pqsf.gateway_call

Summary View MUST present:
- `service_id`
- `tool_id`
- `action`
- `status` (SUCCESS / FAILED)
- `issued_tick` (with human-readable interpretation per Epoch Clock conventions)

Detail View MUST present:
- `pqsec_decision_id` (if present)
- `request_commitment` (hash only)
- `response_commitment` (hash only)

Renderers MUST NOT attempt to interpret `request_commitment` or `response_commitment`
as text. They are commitments, not payloads.

#### 7.3.2 pqsf.gateway_manifest

Summary View MUST present:
- `target_domain`
- `supported_operations` (may be truncated)
- `binary_hash` (security-critical anchor)
- `expiry_tick` status (ACTIVE / EXPIRED / NONE)

Detail View MUST present:
- Full `supported_operations` list
- `binary_hash` in full
- Signature verification status

#### 7.3.3 pqsec.agent_report

Summary View MUST present:
- `decision` (ALLOW / DENY / FAIL_CLOSED_LOCKED)
- `refusal_code` (if present)
- `operation_id`
- `issued_tick`

Detail View MUST present:
- `decision_id`
- `agent_binding`
- `evidence_refs` (when present)

If `refusal_code` is present, renderers MUST display the raw code and its
corresponding definition from the PQSEC Annex AE registry. Renderers MUST NOT
generate independent semantic descriptions for refusal codes.

#### 7.3.4 pqsf.credential_migration

Summary View MUST present:
- `service_id`
- `credential_scope` (when present)
- `status` (COMPLETE / PARTIAL / FAILED)
- `legacy_revoked`
- `issued_tick`

Detail View MUST present:
- `legacy_credential_commitment` (hash only)
- `replacement_credential_commitment` (hash only)
- Signature verification status

Renderers MUST NOT attempt to interpret credential commitments as plaintext.
They are commitments, not credential material.

#### 7.3.5 pqgw.inference_receipt

Summary View MUST present:
- `provider_id`
- `model_id`
- `status` (DELIVERED / REFUSED / PROVIDER_FAILED)
- `refusal_code` (if present, with PQSEC Annex AE.59 definition)
- `latency_bucket`
- `issued_tick` (with human-readable interpretation per Epoch Clock conventions)

Detail View MUST present:
- `decision_id`
- `intent_hash` (hash only)
- `model_identity_hash` (if present, hash only)
- `request_commitment` (hash only)
- `response_commitment` (hash only)
- Signature verification status

Renderers MUST visually distinguish PQ Gateway product refusal codes (AE.59)
from PQSEC enforcement refusal codes. A distinct indicator (label, colour, or
icon) MUST be used so holders understand that product refusals are routing or
enrollment failures, not enforcement denials.

#### 7.3.6 pqgw.policy_compiled

Summary View MUST present:
- Policy template name (informative label)
- `governance.version` (monotonic policy version)
- Number of active custom rules
- `issued_tick`

Detail View MUST present:
- Full PolicyBundle hash
- DelegationConstraint hash (if present)
- Tool Capability Profile hash (if present)
- Diff summary from previous active policy (additions, removals, modifications)
- Signature verification status

#### 7.3.7 pqgw.provider_registered

Summary View MUST present:
- `provider_name`
- `provider_class` (cloud_api / local_inference / federated)
- `privacy_classification` (provider_visible / encrypted_inference / local_only)
- Model count
- `status` (active / suspended / revoked)

Detail View MUST present:
- Full `provider_id`
- `adapter_manifest_hash`
- Per-model identity and fingerprint status (pinned / fingerprinted / unverified)
- `stp_supported`
- Signature verification status

#### 7.3.8 pqgw.provider_verification

Summary View MUST present:
- `provider_name`
- Verification result (passed / failed)
- Failure reason (if failed)
- `issued_tick`

Detail View MUST present:
- Fingerprint comparison (baseline hash vs current hash)
- Adapter binary hash verification result
- STP endpoint reachability (informative)
- Signature verification status

#### 7.3.9 pqgw.enrollment_complete

Summary View MUST present:
- Enrollment status (complete)
- Policy template selected
- Number of registered providers
- `issued_tick`

Detail View MUST present:
- Wallet public key hash (not the key)
- PolicyBundle hash
- Provider registry snapshot hash
- Signature verification status

#### 7.3.10 pqgw.usage_receipt

Summary View MUST present:
- `provider_id`
- `model_id`
- `token_usage` (total tokens)
- Whether usage was estimated (`token_usage_estimated`)
- `billing_period_id`
- `issued_tick`

Detail View MUST present:
- `user_binding` (hash only)
- `operation_class`
- Token breakdown (input / output / total)
- Signature verification status

Renderers MUST NOT interpret `user_binding` as a user identifier.
It is a SHAKE256-256 commitment, not a username or account reference.

#### 7.3.11 pqgw.payment_receipt

Summary View MUST present:
- `billing_period_id`
- Payment method (type only, no sensitive details)
- Payment status (`confirmed` / `pending` / `failed`)
- `issued_tick`

Detail View MUST present:
- `user_binding` (hash only)
- Amount and currency
- Payment method reference (masked, e.g. last four digits)
- Billing period start and end ticks
- Signature verification status

Renderers MUST NOT display raw payment credentials, card numbers, or payment processor tokens. Only masked references and hash commitments are permitted.

Renderers MUST visually distinguish payment receipts from inference and usage receipts to prevent confusion between operational and financial records.

---

## 8. Implementation Notes (Non-Normative)

- Start with Summary View for all artefact types. Detail View
  is essential but Summary View is what holders interact with
  daily.
- Use natural language for Summary View. "Your consent to send
  Bitcoin expires in 3 ticks (approximately 3 minutes)" is
  better than "expiry_tick: 1004523".
- Colour coding for freshness states is intuitive but MUST NOT
  be the only indicator (accessibility).
- Drift percentages are more intuitive than fixed-point ratios
  for Summary View. Always show the fixed-point values in
  Detail View.
- Mobile-first rendering is recommended. Holders interact with
  governance state on phones more than desktops.
- Consider progressive disclosure: Summary → Detail → Raw as
  the holder drills down.
- Export should produce a self-contained file that includes
  the artefact, its signature, and enough context to verify
  independently.

---

## 9. Conformance Checklist (Normative)

A PQHR implementation is conformant if it satisfies all REQUIRED items:

1. Implements Summary View and Detail View for all rendered artefact classes.
2. Provides navigation Summary → Detail for every artefact.
3. Does not omit fields affecting authority, consent, exposure, or rights.
4. Surfaces unknown fields (warning + display + export).
5. Displays signature verification status (VERIFIED / FAILED / NOT_PRESENT / NOT_SUPPORTED).
6. Provides export of canonical artefact bytes + signature for holder-owned artefacts.
7. Provides access to historical outcomes and mode transitions subject to disclosed retention limits.
8. Renderer enforces mechanical prohibitions per Annex A.3.
9. Renderer environment isolation satisfies Annex A.4.
10. Unknown field handling conforms to Annex A.6.
11. If RenderAttestation is implemented, artefact conforms to Annex A.5 schema.
12. If the active PQSEC Implementation Profile is `pqsec-profile-highassurance-v1`, RenderAttestation is implemented and conforms to Annex A.5 and A.5.1.

Annex A conformance is REQUIRED for high-assurance profiles (PQSEC Annex BA `highassurance-v1`).

---

## 10. Accessibility Requirements (Normative)

Conformant renderers MUST:

1. Provide screen-reader labels for:
   - refusal codes
   - predicate names
   - ALLOW/DENY/LOCK outcomes
   - verification status

2. Provide a non-colour-only encoding of status:
   - text labels MUST exist for all states
   - icons MAY be used but MUST be redundant to text

3. Provide full-value access:
   - hashes, ids, and signatures MUST be copyable in full
   - truncation MUST be reversible via tap/click

4. Provide consistent focus order and keyboard navigation for:
   - Summary, Detail, Raw views
   - all confirmation dialogs

Refusal code: `E_RENDER_ACCESSIBILITY_NONCONFORMANT`

---

## 11. Internationalisation Requirements (Normative)

Renderers MUST:

1. Treat all cryptographic identifiers (hashes, signatures, ids) as language-neutral and never localise them.
2. Localise human labels only (state names, headings, warnings).
3. Preserve deterministic ordering:
   - list ordering in Detail View MUST reflect canonical ordering rules (e.g. lexicographic ids) even when labels are localised.
4. Avoid locale-specific number formatting for identifiers. Numeric ids used as identifiers MUST render as ASCII without separators.

Refusal code: `E_RENDER_I18N_NONCONFORMANT`

---

## Annex A -- Rendering Threat Model and Anti-Manipulation Controls (Normative)

### A.1 Purpose

This annex defines mandatory and profile-scoped controls to prevent rendering-layer manipulation of EnforcementOutcome artefacts.

Rendering is observational only. It MUST NOT:

* Alter enforcement state
* Suppress or transform refusal codes
* Modify predicate result visibility
* Introduce interpretive assertions not present in the source artefact

Rendering is not authority. EnforcementOutcome remains the sole decision source of truth.

### A.2 Threat Model

The renderer SHALL treat the following as adversarial goals:

1. Omission of refusal or predicate detail
2. Visual minimisation or deprioritisation of adverse results
3. Locale-based semantic distortion
4. Selective emphasis or reassurance framing
5. Unknown-field injection masking
6. Environment-based output variance
7. Renderer compromise or build tampering

These are considered rendering-layer exploit attempts.

### A.3 Mechanical Prohibitions

The renderer MUST:

* Display decision, refusal codes, and predicate results with equal prominence tiers
* Render all normative fields defined by the artefact schema
* Surface unknown fields with a visible warning marker
* Prevent truncation of refusal reasons in summary views
* Provide deterministic expansion from summary to detail view

The renderer MUST NOT:

* Collapse refusal codes into generic error messages
* Replace refusal semantics with advisory language
* Suppress adverse predicates in compact views
* Hide fields based on locale or UI theme
* Vary structural prominence based on user role, theme, locale, or device class

### A.4 Environment Isolation

Rendering environments MUST:

* Operate read-only with respect to enforcement artefacts
* Prevent renderer-to-enforcement callbacks
* Disallow side effects during render execution

Renderer state MUST NOT influence EnforcementOutcome evaluation.

### A.5 RenderAttestation (Optional / Profile-Scoped)

Implementations MAY emit a RenderAttestation artefact.

If emitted, it MUST bind:

```
RenderAttestation = {
  renderer_build_hash: bstr(32),           ; SHAKE256-256(build_bytes)
  input_artefact_hashes: [* bstr(32)],     ; SHAKE256-256(canonical_bytes_i) per artefact
  rendered_output_hash: bstr(32),          ; SHAKE256-256(rendered_output_bytes)
  locale_identifier: tstr,                 ; locale string used during rendering
  render_config_hash: bstr(32)             ; SHAKE256-256(render_config_bytes)
}
```

Purpose:

* Detect locale-based manipulation
* Detect environment variance
* Detect renderer build substitution
* Detect post-render output tampering

If RenderAttestation is present, it MUST be independently verifiable.

RenderAttestation MUST NOT grant authority. It is evidence only.

#### A.5.1 High-Assurance Requirement (Normative)

If the active PQSEC Implementation Profile is
`pqsec-profile-highassurance-v1` (as defined in PQSEC Annex BA),
RenderAttestation support is REQUIRED.

In such deployments:

1. Every Summary → Detail rendering transition for an artefact that
   affects authority, consent, exposure, or rights MUST be covered
   by a RenderAttestation artefact.
2. RenderAttestation MUST bind the exact canonical artefact bytes
   rendered, not a transformed representation.
3. Absence of RenderAttestation in a high-assurance deployment
   constitutes non-conformance with PQHR and with the declared
   PQSEC implementation profile.
4. RenderAttestation MUST include the EnforcementOutcome.decision_id
   when rendering an EnforcementOutcome artefact, binding the rendering
   to a specific enforcement decision instance.

### A.6 Unknown Field Handling

If an artefact contains unknown normative fields:

* Renderer MUST display a visible integrity warning
* Renderer MUST NOT silently ignore unknown fields
* Renderer MUST allow raw detail view inspection

---

## Annex B -- Render Model and Snapshot Testing (Normative)

### B.1 Canonical JSON Requirement

RenderModel JSON MUST use JCS Canonical JSON (RFC 8785).

All snapshot comparisons MUST be performed on JCS-canonicalized JSON bytes.

### B.2 RenderModel Object

A conformant renderer MUST be able to produce the following deterministic JSON object for any rendered artefact:

```
RenderModel = {
  v: 1,
  artefact_type: tstr,
  artefact_hash: bstr(32),
  fidelity: "SUMMARY" | "DETAIL" | "RAW",

  rows: [* {
    key: tstr,
    value: tstr,
    value_kind: "TEXT" | "TICK" | "HASH" | "BOOL" | "ENUM" | "COUNT",
    normative: bool
  }],

  warnings: [* tstr] / null,

  signature_status: "VERIFIED" | "FAILED" | "NOT_PRESENT" | "NOT_SUPPORTED",
  signature_key_hint: tstr / null,
  unknown_fields_present: bool
}
```

Rules:

1. `rows` MUST be sorted lexicographically by `key` in bytewise order.
2. All HASH values MUST be full-length (no truncation) in the Render Model. Truncation is a UI presentation concern, not a model concern.
3. DETAIL view MUST include raw tick numeric value as decimal ASCII.
4. Any human-time formatting MUST be in a separate row with `normative: false`.
5. RenderModel generation MUST be deterministic for identical canonical artefact bytes and identical `locale_identifier`.
6. `warnings` MUST include `"UNKNOWN_FIELDS_PRESENT"` if `unknown_fields_present` is true.

### B.3 Snapshot Conformance

Implementations claiming PQHR conformance MUST publish a snapshot test pack containing:

1. Input canonical artefact bytes (CBOR or JCS bytes as applicable)
2. Expected SUMMARY RenderModel (JCS canonical JSON)
3. Expected DETAIL RenderModel (JCS canonical JSON)
4. Expected RenderAttestation values when RenderAttestation is enabled

Snapshot comparisons MUST be byte-identical for:

* RenderModel JSON (after JCS canonical serialization per RFC 8785)
* RenderAttestation fields

This annex defines the Render Model contract only. It does not define UI layout.

---

## Changelog

### Version 1.0.0

Initial specification defining human-readable policy interface.

* Two mandatory fidelity levels: Summary View, Detail View
* Optional Raw View for technical holders
* Rendering requirements for seven artefact classes
* Six prohibited rendering patterns
* Five holder inspection rights
* Rendering integrity requirements
* Cross-specification alignment
* Added **§7.3.5 -- pqgw.inference_receipt rendering**: Summary and Detail View requirements for inference receipts. Includes normative requirement to visually distinguish PQ Gateway product refusal codes (PQSEC Annex AE.59) from PQSEC enforcement refusal codes.
* Added **§7.3.6 -- pqgw.policy_compiled rendering**: Summary and Detail View requirements for policy compilation receipts, including diff summary from previous active policy.
* Added **§7.3.7 -- pqgw.provider_registered rendering**: Summary and Detail View requirements for provider registration receipts, including per-model identity and fingerprint status.
* Added **§7.3.8 -- pqgw.provider_verification rendering**: Summary and Detail View requirements for provider verification receipts.
* Added **§7.3.9 -- pqgw.enrollment_complete rendering**: Summary and Detail View requirements for enrollment completion receipts.
* Added **§7.3.10 -- pqgw.usage_receipt rendering**: Summary and Detail View requirements for usage receipts, including token estimation indicator.
* Added **§7.3.11 -- pqgw.payment_receipt rendering**: Summary and Detail View requirements for payment receipts.
* Added **Section 4.9 -- PQPS Deletion Rendering Obligation**: renderers MUST NOT display "deleted" for PQPS-managed state without validated pqps.delete_receipt. MediaDeleteReceipt alone is insufficient. Links to PQPS C.1.3.1 and PQSF 22X.6.
* Updated **dependency table** to require PQSF ≥ 2.0.3, PQSEC ≥ 2.0.3, PQAI ≥ 1.2.0, PQPS ≥ 1.0.0.
* Added Annex A -- Rendering Threat Model and Anti-Manipulation Controls (Normative), defining adversarial rendering goals, mechanical prohibitions, environment isolation requirements, optional RenderAttestation artefact schema with locale and build hash binding, and unknown field handling discipline.
* Updated §9 Conformance Checklist with Annex A requirements. Annex A conformance is REQUIRED for high-assurance profiles.

---

## Acknowledgements (Informative)

This specification exists because security that cannot be
inspected by the people it protects is not security. It is
obscurity with better cryptography.

The author acknowledges:

* **Satoshi Nakamoto**, for demonstrating that trustless systems
  require transparent verification.
* **The PQ ecosystem contributors**, whose artefact designs
  provide the substrate that this specification makes visible.

Any omissions are unintentional.

---

If you find this work useful and wish to support continued development, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw