# TASKS — open-teaching-datasets

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) ·
> Lane: donated · Risk tier: low (escalation rules per PLAN §Quality)

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug from the tables (e.g. `open-teaching-datasets-template-001`).
- `title` — the table's Title.
- `project` — `open-teaching-datasets`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all tasks (no funded escrow). A funded task would add `fundedBudgetUsd`.
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer","open-science","data-education","bioinformatics"]`.
- `riskTier` — `low | medium | high`. **Default `low`** (synthetic data for a data-science
  audience). **`medium`** for any derived-from-real / license / de-id judgement or aggregate
  cell-size work. **`high`** (patient-facing clinical content) is **out of scope** for this project.
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. **Synthetic/derived data files →
  `dataset`**; datasheets/exercises/notebooks/methodology → `document`; tooling/code → `pr`;
  translated material → `translation`.
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until an adoption partner is confirmed.
- `verifiedNeed` — **`false`** until a named educator/program agrees to adopt a dataset (general
  need is real; per-task delivery need is unproven).
- `outputLicense` — **`CC0-1.0`** for purely-synthetic data + docs; **`CC-BY-4.0`** when calibrated
  from / derived from a CC-BY source (attribution propagated); **license propagated from source**
  for share-alike derived data; **`MIT`** for code. Never more permissive than the source allows.

---

## Milestone M0 — Foundation, safety gate & cold-start

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-template-001 | Teaching-dataset template + canonical metadata/datasheet model | writing | small | low | document | — | Technical, Educator |
| open-teaching-datasets-gate-002 | Source-eligibility + de-identification + license + safety gate checklist | design-spec | small | medium | document | — | Safety |
| open-teaching-datasets-policy-003 | NC / share-alike acceptance policy (incl. COSMIC/OncoKB) for cancer sources | design-spec | small | medium | document | gate-002 | Safety |
| open-teaching-datasets-synthesis-004 | Synthetic-data generation methodology (seed + provenanced params + determinism) | design-spec | medium | low | document | template-001 | Technical |
| open-teaching-datasets-reviewer-005 | Name/secure the Safety/eligibility reviewer (blocking gate role) | research | small | low | document | — | Maintainer |
| open-teaching-datasets-tooling-006 | Generator harness + validation script + Croissant emitter | code | medium | low | pr | template-001, synthesis-004 | Technical |
| open-teaching-datasets-outreach-007 | Adoption-channel outreach + candidate partner shortlist | research | small | low | document | — | Maintainer |
| open-teaching-datasets-pilot-008 | Pilot: synthetic survival-analysis cohort (data file) | data | medium | low | dataset | template-001, gate-002, synthesis-004, tooling-006, reviewer-005 | Safety, Technical, Domain |
| open-teaching-datasets-pilot-lesson-009 | Pilot teaching layer: exercises + solutions + notebook for the cohort | writing | small | low | document | pilot-008 | Educator, Technical |

**Acceptance criteria — key tasks**

- **template-001 (template + canonical model)**
  - [ ] Canonical metadata model documents every field from PLAN §Solution (learningObjectives,
        mode, source/license, generation seed+params **or** derivation, safety block, fields[],
        knownLimitations, biases, exercises, formats, version, doi, completenessScore,
        reproducibilityCheck).
  - [ ] Markdown template covers datasheet (Datasheets-for-Datasets), data dictionary, provenance,
        license record, known limitations & biases, and the **mandatory safety notice**.
  - [ ] Safety notice text is fixed and required on every artifact: "Synthetic (or derived from open
        aggregate data) — **not real patients, not medical advice, not for clinical use** —
        education only."
  - [ ] States deliverable is a teaching dataset for *data-skills* education; default
        `outputLicense` rules recorded (CC0 / CC-BY / MIT).
  - [ ] At least one filled-in worked example skeleton; `pnpm build && pnpm test && pnpm lint` green
        for any committed tooling; commit DCO signed-off.

- **gate-002 (eligibility + de-id + license + safety gate)**
  - [ ] Encodes the **cancer eligibility gate first**: synthetic-default; derived only if
        open-access **and** de-identified/aggregate **and** no controlled-access lineage; explicit
        **EXCLUDE** of dbGaP, EGA, individual-level biobanks, and identifiable patient data.
  - [ ] Objective license criterion: PASS only if `license.permitsReuse: true` (and
        `permitsDerivatives: true` for derived) recorded from a cited clause/URL; missing/unparseable
        = FLAG/EXCLUDE (no default-allow). Defers NC/share-alike to `policy-003`.
  - [ ] Requires recording license id + URL + snapshot (committed copy + SHA-256 + Wayback URL).
  - [ ] De-id methodology for derived mode: direct-identifier heuristics, quasi-identifier
        **k ≥ 5**, geo-precision threshold, linkage risk; never finer than source; halt-on-signal.
  - [ ] Requires the **safety notice** present + accurate, and confirms no medical-advice/clinical
        content.
  - [ ] Produces a committed, reviewable **PASS/FLAG/EXCLUDE artifact** per dataset recording which
        checks ran and what fired.

- **synthesis-004 (synthetic generation methodology)**
  - [ ] Specifies that synthetic inputs are **only published aggregate statistics** (each cited with
        value) — **never** individual-level data, even for fitting.
  - [ ] Requires a **fixed RNG seed** and a parameter file where every parameter carries
        `provenance{citation,url}`; output is deterministic.
  - [ ] Defines the **determinism/reproducibility check**: same seed + params ⇒ identical bytes;
        this is both a quality gate and the proof the data is not real.
  - [ ] Documents how clinical *plausibility* is reviewed without implying any real individual.

- **pilot-008 (synthetic survival cohort — data file)**
  - [ ] Generated by the harness from a committed seed + parameters, each parameter provenanced to a
        **published aggregate** survival/stage source (citation + value).
  - [ ] Passed `gate-002` (synthetic; no controlled-access; safety notice present) with the artifact
        committed; `riskTier: low`.
  - [ ] Fields documented (data dictionary): patient_id (synthetic), stage, treatment_arm,
        time_to_event, event_observed (censoring), etc.; clinically *plausible* censoring + survival.
  - [ ] **Reproducibility check = PASS** (independent reviewer re-runs seed/params → identical file).
  - [ ] Completeness score recorded (target ≥ 90/100); Zenodo DOI minted; `outputLicense: CC0-1.0`
        (or CC-BY-4.0 if calibrated from a CC-BY source, attributed).
  - [ ] Published via Zenodo + a training-repo PR (self-serve fallback) and, if a channel exists,
        **adopted** with the steward's `outcomes/<dataset-id>.json` evidence recorded — else
        published with the adoption blocker surfaced.

**M0 Definition of Done:** Safety/eligibility reviewer named (blocking role filled before pilot
review); template + canonical model + eligibility/de-id/license/safety gate + NC/share-alike policy
published (policy merged before any triage); synthetic-generation methodology (seed + provenanced
params + determinism) documented; generator + validation script + Croissant emitter green in CI with
golden fixtures; one synthetic pilot dataset produced end-to-end (gate PASS, ≥ 90/100 completeness,
reproducibility PASS, exercises+solution+notebook, safety notice, Zenodo DOI) and **adopted** via a
confirmed channel or **published** via the self-serve fallback with the blocker surfaced; ≥ 1
adoption-outreach thread opened.

---

## Milestone M1 — Source catalog, methodology hardened & first adoptions

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-catalog-010 | Candidate-source catalog + target learning objectives (license-classified) | research | medium | medium | document | gate-002, policy-003 | Safety |
| open-teaching-datasets-snapshot-011 | License-text snapshot capture (committed copy + SHA-256 + Wayback) | code | small | low | pr | gate-002, tooling-006 | Technical |
| open-teaching-datasets-autograde-012 | Exercise/solution framework + autograding scaffolding | code | medium | low | pr | template-001, tooling-006 | Technical, Educator |
| open-teaching-datasets-ds-expr-013 | Synthetic differential-expression matrix (data file) | data | medium | low | dataset | catalog-010, synthesis-004, tooling-006 | Safety, Technical, Domain |
| open-teaching-datasets-ds-expr-lesson-014 | Differential-expression teaching layer (exercises + notebook) | writing | small | low | document | ds-expr-013, autograde-012 | Educator, Technical |
| open-teaching-datasets-ds-clean-015 | Synthetic "messy" clinical data-cleaning dataset + exercises | data | medium | low | dataset | catalog-010, synthesis-004, tooling-006 | Safety, Technical, Educator |
| open-teaching-datasets-partner-016 | Secure first confirmed adoption partner | research | small | low | document | outreach-007 | Steward |

**Acceptance criteria — key tasks**

- **catalog-010 (candidate-source catalog)**
  - [ ] Each candidate input source assigned a disposition: `APPROVED` (open-access + license
        verified `permitsReuse:true` with cited URL + eligibility gate pass), `VERIFY-LATER`, or
        `EXCLUDED` (controlled-access lineage, NC-redistribution per `policy-003`, or identifiable).
  - [ ] dbGaP/EGA/biobanks/COSMIC-OncoKB-redistribution explicitly listed as `EXCLUDED` with reason.
  - [ ] Each `APPROVED` entry paired with a **target learning objective** (what skill it teaches).
  - [ ] Output is a committed, license-classified catalog (≥ 8 `APPROVED` candidates), ordered by
        realistic adoption path; one gate artifact per decision.

- **autograde-012 (exercise/autograding framework)**
  - [ ] Exercise schema (prompt, difficulty, solution reference, expected output) integrated with
        the canonical model.
  - [ ] Autograder runs in CI: a committed **reference solution scores full marks** and a known-wrong
        answer does not; deterministic.
  - [ ] Code MIT; `pnpm build && pnpm test && pnpm lint` green; DCO signed-off; no credentials.

- **ds-expr-013 (synthetic differential-expression matrix)**
  - [ ] Synthetic expression matrix generated from a seed + parameters provenanced to **published
        aggregate** expression characteristics; no individual-level data input.
  - [ ] `gate-002` PASS artifact committed; safety notice present; reproducibility check PASS.
  - [ ] Data dictionary covers genes × samples + group labels; known limitations/biases documented;
        completeness ≥ 90/100; Croissant metadata valid; Zenodo DOI; `outputLicense: CC0-1.0`.

- **partner-016 (first confirmed adoption partner)**
  - [ ] A named educator/program confirms they will use a dataset in real teaching.
  - [ ] Adoption mechanism documented (lesson repo PR / platform / syllabus inclusion).
  - [ ] Tasks for that partner updated to `verifiedNeed: true` with `requestor` set; steward records
        the adoption-evidence artifact.

**M1 Definition of Done:** candidate-source catalog committed (license-classified, ≥ 8 APPROVED);
≥ 3 datasets shipped (gate artifact each) across ≥ 2 skill areas; ≥ 1 confirmed adoption partner;
exercise/autograding framework green in CI; license-snapshot capture working in the decided format.

---

## Milestone M2 — Derived-from-open, epidemiology & scale

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-ds-incidence-017 | Aggregate cancer-incidence teaching dataset (derived, k≥5) | data | medium | medium | dataset | catalog-010, snapshot-011 | Safety, Technical, Domain |
| open-teaching-datasets-ds-variant-018 | Synthetic variant-annotation teaching table + exercises | data | medium | medium | dataset | catalog-010, synthesis-004 | Safety, Technical, Domain |
| open-teaching-datasets-ds-derived-019 | First derived-from-open dataset (open de-identified subset) | data | large | medium | dataset | catalog-010, snapshot-011, partner-016 | Safety, Technical, Domain |
| open-teaching-datasets-zenodo-020 | Zenodo packaging + DOI + semantic versioning automation | code | small | low | pr | tooling-006 | Technical |

**Acceptance criteria — key tasks**

- **ds-incidence-017 (aggregate incidence dataset, derived)**
  - [ ] Source is **published aggregate** incidence (SEER/GLOBOCAN-style), open-access, no
        controlled-access lineage; `gate-002` PASS with license id+URL+snapshot committed.
  - [ ] Published at the source's aggregation level or coarser; **every cross-tab cell ≥ 5** (k≥5
        recorded); no re-identification path; domain reviewer confirms de-id adequacy.
  - [ ] `outputLicense` propagated/compatible with source (CC-BY-4.0 or public-domain as
        applicable); attribution recorded; `riskTier: medium`.
  - [ ] Data dictionary + Datasheet + Croissant + ≥ 1 exercise; completeness ≥ 90/100; Zenodo DOI.

- **ds-derived-019 (first derived-from-open dataset)**
  - [ ] Source verified open-access + already de-identified/aggregate via the inspection access
        protocol (bounded sample, no full extract, no real rows committed); `gate-002` PASS.
  - [ ] Domain reviewer + Safety reviewer sign-off; `riskTier: medium`; k≥5 where aggregated.
  - [ ] License propagated from source; provenance (source URL, version, retrieval date, derivation
        steps) recorded; safety notice present; reproducible re-derivation documented.

**M2 Definition of Done:** ≥ 1 derived-from-open dataset shipped with full de-id/license artifact +
domain sign-off; ≥ 1 aggregate-incidence dataset shipped (k≥5); ≥ 6 datasets cumulative across ≥ 3
skill areas; Zenodo packaging/DOI/versioning automated; median per-dataset effort reduced vs.
M0/M1 baseline.

---

## Milestone M3 — Adoption outcomes, collection & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-adoption-021 | Track course/workshop adoptions + learner-completion outcomes | research | small | low | document | partner-016, pilot-008 | Steward |
| open-teaching-datasets-collection-022 | Published teaching collection / course pack + unifying guide | writing | medium | low | document | ds-expr-013, ds-clean-015, ds-incidence-017 | Educator, Technical |
| open-teaching-datasets-refresh-023 | Maintenance / versioning / errata + regeneration process | maintenance | small | low | document | zenodo-020, tooling-006 | Maintainer |

**Acceptance criteria — key tasks**

- **adoption-021 (adoption + learner-outcome tracking)**
  - [ ] ≥ 4 datasets adopted across ≥ 2 programs, each with a committed `outcomes/<dataset-id>.json`
        evidence artifact (syllabus/workshop URL or educator confirmation).
  - [ ] ≥ 100 learner-completions recorded (educator-reported or platform metric); ≥ 3 verifiable
        external reuse events; no self-reported-only evidence.

- **refresh-023 (maintenance/errata)**
  - [ ] Documented process to issue corrected versions; synthetic datasets **regenerate
        deterministically** from recorded seed/params (fixes are reproducible).
  - [ ] Errata + version bump flow defined; stale/incorrect datasets become `maintenance` tasks;
        reproducibility check re-run on each release.

**M3 Definition of Done:** ≥ 4 datasets adopted across ≥ 2 programs (evidence recorded); ≥ 100
learner-completions; ≥ 3 verifiable external reuse events; a published teaching collection;
maintenance/versioning/errata process documented and a steward named.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-i18n-024 | Translate a datasheet + exercises (domain-reviewed) | translation | small | medium | translation | Widens reuse; needs language + domain reviewer |
| open-teaching-datasets-r-notebooks-025 | Parallel R notebooks for shipped datasets | code | medium | low | pr | If partner audience uses R (biostatistics) |
| open-teaching-datasets-img-meta-026 | Imaging/pathology *metadata* teaching table (openly-licensed images only) | data | medium | medium | dataset | Metadata only; images must be openly licensed; no patient images |
| open-teaching-datasets-staging-027 | Synthetic TNM-staging-logic teaching dataset + exercises | data | small | low | dataset | Teaches staging rules; synthetic; not clinical guidance |
| open-teaching-datasets-registry-028 | Example consent-first registry *schema* (governance, synthetic only) | design-spec | medium | medium | document | Schema/governance example; synthetic/aggregate only; never a live store |
| open-teaching-datasets-galaxy-029 | Galaxy / Bioconductor training integration | code | medium | low | pr | Packages datasets as training material for adoption channels |

---

## Example task JSON

```json
{
  "id": "open-teaching-datasets-template-001",
  "title": "Teaching-dataset template + canonical metadata/datasheet model",
  "project": "open-teaching-datasets",
  "type": "writing",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer", "open-science", "data-education", "bioinformatics"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Cancer-data education is gatekept by data access: realistic datasets are controlled-access (dbGaP/EGA/biobanks, needing IRB/DAC approval), while freely-downloadable ones are non-oncology toy tables or large archives with murky licenses and de-identification status. This project produces small, well-documented, openly-licensed teaching datasets that are synthetic-by-default (and, when derived, only from open-access already-de-identified/aggregate sources). Before producing any dataset, the project needs one reusable template and a canonical metadata model that all outputs (datasheet, data dictionary, Croissant metadata, exercises) are projections of. Binding cancer guardrails apply: open-access/aggregate/de-identified/synthetic only; no controlled-access or identifiable patient data; education-only; not medical advice; provenance on every assertion.",
  "objective": "Create the reusable teaching-dataset template and the canonical metadata/datasheet model that every per-dataset task will use, including the mandatory non-removable safety notice and the synthetic-generation provenance fields.",
  "acceptanceCriteria": [
    "Canonical metadata model documents all fields: id, title, learningObjectives[], audience, difficulty, prerequisites[], mode{synthetic|derived}, source{name,url,license{id,url,permitsReuse,permitsDerivatives,shareAlike,snapshotRef},accessTier,deIdStatus,controlledAccessLineage:false}, generation{seed,model,params[]{name,value,provenance{citation,url}}} or derivation{steps[],aggregationLevel,kAnonymityMin}, safety{realPatients:false,notMedicalAdvice:true,notForClinicalUse:true}, fields[]{name,type,units,allowedValues,nullable,description,caveats}, knownLimitations[], biases[], exercises[]{prompt,solutionRef,difficulty}, formats[], specVersions{croissant}, version, doi, completenessScore{before,after}, reproducibilityCheck.",
    "Markdown template covers the Datasheets-for-Datasets questionnaire, data dictionary, provenance record, license record, known limitations & biases, and the teaching layer (objectives, exercises, solutions, notebook).",
    "A fixed, non-removable safety notice is specified for every artifact: 'Synthetic (or derived from open aggregate data) — not real patients, not medical advice, not for clinical use — education only.'",
    "Template states the deliverable is a teaching dataset for data-skills education (not medical content), and records the output-license rules: CC0-1.0 for purely-synthetic data+docs, CC-BY-4.0 when calibrated from/derived from CC-BY (attribution propagated), MIT for code, never more permissive than the source allows.",
    "At least one filled-in worked example skeleton is included; pnpm build && pnpm test && pnpm lint pass for any committed tooling; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\elyos\\planning\\projects\\open-teaching-datasets\\PLAN.md",
    "C:\\code\\elyos\\planning\\ROADMAP.md",
    "C:\\code\\elyos\\docs\\good-deed-definition.md",
    "Datasheets for Datasets (Gebru et al.)",
    "Croissant ML metadata specification (MLCommons v1.0)"
  ],
  "output": "A teaching-dataset documentation template plus a canonical metadata/datasheet model definition (with the mandatory safety notice and synthetic-provenance fields), committed to the project repo and ready for reuse by every per-dataset task.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Task count summary

- **M0:** 9 tasks · **M1:** 7 tasks · **M2:** 4 tasks · **M3:** 3 tasks · **Backlog:** 6 tasks.
- **Total: 29 tasks** (23 scheduled across M0–M3 + 6 backlog).
- Deliverable mix: `dataset` (synthetic/derived data files), `document` (datasheets/exercises/
  methodology/catalog), `pr` (tooling), `translation` (backlog i18n).
- Every task: `lane: donated`, `urgent: false`, `verifiedNeed: false`, `requestor: TO BE SECURED`
  until an adoption partner is confirmed (then per-partner tasks flip to `verifiedNeed: true`).
- Risk: bulk `low` (synthetic); `medium` only for license/de-id judgement and derived-from-real /
  aggregate-cell work; **no `high`** tasks (patient-facing clinical content is out of scope).

---

## Generated task index

> Auto-generated by the Elyos task-decomposition agent on 2026-06-29.
> All 29 tasks validated against `packages/schema` (draft-07, `additionalProperties:false`).
> Seed file `tasks/open-teaching-datasets-template-001.json` was pre-existing; 28 new files generated.

| File | Title | Type | Deliverable | Risk | Priority |
| --- | --- | --- | --- | --- | --- |
| open-teaching-datasets-template-001.json | Teaching-dataset template + canonical metadata/datasheet model | writing | document | low | high |
| open-teaching-datasets-gate-002.json | Source-eligibility + de-identification + license + safety gate checklist | design-spec | document | medium | high |
| open-teaching-datasets-policy-003.json | NC / share-alike acceptance policy (incl. COSMIC/OncoKB) | design-spec | document | medium | high |
| open-teaching-datasets-synthesis-004.json | Synthetic-data generation methodology | design-spec | document | low | high |
| open-teaching-datasets-reviewer-005.json | Name/secure the Safety/eligibility reviewer | research | document | low | high |
| open-teaching-datasets-tooling-006.json | Generator harness + validation script + Croissant emitter | code | pr | low | high |
| open-teaching-datasets-outreach-007.json | Adoption-channel outreach + candidate partner shortlist | research | document | low | high |
| open-teaching-datasets-pilot-008.json | Pilot: synthetic survival-analysis cohort (data file) | data | dataset | low | high |
| open-teaching-datasets-pilot-lesson-009.json | Pilot teaching layer: exercises + solutions + notebook | writing | document | low | high |
| open-teaching-datasets-catalog-010.json | Candidate-source catalog + target learning objectives | research | document | medium | medium |
| open-teaching-datasets-snapshot-011.json | License-text snapshot capture | code | pr | low | medium |
| open-teaching-datasets-autograde-012.json | Exercise/solution framework + autograding scaffolding | code | pr | low | medium |
| open-teaching-datasets-ds-expr-013.json | Synthetic differential-expression matrix (data file) | data | dataset | low | medium |
| open-teaching-datasets-ds-expr-lesson-014.json | Differential-expression teaching layer | writing | document | low | medium |
| open-teaching-datasets-ds-clean-015.json | Synthetic "messy" clinical data-cleaning dataset + exercises | data | dataset | low | medium |
| open-teaching-datasets-partner-016.json | Secure first confirmed adoption partner | research | document | low | medium |
| open-teaching-datasets-ds-incidence-017.json | Aggregate cancer-incidence teaching dataset (derived, k>=5) | data | dataset | medium | medium |
| open-teaching-datasets-ds-variant-018.json | Synthetic variant-annotation teaching table + exercises | data | dataset | medium | medium |
| open-teaching-datasets-ds-derived-019.json | First derived-from-open dataset (open de-identified subset) | data | dataset | medium | medium |
| open-teaching-datasets-zenodo-020.json | Zenodo packaging + DOI + semantic versioning automation | code | pr | low | medium |
| open-teaching-datasets-adoption-021.json | Track course/workshop adoptions + learner-completion outcomes | research | document | low | low |
| open-teaching-datasets-collection-022.json | Published teaching collection / course pack + unifying guide | writing | document | low | low |
| open-teaching-datasets-refresh-023.json | Maintenance / versioning / errata + regeneration process | maintenance | document | low | low |
| open-teaching-datasets-i18n-024.json | Translate a datasheet + exercises (domain-reviewed) | writing | translation | medium | low |
| open-teaching-datasets-r-notebooks-025.json | Parallel R notebooks for shipped datasets | code | pr | low | low |
| open-teaching-datasets-img-meta-026.json | Imaging/pathology metadata teaching table | data | dataset | medium | low |
| open-teaching-datasets-staging-027.json | Synthetic TNM-staging-logic teaching dataset + exercises | data | dataset | low | low |
| open-teaching-datasets-registry-028.json | Example consent-first registry schema (governance, synthetic only) | design-spec | document | medium | low |
| open-teaching-datasets-galaxy-029.json | Galaxy / Bioconductor training integration | code | pr | low | low |

**Fan-out dimensions:** none (no explicit enumeration dimensions in TASKS.md warranted fan-out).
**Schema note:** `i18n-024` uses `type:"writing"` + `deliverable:"translation"` per the Translation spec rule.
