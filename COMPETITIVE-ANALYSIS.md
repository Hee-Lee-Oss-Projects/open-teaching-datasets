# Competitive & Improvement Analysis — `open-teaching-datasets`

> Scope: rigorous review of the Elyos cancer-research good-deed project that produces small,
> well-documented, OPEN teaching datasets (de-identified / synthetic) for cancer-data education.
> Prepared 2026-06-29. Sources cited inline; URLs collected at the end of each section.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on safety architecture: synthetic-by-default, a blocking
eligibility/de-id/license gate, provenance-on-every-parameter, determinism-as-proof-it-isn't-real,
and a non-removable safety notice. The 17 required sections are present and the schema-conformance
self-review is credible. The findings below are gaps and risks, ordered by importance.

**A. De-identification / synthetic provenance — mostly rigorous, two real holes.**
- The "calibrate only to *published aggregate* statistics, never individual-level records"
  rule is the single most important correctness decision and it is correctly front-loaded. Good.
- **Hole 1 — small-cell leakage through synthetic calibration.** The plan enforces k≥5 for
  *derived* cross-tabs but does **not** impose an equivalent floor on the *aggregate inputs to
  synthetic generation*. A published table can itself contain small cells (e.g. a rare-subtype
  stage×age cell of n=3). Calibrating a generator tightly to a sparse published cell can make the
  synthetic output effectively memorize/expose that cell. The gate should require that calibration
  inputs are themselves k≥5 (or coarsened) and that the generator is checked for
  over-fitting/memorization of sparse parameters. This is a genuine, citable failure mode in
  synthetic-IPD work (SynthIPD, CK4Gen).
- **Hole 2 — "plausible but synthetic" can still mislead.** Determinism proves the data isn't
  *real*, but it does **not** prove it is *clinically valid*. The plan leans on a domain reviewer
  for plausibility but has no quantitative fidelity check (e.g. does the synthetic KM curve recover
  the cited survival quantiles within tolerance? do marginals match the cited distribution?). Add a
  **statistical-fidelity acceptance test** (recover cited aggregates within a stated tolerance) as
  a ship gate, distinct from determinism.

**B. Datasheets / documentation.** The canonical metadata model is the plan's best asset and the
0–100 completeness score (≥90 to ship) makes "well-documented" checkable — a real differentiator
(see §3). Two gaps: (i) the model cites Gebru "Datasheets for Datasets" but does **not** adopt a
**health-specific** documentation overlay, even though the literature now argues generic datasheets
under-serve clinical data (Healthcare-AI Datasheet, DAIMS, Biomedical Data Manifest — §2/§3). For a
*cancer* teaching commons this is a missed easy win. (ii) Croissant v1.0 is pinned but there is no
field reserved for **synthetic-data provenance in the Croissant `RAI` (Responsible AI) extension** —
worth aligning to so the synthetic label survives ingestion by HuggingFace/Kaggle/OpenML.

**C. Teaching-appropriateness & size.** Strong: learning-objective-first design ("build datasets to
teach a thing, not data in search of a lesson"), graded difficulty, exercises+solutions+notebooks,
autograding. Gap: **no explicit size/complexity ceiling tied to the 90-minute-lesson claim** in the
problem statement. A concrete budget (e.g. ≤ a few thousand rows / loads in seconds / runs on a
laptop with no GPU) should be a documented design constraint, not prose.

**D. License per dataset.** Excellent and conservative: accepted-list, NC default-EXCLUDE
(COSMIC/OncoKB named), cited clause + SHA-256 snapshot + Wayback, never-relicense-more-permissive.
One subtlety: the plan treats **calibration from a CC-BY aggregate** as forcing CC-BY output, but
the legal question of whether a synthetic dataset *calibrated to* (not copied from) an aggregate is
a "derivative work" is genuinely unsettled and jurisdiction-dependent. The plan should state this is
a **deliberately conservative attribution choice, not a claimed legal necessity** — otherwise it
implies a copyright theory that may not hold and that could confuse downstream reusers.

**E. Avoiding real patient data.** Airtight at the policy layer (EXCLUDE list: dbGaP/EGA/biobanks/
identifiable; registry-as-schema-only; inspection protocol bounded to ≤1k rows/≤5MB header reads).
The residual risk is **human error in the derived lane**, which the plan correctly gates to medium
and routes through a domain reviewer. Acceptable.

**F. Versioning.** SemVer + Zenodo DOI per release + deterministic regeneration for errata is
correct and better than most competitors. Gap: no stated **deprecation/erratum-propagation**
mechanism for datasets already adopted downstream (how does an instructor learn a dataset they
forked was corrected?). Add an errata feed / "supersedes" pointer in the metadata.

**G. Scope vs. siblings.** The plan references `open-data-datasheets` but the prompt names three
cancer siblings — **`synthetic-cancer-data`**, **`cancer-dataset-datasheets`**, and learner-side
projects (`bioinformatics-from-zero`, `oncology-data-literacy`). The plan does **not** draw an
explicit boundary against `synthetic-cancer-data` (which presumably also generates synthetic cancer
data) or `cancer-dataset-datasheets` (which presumably documents existing datasets). **This is the
biggest completeness gap**: without an interface contract, the three projects will overlap on
synthetic generation and datasheet tooling. Recommended split: `synthetic-cancer-data` owns the
*generation engine/methodology*; `cancer-dataset-datasheets` owns the *datasheet standard + audits
of third-party datasets*; `open-teaching-datasets` **consumes both** and owns the *pedagogical
layer* (objectives, exercises, notebooks, adoption). Make this a dependency, not a duplication.

**H. Learning-objective alignment.** Good in principle; the success metrics correctly center
adoption and learner-completions over vanity counts. Gap: there is **no rubric mapping each dataset
to a named competency framework** (e.g. a survival-analysis or data-literacy skills map). Without it,
"teaches the stated skill" is reviewer-subjective. Tie objectives to an external competency taxonomy.

Sources: https://arxiv.org/pdf/2509.16466 · https://arxiv.org/pdf/2410.16872 ·
https://arxiv.org/html/2501.05617 · https://arxiv.org/pdf/2501.14094 ·
https://www.nature.com/articles/s41597-026-06670-0 · https://mlcommons.org/working-groups/data/croissant/

---

## 2. Competitive landscape

| Resource | What it is | Strengths | Weaknesses (vs. our thesis) |
|---|---|---|---|
| **UCI ML Repository — Breast Cancer Wisconsin (Diagnostic/Original)** | The canonical teaching cancer dataset; 569×30 FNA features | Free, tiny, clean, ubiquitous, well-known target | Decades-old, **overused to the point of memorization**; near-100% accuracy makes it pedagogically inert; FNA-image-features only — no survival, no genomics, no staging, no "messy" data; minimal datasheet-grade documentation |
| **scikit-learn toy datasets (`load_breast_cancer`)** | A bundled copy of UCI WDBC | Zero-friction (`return_X_y`, `as_frame`); preloaded, no I/O | Same single overused dataset; "classic and very easy binary classification" by sklearn's own docs; teaches modeling, not data realities (no missingness, no provenance work, no censoring) |
| **Kaggle Datasets** | Community upload hub; many "cancer" sets | Volume, variety, notebooks, Croissant export | **Provenance/licensing crisis**: ~35% ad-hoc licenses; documented retractions from flawed medical Kaggle sets (stroke/diabetes sets with no provenance; a "stroke" face-image set containing celebrities and children). Self-reported metadata. The anti-pattern we exist to fix |
| **Bioconductor data packages (`curatedTCGAData`, `RTCGA`, `RTCGA.clinical`)** | Real TCGA multi-omics as R objects | Real, rich, integrated (MultiAssayExperiment), workshop-proven | Large, R-only, steep ramp; built for research not 90-min lessons; documentation is vignette-style not datasheet-style; not "designed to teach a graded objective" |
| **TCGA / GDC teaching subsets** | Real tumor genomics archive | Maximally realistic | Controlled-access lineage concerns for some tiers; huge; license/processing provenance murky to a novice; cannot drop into a classroom safely without curation — exactly our gap |
| **R built-in / survival pkg datasets (`flchain`, `lung`, SUPPORT)** | Small survival datasets shipped with R packages | Tiny, documented, beloved for Cox/KM teaching | Mostly **not cancer-specific** (flchain/SUPPORT are general); old; documentation is help-page not datasheet; licensing often implicit |
| **Vanderbilt Biostatistics Datasets (hbiostat.org/data)** | Frank Harrell's curated teaching/biostat datasets | Genuinely curated for teaching; citable; survival-rich | Citation-courtesy licensing (not a clean open license); not cancer-focused; no Croissant/machine-readable metadata; no synthetic mode |
| **OpenML** | Open dataset+benchmark platform; co-created Croissant | Machine-readable metadata, reproducible tasks, **400k+ Croissant datasets**, API | General-purpose; cancer content is just re-hosted UCI/Kaggle; ~25% invalid Croissant; no pedagogy/exercise layer, no synthetic-provenance, no de-id guarantees |
| **Papers with Code / benchmark hubs** | Leaderboard-linked dataset index | Discoverability, SOTA linkage | Benchmark-first not teaching-first; no datasheets, no exercises, no de-id/synthetic guarantees; many medical sets inherit Kaggle/UCI provenance problems |
| **Synthea / SEER-derived synthetic cohorts** | Synthetic patient generators; SEER-calibrated synthetic cancer cohorts (e.g. 360k cases; 50k laryngeal) | Synthetic, license-free, realistic, modular; proves the synthetic-from-aggregate approach works | Built for HIT/EHR dev & ML, **not classroom pedagogy**; large; no graded exercises/objectives; documentation is engine-level not lesson-level — adjacent, not a teaching competitor |

**Net read.** There is no incumbent that is *simultaneously* (a) cancer-specific, (b) small and
classroom-sized, (c) synthetic-or-open with airtight de-id, (d) datasheet-grade documented with a
machine-readable completeness score, and (e) shipped with graded exercises/solutions/notebooks and
a real adoption channel. Each competitor wins on one or two axes and loses the rest.

Sources: https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic ·
https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_breast_cancer.html ·
https://scikit-learn.org/stable/datasets/toy_dataset.html ·
https://www.the-scientist.com/flawed-kaggle-datasets-used-in-clinical-models-spark-retractions-74564 ·
https://www.medrxiv.org/content/10.64898/2026.02.24.26347028v1 ·
https://bioconductor.org/packages/release/data/experiment/html/curatedTCGAData.html ·
https://bioconductor.org/packages/release/bioc/html/RTCGA.html ·
https://hbiostat.org/data/ · https://pmc.ncbi.nlm.nih.gov/articles/PMC12416095/ ·
https://mlcommons.org/2024/03/croissant_metadata_announce/ ·
https://synthetichealth.github.io/synthea/ · https://github.com/lhs-open/synthetic-data ·
https://www.simonwenkel.com/2018/08/07/exploring-less-known-datasets-for-machine-learning.html

---

## 3. Gaps we can fill

1. **Retire the over-fitted Wisconsin monoculture.** The field teaches cancer ML on one 569-row
   FNA dataset (UCI/sklearn) that is so clean it yields ~100% accuracy and rewards memorization, not
   learning. Practitioners openly call iris/titanic/MNIST "completely overused"; the same is true of
   WDBC for cancer. We can offer **graded, varied, modern alternatives** (survival with censoring,
   differential expression, variant tables, deliberately messy cleaning sets, staging logic).
2. **Provenance/licensing trustworthiness.** The Kaggle medical-data crisis (retractions, no
   provenance, ad-hoc licenses, celebrity/child images mislabeled as stroke) is a standing
   indictment. A **license-verified, provenance-on-every-assertion, snapshot-captured** commons is a
   direct answer to a documented, embarrassing failure mode.
3. **Datasheet-grade docs with a checkable score.** Almost no teaching cancer dataset ships a real
   datasheet; health-AI documentation literature says generic datasheets under-serve clinical data.
   We can ship the **first cancer-teaching datasets with a health-aware datasheet + 0–100
   completeness score + machine-readable Croissant**.
4. **Pedagogy as a first-class artifact.** UCI/Kaggle/OpenML/Bioconductor ship *data*; almost none
   ship *learning objectives + graded exercises + worked solutions + runnable notebooks*. The lesson
   layer is the gap.
5. **Classroom-sized + safe-by-construction.** Bioconductor/TCGA are realistic but too big and
   research-shaped; sklearn is small but inert. A **90-minute, laptop-runnable, synthetic-or-open**
   middle ground does not exist for cancer.
6. **Survival/Cox/KM cancer-specific teaching data.** The good small survival datasets (flchain,
   SUPPORT, lung) are general, old, and lightly licensed; cancer-specific, openly-licensed,
   well-documented survival teaching sets are scarce.
7. **Reproducibility as a guarantee.** Determinism-from-seed (byte-for-byte) is a guarantee no
   incumbent teaching dataset offers and that doubles as proof-of-non-realness.

Sources: as §2; plus https://arxiv.org/html/2501.05617 ·
https://www.biorxiv.org/content/10.1101/2025.11.18.689064.full.pdf

---

## 4. Differentiators to win

- **Safety-by-construction + trust.** Synthetic-default + airtight de-id gate + provenance-on-every-
  parameter + license snapshots = the *trustworthy* cancer teaching commons in a field with a
  documented trust crisis. This is the single strongest differentiator.
- **The whole lesson, not just the table.** Objective-first design with exercises, solutions,
  notebooks, and autograding — a drop-in lesson, not a CSV.
- **Checkable documentation.** A 0–100 completeness score (≥90 to ship) + Croissant turns
  "well-documented" from a vibe into a gate; nobody else does this for cancer teaching data.
- **Reproducible to the byte.** Anyone can regenerate and verify — auditable trust + free errata.
- **Adoption-as-done.** "Delivered, not merged": a dataset only ships when a real course/workshop
  adopts it (or a verifiable external reuse event occurs). Competitors optimize downloads; we
  optimize learning outcomes.
- **A reusable standard, not just artifacts.** The template + metadata model can become *the* way
  cancer teaching datasets are documented (see §7).

---

## 5. Claude API leverage — and the hard limits

**Where Claude adds genuine leverage (all human-reviewed):**
1. **Documentation & datasheet drafting at scale.** Generate first-draft Datasheets-for-Datasets,
   data dictionaries (per-column type/units/caveats), known-limitations/biases sections, and the
   safety notice from the canonical metadata object — then a human verifies. High-volume, structured,
   review-friendly. (Use prompt caching on the standard template; structured tool-use to emit the
   canonical JSON so outputs are validatable, not prose.)
2. **Teaching layer generation.** Draft learning objectives, graded exercises, worked solutions,
   instructor notes, and notebook scaffolds aligned to a stated competency — Claude is strong at
   pedagogical scaffolding and at producing parallel Python/R variants.
3. **Synthetic-data *specification* assistance + reproducibility narration.** Claude can draft the
   generator's parameter file *structure*, write the provenance scaffolding, propose plausible
   schema/field semantics, and explain the generation recipe — while the actual numeric calibration
   stays in deterministic, audited code.
4. (Secondary) **Triage assistant** for the source catalog: summarize a source's stated license/
   access tier and flag candidates — as a *first-pass funnel*, never the decision.

**Where Claude must NOT decide (hard gates — human/tooling-owned):**
- **De-identification / synthetic validity is not LLM-assured.** Whether a synthetic dataset leaks a
  sparse cell, or whether a derived dataset is truly de-identified, must be established by
  **deterministic statistical tests + a named human reviewer**, not by an LLM's say-so. An LLM "looks
  fine" is not a de-id guarantee.
- **No real patient data, ever — and Claude cannot be the thing that "checks" that.** Source
  eligibility (open-access, no controlled-access lineage) is a human gate with a committed artifact.
- **License verification is human-verified.** Claude may *summarize* a license; a human records
  `permitsReuse/permitsDerivatives` with a cited clause + snapshot. NC default-EXCLUDE is policy,
  not model judgment.
- **No fabricated provenance.** Every calibration parameter must cite a *real, retrievable* aggregate
  source. LLMs hallucinate citations; provenance must be checked against the actual source, and any
  parameter whose citation can't be verified is rejected. This is the highest-risk misuse and must be
  enforced by tooling (citation-resolves check) + reviewer, not trusted to the model.
- **No clinical interpretation.** Claude must not author medical/clinical narrative; that content is
  out of scope and `riskTier: high`.

Reference for API specifics (models/pricing/caching/tool-use): see the `claude-api` skill before
implementing — do not hardcode model IDs or prices from memory.

---

## 6. Ten concrete optimizations

1. **Add a k≥5 floor (and an anti-memorization check) to *synthetic* calibration inputs**, not just
   to derived cross-tabs — closes the sparse-cell leakage hole (§1.A).
2. **Add a statistical-fidelity acceptance test** as a ship gate: the synthetic output must recover
   each cited aggregate (KM quantiles, marginals) within a stated tolerance. Fidelity ≠ determinism.
3. **Adopt a health-aware datasheet overlay** (map Datasheets-for-Datasets → Healthcare-AI Datasheet
   / DAIMS / Biomedical Data Manifest fields) so cancer specifics are captured.
4. **Emit the synthetic label + provenance into Croissant's Responsible-AI extension** so the
   "synthetic / not real patients" status survives ingestion by HuggingFace/Kaggle/OpenML.
5. **Codify a hard size/runtime budget** (e.g. ≤ a few-thousand rows, loads in seconds, no GPU,
   runs in a free notebook) as a documented design constraint tied to the 90-minute claim.
6. **Write the explicit sibling interface contract** (§1.G): consume `synthetic-cancer-data`'s
   engine and `cancer-dataset-datasheets`' standard; own only the pedagogy + adoption. Prevents
   triplicated effort.
7. **Map every dataset to a named competency taxonomy** so "teaches the stated skill" is rubric-
   checkable by the educator reviewer, not subjective.
8. **Add an errata/"supersedes" feed** in the metadata so already-adopted forks can learn a dataset
   was corrected; pair with deterministic regeneration.
9. **Reframe CC-BY-on-calibration as a conservative choice, not a legal necessity** (§1.D), to avoid
   asserting an unsettled "synthetic = derivative work" copyright theory.
10. **Build a citation-resolves check** into CI: every `generation.params[].provenance.url/citation`
    must resolve and (where possible) the cited value must be machine-extracted/asserted, catching
    fabricated or drifted provenance before review.

---

## 7. Parallel & perpendicular spin-offs

- **(parallel) `synthetic-cancer-data`** — the shared generation engine/methodology. This project
  should *depend on* it for generation and *return* hardened, classroom-tested generators. Avoid
  rebuilding a generator here.
- **(parallel) `cancer-dataset-datasheets`** — the datasheet *standard* + audits of third-party
  cancer datasets (a natural home for publicly scoring UCI/Kaggle/TCGA documentation quality). This
  project consumes the standard and feeds back teaching-specific fields.
- **(perpendicular) `bioinformatics-from-zero` / `oncology-data-literacy`** — learner-facing
  curricula that are the *primary adoption channel*. These projects are our beneficiaries: co-design
  datasets to their syllabi to convert `verifiedNeed: false → true`.
- **(spin-off) A "Teaching-Dataset Standard."** Generalize the template + 0–100 completeness score +
  health-aware datasheet + Croissant-RAI synthetic-provenance profile into a domain-agnostic standard
  for *any* small, safe, pedagogically-designed teaching dataset — a candidate contribution to
  MLCommons/Carpentries.
- **(spin-off) An MCP server** exposing the catalog: tools like `list_datasets(objective, difficulty)`,
  `get_datasheet(id)`, `get_exercises(id)`, `verify_provenance(id)`, `emit_croissant(id)`. Lets an
  educator's or learner's agent pull a license-clean, de-id-safe cancer teaching dataset on demand —
  high leverage and a natural showcase for the canonical-metadata-as-single-source-of-truth design.
- **(spin-off) An autograder service / Carpentries-style lesson pack** bundling the datasets into a
  course, satisfying M3's "teaching collection" exit criterion.

---

## 8. Open questions

1. **Sibling boundaries (highest priority).** What is the binding interface contract between
   `open-teaching-datasets`, `synthetic-cancer-data`, and `cancer-dataset-datasheets`? Without it,
   all three will build a generator and a datasheet tool.
2. **Synthetic calibration small-cell policy.** What is the minimum cell size / coarsening rule for
   *aggregate inputs* to synthetic generation, and what memorization test rejects an over-fitted
   generator?
3. **Statistical-fidelity tolerance.** What numeric tolerance defines "recovers the cited aggregate"
   for KM curves, stage distributions, expression marginals — and who sets it per dataset type?
4. **Copyright theory for calibration.** Is calibrating to a CC-BY aggregate legally a derivative
   work, and do we want to *assert* that or merely *attribute conservatively*?
5. **First adoption partner.** Which channel (Carpentries lesson repo, Galaxy Training, Bioconductor
   workshop, a named university course, an oncology-data-literacy program) confirms first, and does
   its audience require R as well as Python from M1?
6. **NC calibration-only policy.** Is calibration (no redistribution) from an NC cancer resource
   (COSMIC/OncoKB) ever acceptable, or blanket-EXCLUDE? (Plan defaults EXCLUDE; leaves a door open.)
7. **Verifiable-reuse definition.** What concrete evidence counts as an external reuse event for the
   outcome metric (citation? syllabus listing? fork-with-use? completion metric)?
8. **Competency taxonomy.** Which external skills framework do we map learning objectives to?

---

### Source list (deduplicated)

UCI WDBC: https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic ·
sklearn: https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_breast_cancer.html ,
https://scikit-learn.org/stable/datasets/toy_dataset.html ·
Kaggle medical-data failures: https://www.the-scientist.com/flawed-kaggle-datasets-used-in-clinical-models-spark-retractions-74564 ,
https://www.medrxiv.org/content/10.64898/2026.02.24.26347028v1 ·
Bioconductor: https://bioconductor.org/packages/release/data/experiment/html/curatedTCGAData.html ,
https://bioconductor.org/packages/release/bioc/html/RTCGA.html ·
Vanderbilt/hbiostat: https://hbiostat.org/data/ ·
OpenML: https://pmc.ncbi.nlm.nih.gov/articles/PMC12416095/ ·
Croissant: https://mlcommons.org/working-groups/data/croissant/ , https://mlcommons.org/2024/03/croissant_metadata_announce/ ·
Synthea / SEER synthetic: https://synthetichealth.github.io/synthea/ , https://github.com/lhs-open/synthetic-data ·
Synthetic-IPD / fidelity: https://arxiv.org/pdf/2509.16466 , https://arxiv.org/pdf/2410.16872 ·
Health datasheets: https://arxiv.org/html/2501.05617 , https://arxiv.org/pdf/2501.14094 , https://www.nature.com/articles/s41597-026-06670-0 , https://www.biorxiv.org/content/10.1101/2025.11.18.689064.full.pdf ·
Toy-dataset overuse: https://www.simonwenkel.com/2018/08/07/exploring-less-known-datasets-for-machine-learning.html
