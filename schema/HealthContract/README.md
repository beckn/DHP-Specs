# HealthContract Schema

**Container:** `Contract.contractAttributes`
**Extends:** `ServiceContract/v2.1`
**Protocol Version:** 2.0
**Semantic Model:** generalised
**Use Cases:** UC1 single patient engagement, UC2 camp engagement
**Tag:** health health-service contract

## Overview

HealthContract captures contract-level metadata for a health service engagement on the ONHS network. It extends ServiceContract with a `healthServiceType` discriminator, consent artefact metadata, clinical input artifact references, an artifact access endpoint, and camp linkage for UC2 multi-patient engagements.

HealthContract is the central schema of the Health* pack. Its `healthServiceType` field is the runtime discriminator that makes certain fields conditionally required in HealthPerformance, HealthConsideration, and HealthParticipant — making it the single source of truth for "what kind of health service is being contracted".

## Attachment Points

Attaches to `Contract.contractAttributes`. Present in `on_confirm` and `on_status` responses. The `healthServiceType` value is set at `select` time and remains fixed for the lifetime of the contract.

## Design Rationale

- **`healthServiceType` on Contract, not just Resource** — Although the service type is an intrinsic property of the Resource, copying it onto the Contract makes it immediately accessible to any party processing the contract without having to dereference back to the catalog. This is particularly important for Provider NP backend systems that route confirmed contracts to different processing pipelines based on service type.

- **`consent` as a structured sub-object** — DIAGNOSTIC_ANALYTICS services require explicit patient consent before any clinical data is processed. Modelling consent as a structured sub-object (rather than a document reference alone) makes the consent standard, purpose of collection, retention period, and withdrawal timestamp machine-readable — enabling automated compliance checks and data governance workflows. The `consent` field is optional in the schema; the lifecycle obligation (BAP must include consent at `init` for DIAGNOSTIC_ANALYTICS/RADIOLOGY) is a protocol rule documented in the IG, not a schema type constraint. This separation correctly reflects that consent is meaningful only at specific lifecycle stages and in specific message directions, which OpenAPI conditional logic cannot express cleanly.

- **`artifactAccessEndpoint` with dual semantics** — For DIAGNOSTIC_ANALYTICS and RADIOLOGY the endpoint is an upload target to which the Seeker NP posts clinical input files (audio samples, imaging scans). For TELECONSULTATION it is the video session join URL issued by the provider platform. A single field with type-dependent semantics avoids fragmenting the contract schema into per-service-type variants.

- **`inputArtifacts` as labeled document references** — Rather than embedding artifact binaries in the contract, `inputArtifacts` carries labels that reference entries in the Contract's `Descriptor.docs[]` array. This keeps the contract payload lean and delegates storage to the document layer.

- **`campId` linkage** — UC2 consuming engagements (per-patient contracts created under a camp) carry the parent camp HealthEntitlement ID via `campId`. This enables the Provider NP to group all patient contracts belonging to a camp for end-of-camp reconciliation without requiring a separate lookup.

## Non-Goals

- Does not model appointment scheduling or service location — those belong in HealthPerformance.
- Does not model pricing or payment — those belong in HealthConsideration.
- Does not model the clinical output (reports, diagnoses) — those belong in HealthPerformance.
- Does not model participant identity (patient ABHA, field worker) — those belong in HealthParticipant.

## Upstream Candidates

- `artifactAccessEndpoint` + `inputArtifacts` — the pattern of a provider-issued access endpoint (upload URL or session URL) returned at select time, with the seeker submitting artifact references at init, is applicable to any domain where the provider needs to receive input data before performing a service (e.g. document review services, remote diagnostics). Could be generalised in ServiceContract.
- `consent` sub-object (standard, purpose of collection, retention, withdrawal) — applicable to any domain with regulated data collection (financial services, HR, education). The structure is generic enough for the generic-service base or Beckn core.
