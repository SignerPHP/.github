# Research Protocol (RFC Draft): Reproducible PDF Signing Qualification and Benchmarking for SignerPHP

**Status:** Proposal for maintainer consultation; not approved for implementation  
**Version:** 0.1 - discussion draft  
**Date:** 2026-10-10  
**Proposed audience:** SignerPHP maintainers and interested contributors  
**Requested architectural feedback:** `@jeidison`  
**Authority:** This document is a proposal. It does not authorize repository creation, issue publication, implementation, or changes to existing packages.

## 1. Executive summary

This proposal investigates a sustainable, reproducible qualification and benchmarking methodology for PDF parsing and digital signing in the SignerPHP ecosystem. The aim is to identify functional limitations, interoperability differences, regressions, and performance characteristics using traceable public PDF sources, controlled execution, and independent verification.

The current SignerPHP ecosystem has two distinct production libraries: [`signerphp/pdf-core`](https://github.com/SignerPHP/pdf-core) (PDF structures and incremental updates) and [`signerphp/pdf-signer`](https://github.com/SignerPHP/pdf-signer) (digital signing, CMS/PAdES, certificates, and provider orchestration). The historical [`signer-php`](https://github.com/SignerPHP/signer-php) repository is deprecated, and its maintainer has expressed a preference for retaining the current modular package names in [issue #28](https://github.com/SignerPHP/signer-php/issues/28). The [JSignPdf PHP wrapper](https://github.com/JSignPdf/jsignpdf-php) may serve as an external comparison reference; it is not the primary LibreSign signing engine.

**Key storage rule:** Original public PDF corpora remain with their upstream projects. SignerPHP would maintain acquisition methods and verification logic, *not* its own versioned copies or mirrors of upstream PDFs. Each experiment records lightweight provenance, exact input hashes, and immutable upstream revisions where available.

**Decision requested now:** Review the research questions, methodology, boundaries, proposed ownership, and possible repository structure. No issues, epics, new repositories, or implementation are requested at this stage.

## 2. Problem statement

Existing package-level unit, integration, and golden-contract tests are necessary, but they do not by themselves establish broad PDF interoperability, independently verified signing correctness, or comparable performance under a documented experimental protocol.

A number of practical uncertainties remain:

- Does the selected implementation handle PDFs from different generators, structural variants, incremental revisions, and malformed input cases as intended?
- Are outputs valid according to independent checks, beyond the implementation's own success return or verifier?
- Which differences are attributable to capabilities, inputs, algorithms, environment, or actual defects?
- Can performance claims be reproduced and interpreted without comparing different document populations or measurement boundaries?
- How can the evidence be retained without storing copied public PDF corpora or growing histories in production-library repositories?

The research must avoid assuming that agreement between multiple implementations establishes normative PDF/PAdES compliance.

## 3. Purpose and objectives

### General objective

Design and empirically validate a reproducible, independent method for functional qualification, differential analysis, regression detection, and performance measurement of PDF signing implementations in the SignerPHP ecosystem.

### Specific objectives

1. Define acquisition and input-provenance requirements without copying or versioning upstream public PDF corpora.
2. Define observable, versioned scenario, capability, attempt, artifact, validation, and measurement contracts.
3. Evaluate a controlled PHP-orchestrated execution model with sandboxing and modular adapters.
4. Define independent verification of PDF structure, signed-byte coverage, CMS cryptography, prior-signature preservation, and applicable profile requirements.
5. Establish a differential-analysis method distinguishing unsupported scenarios, expected rejections, execution defects, and independently invalid artifacts.
6. Establish a scientifically defensible protocol for process-level time, CPU, memory, and regression measurements; investigate optional PHP-only phase profiling separately.
7. Propose an auditable results-retention policy independent of both source-code repositories and upstream corpus storage.
8. Obtain maintainer feedback on the architecture and organizational fit *before* creating an implementation backlog.

## 4. Research questions

**RQ1 - Reproducibility and provenance:** Can selected public PDF inputs be reacquired and verified by upstream location, revision, path, retrieval date, and SHA-256 without maintaining an upstream PDF mirror?

**RQ2 - Functional qualification:** Which PDF structure, signing, and document-state categories are associated with supported behavior, expected rejection, functional failures, or independently invalid outputs?

**RQ3 - Independent verification:** How consistently can distinct verification methods determine applicable PDF structural, cryptographic, and signature-preservation properties, and where do their limitations or disagreements arise?

**RQ4 - Comparative performance:** For a shared population of *eligible and independently valid* scenarios, what are the distributions of complete-process duration, CPU time, and memory usage, under controlled execution conditions?

**RQ5 - Diagnostic and operational usefulness:** Can the evidence from a failed or divergent experiment be replayed against a different code revision and reliably distinguish fixed, persistent, and newly introduced regressions?

These are research questions, not preordained claims that any implementation is faster, more conformant, or more secure than another. A pilot study will inform feasible sample sizes and whether confirmatory hypotheses are warranted.

## 5. Scope and responsibility boundaries

**In scope:** PDF acquisition/provenance, structurally diverse document inputs, explicit signing scenarios, independent verification, isolated execution, regression analysis, process-level benchmarking, and optional diagnostic instrumentation for PHP libraries.

**Initial implementation candidates:** `SignerPHP/pdf-signer`, the deprecated `SignerPHP/signer-php` as a pinned legacy reference when feasible, and JSignPdf via `JSignPdf/jsignpdf-php` as an external process reference. Actual capability and environment compatibility must be checked before selecting scenarios. Additional implementations could be considered later via adapters.

**Library boundary:** `pdf-core` retains PDF parsing and incremental-serialization responsibilities; `pdf-signer` retains cryptographic signing responsibilities. The qualification method must not introduce production runtime dependencies in the opposite direction.

**External-reference boundary:** JSignPdf may be evaluated through the complete observable process, including child-process resource accounting as applicable. No internal JSignPdf/Java instrumentation, Java heap profiling, or implementation-stage comparison is proposed.

**Excluded from initial work:** LibreSign/Nextcloud operational throughput; runtime dependency on repositories in another organization; general PDF malware classification or mandatory machine learning; paid remote AI services; indefinite retention of generated PDFs; silent algorithm substitution; claims of comprehensive ISO/PAdES conformity based on a limited test suite.

## 6. Study design and experimental method

### 6.1 Study type

An experimental software-engineering study with reproducible inputs, documented environment, controlled independent variables, per-attempt machine-readable evidence, differential observations, and explicit threats-to-validity reporting.

The methodology has two connected but distinct analyses:

1. **Qualification study:** Does an implementation exhibit the outcome required by a scenario, according to independent evidence?
2. **Performance study:** For comparable eligible scenarios, what resource distributions are observed, with failures and excluded cases reported rather than hidden?

The performance study must not replace the qualification study, and neither should claim to prove universal standards compliance.

### 6.2 Population, sources, and sampling

Candidate upstream corpora include Mozilla PDF.js, veraPDF, Poppler, and appropriately licensed generated fixtures. These are research candidates, not automatic approvals for download, use, or redistribution.

Construct documented strata according to properties such as PDF version, classic/stream/hybrid cross-reference type, object streams, incremental revisions, prior signatures, form fields, encryption, corruption/malformed inputs, size, rendering/appearance, and applicable signing profiles. Categories may overlap.

Use a small pilot to verify source availability, stratification quality, analyzer support, run cost, and coverage gaps. Refine a representative main panel only after the pilot. Reserve selected cases for regression tests and make clear that a failure-enriched panel is not representative of overall compatibility rates.

**Do not version the original corpus locally.** Version only acquisition code, source configuration, schemas, and analysis logic. For each experiment, record the actual selected inputs and source revisions in its run-specific provenance and results records. Caches are temporary, ignored by Git, and reconstructible when upstream content remains available. Small original synthetic fixtures required for unit/CI testing may be stored or generated separately, with documented ownership.

### 6.3 Input provenance and licensing

For every input, record where available:

- Source identifier and canonical upstream URL;
- Immutable commit, release, or equivalent revision;
- Original path and immutable permalink;
- Retrieval time (UTC), SHA-256, and byte size;
- Multiple origins if the same content occurs at different sources;
- Source-project license, separately from the individual document's established license;
- SPDX license identifier/expression where evidence permits, or an explicit `NOASSERTION`/unresolved status;
- Licensing evidence URL and any restrictions on redistribution.

A Git repository's software license does not automatically establish the license of every contained PDF. PDF-internal creation/modification metadata must not be mistaken for upstream publication or acquisition dates.

If upstream content disappears, report the loss of reproducibility explicitly; never substitute another file silently under the same experiment identity.

### 6.4 Variables and controls

**Independent variables:** selected document; scenario; implementation and pinned revision; signing configuration (including explicit digest and signature algorithms); and specified runtime/environment parameters.

**Dependent observations:** capability outcome, operation result, validator evidence/verdicts, errors, artifact hashes, duration, CPU, peak process memory, and separately reported PHP-phase diagnostics.

**Controls:** recorded OS/container/CPU/PHP/Java versions, dependency lockfiles, isolation policy, temperature/cache/startup conditions where relevant, explicit warm-up, signing certificate fixtures, and local deterministic substitutes for external services when needed.

No automatic digest downgrades or other silent scenario changes are allowed. Non-equivalent operation configurations are distinct scenarios. Unsupported capabilities are not counted as functional implementation defects.

### 6.5 Execution isolation

Treat input PDFs as untrusted. Run each attempt with bounded time, memory, CPU, process count, files, and temporary storage; block unrestricted network egress and access to secrets; prevent path traversal and unsafe archive extraction; and terminate child processes on timeout. Prefer disposable execution environments and least-privilege access.

Public key/certificate test fixtures must not cause test credentials to be emitted in outputs. Network-dependent TSA/revocation behaviors require explicit opt-in and controlled fixtures/policies.

### 6.6 Independent evaluation and validity of outputs

Separate:

- Process execution success/failure;
- Scenario capability/eligibility;
- Structural PDF readability and revision integrity;
- ByteRange consistency and CMS cryptographic validity;
- Prior-signature preservation and applicable DocMDP constraints;
- Certificate trust/revocation and RFC3161 timestamp evidence where requested;
- Specific PAdES/PDF profile checks only when supported by an applicable verifier and normative basis;
- Visual/semantic preservation only in explicitly designed scenarios.

Each observation must identify the analyzer, version, policy, input/output artifact identity, verdict, and supporting evidence. Maintain distinct `valid`, `invalid`, `inconclusive`, `not_evaluated`, and `not_applicable` semantics where appropriate. Do not settle conflicting verifier results by majority vote. A scan for PDF tokens is only a diagnostic heuristic, not a normative validator.

### 6.7 Benchmark and statistical protocol

A paired comparison is valid only for a common population of scenarios for which both implementations are eligible, with configuration equivalence documented. Publish coverage, unsupported and failure rates for the *full* attempted population alongside any valid-output-only timing analysis to expose selection bias.

Collect process-level wall time, CPU time, and peak memory using defined measurement boundaries. If an adapter launches a Java process, the measurement policy must account for the relevant process tree. Distinguish RSS/process-group or cgroup metrics from PHP-managed allocation metrics. Mark unavailable metrics as unavailable, not zero.

Define repetitions, warm-up, execution order/counterbalancing, random seeds, and environmental controls based on pilot evidence. Preserve raw per-attempt observations. Report paired distributions (including medians and quantiles), appropriate uncertainty estimates, and the document as the unit of clustering when repeated measurements would otherwise create pseudo-replication. Avoid universal winner declarations based on unrepresentative samples.

Optional phase-level PHP diagnostics may use `hrtime()` and PHP memory APIs for independently selected profiling runs. Such instrumented data are not interchangeable with uninstrumented process-level comparisons, and no internal Java profiling is planned.

### 6.8 Differential and regression analysis

For each input and scenario, retain each implementation's individual attempts and independent validator observations. Classify differences as capability mismatch, expected rejection, reproducible implementation defect, validator disagreement, or unresolved finding as evidence supports.

A regression replay must identify exact document content, original scenario, implementation revisions, and changes in outcome. Maintain distinct representative qualification panels and failure-enriched replay selections.

### 6.9 Ethics, security, and research integrity

Do not store raw public corpora, private keys, signing passwords, real personal data, or unredacted sensitive PDF content in public result repositories. Record redacted diagnostics and hashes instead of attaching sensitive failed documents. Use safe disclosure paths for newly discovered exploitable issues. PDF JavaScript/actions, embedded files, unusual decompression, and metadata may be collected as bounded observations, not automatically assigned a malware verdict.

No external AI service or local ML classifier is needed for the initial research verdict. If machine learning is considered later, it will be a separate, evidence-backed proposal with data rights, false-positive evaluation, reproducibility, and baseline comparisons.

## 7. Analysis outputs and data retention

The planned output classes are:

1. **Research protocol/RFC:** problem, questions, method, assumptions, decisions and revisions.
2. **Tooling and schemas:** acquisition configuration, adapters, runners, validators and analysis source.
3. **Run records:** compact, immutable references to upstream input hashes and revisions; scenario and implementation identities; environment, raw attempts, verification evidence and metrics.
4. **Derived reports:** coverage, divergences, regression summaries, statistical comparisons, confidence/uncertainty notes, and known limitations.

**Provisional ownership:** a separate SignerPHP qualification-tools repository is one candidate, not a decided repository creation. A distinct SignerPHP historical-results repository may be justified only when there is demonstrated retention need, an agreed publishing contract, and maintainer approval. Source PDF binaries remain upstream or in temporary local caches. An existing SignerPHP repository is another ownership alternative for the RFC itself.

Do not create a growing measurement history in the `pdf-signer` or `pdf-core` code repositories. Do not introduce an operational dependency on repositories belonging to LibreSign or other organizations.

## 8. Study stages and review gates

| Stage | Purpose | Evidence required to proceed |
| --- | --- | --- |
| **0 - Maintainer consultation** | Review the problem, method, organizational fit, ownership and naming | Maintainer feedback and documented decisions; no implementation authorization implied |
| **1 - Research contract pilot** | Prototype acquisition provenance, schema and representative sample | Reproducibility checks, hash checks, explicit licensing state, basic schema tests |
| **2 - Qualification pilot** | Trial a minimal isolated runner and independent verification | Verified positive/negative cases, failures preserved, measurable isolation properties |
| **3 - Differential baseline** | Establish comparisons and targeted regression replay | Complete denominators, conflicting observations preserved, limits documented |
| **4 - Performance study** | Run paired measurements on independently valid comparable cases | Repeatability, environment identity, statistical protocol, separate PHP diagnostics |
| **5 - Historical publication** | Decide whether and how to persist results separately | Demonstrated value, size/privacy policy, reviewed publication contract and authorized repository |

These stages are a research map, not a frozen issue hierarchy. After stage 0, decompose approved work into GitHub epics and independently verifiable issues under the official `create-issues-and-epics` skill. Before developing later stages, update their assumptions from the actual prior-stage artifacts.

## 9. Threats to validity and limitations

- **Selection bias:** public upstream corpora may overrepresent known parser edge cases or underrepresent production workloads.
- **Construct validity:** successful signing, cryptographic signature validity, certificate trust, document preservation, and normative conformity are different outcomes.
- **Tool validity:** independent validators can be incomplete or disagree, and heuristic detectors can be fooled.
- **Internal validity:** runtime setup, JVM startup, warm-up, filesystem cache, CPU contention, and instrumentation overhead can influence measurements.
- **Statistical validity:** repeated observations on a document are not independent documents; unequal success populations bias headline speed averages.
- **External validity:** results from controlled PDF fixtures do not establish actual LibreSign/Nextcloud workload throughput or performance on every document in the wild.
- **Reproducibility:** an upstream source may delete or alter access to historical artifacts; a hash confirms identity but cannot guarantee future availability.
- **Legal and privacy limits:** licensing and personal-data restrictions may prohibit retaining or distributing specific artifacts or extracted content.

## 10. Architectural alternatives for maintainer review

**Option A - Dedicated SignerPHP qualification repository (working preference):** isolates tooling from production packages and maintains a neutral adapter boundary. Costs include maintaining a new repository and CI.

**Option B - Place the research tools inside an existing signing-library repository:** simpler initial administration but risks coupling comparisons to one implementation and growing the code repository with non-product tooling.

**Option C - Keep only the RFC under `SignerPHP/.github` and defer infrastructure ownership:** useful for discussion; implementation repository creation follows approval and pilot evidence.

A separate *results* repository is a later decision, not a prerequisite for discussing the RFC or conducting a small local pilot.

## 11. Questions for `@jeidison`

1. Does independent PDF qualification fit SignerPHP's modular ecosystem, or should part of it live closer to the existing packages?
2. Would a dedicated repository be appropriate, and what name would best communicate its responsibility?
3. Should the qualification research begin with a smaller subset of operations or implementation targets?
4. Which existing tests, tooling interfaces, corpus sources or PDF edge cases should be reused before building additional infrastructure?
5. Is the separation of code, upstream PDF sources, and optional historical-results storage proportionate to the ecosystem's maintenance capacity?
6. Are there changes you would recommend to the research questions, validity criteria, or implementation gates?

Feedback is requested on this *proposal*, not on a preauthorized backlog. Assigning an implementation owner, creating a repository, and publishing implementation issues are separate decisions.

## 12. Decision register

| Decision | Current status | Rationale / next action |
| --- | --- | --- |
| Public PDF corpora remain upstream; no SignerPHP PDF mirror | Established requirement | Version acquisition logic; record per-run provenance and hashes |
| Preserve current modular repository/package naming | Maintainer preference expressed in `signer-php#28` | Do not propose renames within this RFC |
| Separate signing-library code from qualification tooling | Proposed | Confirm repository ownership with maintainers |
| Separate growing results history from production repositories | Established architectural direction | Decide physical repository and publication only after pilot |
| Use JSignPdf only through observable end-to-end process behavior | Established requirement | No internal Java metrics or modifications |
| Use current PHP implementations for optional phase diagnostics | Proposed optional capability | Validate cost/benefit after basic process measurements |
| Establish valid/invalid verdicts via independent evidence | Established methodological requirement | Record verifier limits and uncertainty |
| Prioritize machine learning or external AI | Not proposed | Revisit only after deterministic methods are working |
| Create implementation epics, issues or repositories | **Not authorized** | Await maintainer consultation and explicit human approval |

## 13. Supporting material for further examination

- SignerPHP `pdf-signer`: https://github.com/SignerPHP/pdf-signer
- SignerPHP `pdf-core`: https://github.com/SignerPHP/pdf-core
- Legacy naming/deprecation discussion: https://github.com/SignerPHP/signer-php/issues/28
- Historical hybrid-XRef issue: https://github.com/SignerPHP/signer-php/issues/24
- JSignPdf PHP wrapper: https://github.com/JSignPdf/jsignpdf-php
- Example upstream sources for suitability/licensing investigation: https://github.com/mozilla/pdf.js ; https://github.com/veraPDF/veraPDF-corpus ; https://gitlab.freedesktop.org/poppler/test
- PDF standard context: https://pdfa.org/resource/iso-32000-2/
- PAdES baseline specification context: https://www.etsi.org/deliver/etsi_en/319100_319199/31914201/01.02.01_60/en_31914201v010201p.pdf
- Performance experiment methodology: Kalibera and Jones, *Rigorous Benchmarking in Reasonable Time*, ISMM 2013: https://kar.kent.ac.uk/33611/45/p63-kaliber.pdf
- Signed-PDF attack research: Mainka et al., *Shadow Attacks: Hiding and Replacing Content in Signed PDFs*, NDSS 2021: https://www.ndss-symposium.org/ndss-paper/shadow-attacks-hiding-and-replacing-content-in-signed-pdfs/

*References are starting points for methodological review, not declarations that all features, corpus files, or normative profiles are supported.*

## 14. Change log

- **0.1 - 2026-10-10:** Initial consultation draft consolidating the research problem, scientific methodology, no-corpus-mirroring policy, result boundaries, experimental validity, and maintainer decision points.
