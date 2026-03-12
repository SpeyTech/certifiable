# certifiable-* Certification Guide

**Version:** 1.0.0  
**Date:** 12 March 2026  
**Owner:** SpeyTech / The Murray Family Innovation Trust  
**Contact:** william@fstopify.com  
**Patent:** UK GB2521625.0 (Murray Deterministic Computing Platform)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [How certifiable-* Fits Into a Certification Programme](#2-how-certifiable--fits-into-a-certification-programme)
3. [The Fundamental Question](#3-the-fundamental-question)
4. [DO-178C Alignment](#4-do-178c-alignment-aerospace)
5. [IEC 62304 Alignment](#5-iec-62304-alignment-medical-devices)
6. [ISO 26262 Alignment](#6-iso-26262-alignment-automotive)
7. [Cryptographic Artefact Map](#7-cryptographic-artefact-map)
8. [The Three Theorems — Certification Relevance](#8-the-three-theorems--certification-relevance)
9. [Document Inventory](#9-document-inventory)
10. [What certifiable-* Does Not Cover](#10-what-certifiable--does-not-cover)
11. [Contact and Commercial Licensing](#11-contact-and-commercial-licensing)

---

## 1. Introduction

`certifiable-*` is a family of ten open-source repositories implementing a complete, deterministic machine learning pipeline in pure C99. Every component — data loading, training, quantization, deployment, inference, and runtime monitoring — produces cryptographic commitments that bind outputs to inputs across platforms and across time. The pipeline is designed from first principles to support formal certification under aerospace, medical device, and automotive safety standards.

This document covers the certification evidence available from `certifiable-*`: which standard objectives each artefact satisfies, which repository produces each artefact, and which specification document defines it. The artefacts described here provide objective evidence that may support certification activities, but certification approval remains the responsibility of the applicant (integrator) and the relevant authority. For project overview, architecture, and quick-start instructions, see the parent repository README at `https://github.com/SpeyTech/certifiable`.

**This document does not cover:**

- Implementation guidance or integration instructions
- Tool qualification evidence (DO-330 / ISO 26262 Part 8 clause 11)
- System safety assessment or hazard analysis
- Hardware design assurance (DO-254)
- Quality management system requirements
- Risk management processes

These responsibilities remain with the integrator's safety process. `certifiable-*` provides software-layer evidence only. Gaps are stated explicitly throughout this document — they are a credibility signal, not an omission.

---

## 2. How certifiable-* Fits Into a Certification Programme

`certifiable-*` is not a certification framework and does not replace the integrator's certification process. It provides deterministic computation and cryptographic traceability artefacts that can be incorporated into an existing certification programme.

In a typical safety-critical development lifecycle, `certifiable-*` occupies the software computation layer within the system architecture:

```
System Safety Case
│
├─ System requirements and hazard analysis
│   (ARP4761 / ISO 26262 Part 3 / ISO 14971)
│
├─ Hardware platform assurance
│   (DO-254 / ISO 26262 Parts 4–5)
│
├─ Software certification programme
│   (DO-178C / IEC 62304 / ISO 26262 Part 6)
│
│   └─ ML computation component
│        (implemented using certifiable-*)
│
└─ Integration, verification, and certification data package
```

Within this structure, the artefacts produced by `certifiable-*` support several certification activities:

| Certification Activity | certifiable-* Artefact |
|------------------------|------------------------|
| Software configuration identification | Commitment chain (`M_data → H_train → H_cert → R`) |
| Independent verification | `certifiable-verify` replay verification report |
| Numerical correctness evidence | Quantization certificate `H_cert` |
| Configuration integrity | Release bundle root `R` |
| Runtime traceability | Monitor audit chain `H_audit` |

These artefacts may be incorporated into the integrator's certification data package as supporting evidence for software verification, configuration management, and traceability objectives.

The integrator remains responsible for: certification planning documents, system safety analysis, tool qualification, regulatory interface, and final certification submission.

The `certifiable-*` artefacts therefore function as deterministic computation evidence within the broader certification data package.

---

## 3. The Fundamental Question

Every certification assessor, regardless of standard, asks one question:

**Can you prove that what you deployed is exactly what you certified?**

The conventional answer is a configuration management record and a test report. The certifiable-* answer is a cryptographic chain of custody — a sequence of SHA-256 commitments that links every output to every input, across every stage of the pipeline, verifiable by independent replay.

The chain is constructed as follows:

```
DATA ─────────► TRAINING ─────────► QUANT ─────────► DEPLOY ─────────► INFERENCE
  │                 │                  │                 │                   │
  ▼                 ▼                  ▼                 ▼                   ▼
M_data ────────► H_train ────────► H_cert ──────────► R ────────────► H_pred
  │                 │                  │                 │                   │
  └─────────────────┴──────────────────┴─────────────────┴───────────────────┘
                                       │
                    CROSS-ARTEFACT COMMITMENT BINDINGS
                                       │
                                       ▼
                               VERIFICATION REPORT
                              (certifiable-verify)
                                       │
                                       ▼
                               H_audit (monitor chain)
```

Each identifier in this chain is defined in Section 6. Each commitment is produced by a specific repository, specified in a MATH-001 document, and independently verifiable by `certifiable-verify`.

The answer to the fundamental question is: yes. Given the artefact bundle for any release, `certifiable-verify` can deterministically replay the entire pipeline and confirm bit-identical reproduction of every commitment — on any DVM-compliant platform, months or years after the original certification run.

Conventional ML frameworks do not provide this capability. Proof is not asserted; it is computed.

---

## 4. DO-178C Alignment (Aerospace)

**Target level:** DAL-A (most stringent)

The following table maps DO-178C objectives (referenced to Table A positions in DO-178C Annex A) to the certifiable-* artefact that satisfies each objective. For each objective, the table identifies what `certifiable-*` provides, which repository produces the artefact, and which specification document governs it.

Explicit gaps are stated at the end of this section.

The mapping below identifies the certifiable-* artefacts that provide objective evidence. These artefacts support compliance but do not replace the integrator's certification data package.

### Artefact Naming Convention

Artefact identifiers appear consistently across specification documents, repositories, and verification output:

- `M_*` — Merkle commitments
- `H_*` — chain hashes
- `R` — release bundle commitment

---

### DO-178C Objective Mapping

| Objective | DO-178C Reference | What certifiable-* Provides | Repository | Document |
|-----------|-------------------|-----------------------------|------------|----------|
| **SW-001** Software development plan | Table A-2 | MATH-001 documents define the mathematical specification that governs all code. SRS documents define requirements with SHALL statements and verification methods. The DVM specification defines the sole legal execution semantics. | All repos | `{PREFIX}-MATH-001`, SRS series |
| **SW-011** Software requirements | Table A-3 | Each repository contains a requirements/SRS-00N series. Requirements are stated as SHALL statements, each with a verification method (test, analysis, or inspection). Traceability tags (`@traceability`) link source code to requirements via pre-commit enforcement. | All repos | SRS-001 through SRS-00N per repo |
| **SW-021** Software design | Table A-4 | STRUCT-001 documents specify every data structure as a direct derivation from MATH-001. Code is a literal transcription of the mathematical specification — design and implementation are the same artefact, with the math as the authority. | All repos | `{PREFIX}-STRUCT-001` |
| **SW-031** Software coding standards | Table A-5 | All source code targets C99. MISRA-C 2012 compliance is enforced via cppcheck in pre-commit hooks across all repositories. Strict compiler flags (`-Wall -Wextra -Wpedantic -Werror -Wshadow -Wconversion -fno-fast-math`) are set in CMakeLists.txt. | All repos | `.pre-commit-config.yaml`, `CMakeLists.txt` |
| **SW-041** Integration | Table A-6 | `certifiable-harness` provides end-to-end cross-platform integration tests across all seven pipeline stages. `certifiable-verify` provides the integration verification report binding all stage hashes. | `certifiable-harness`, `certifiable-verify` | CV-MATH-001, SRS-007-BINDING |
| **SW-051** Verification of software requirements | Table A-7 | 13,000+ assertions across the ecosystem. Each test suite maps to SRS requirements via traceability tags. Bit-identity tests verify cross-platform reproducibility on x86, ARM, and RISC-V. | All repos | Per-repo SRS series |
| **SW-061** Verification of software design | Table A-7 | STRUCT-001 documents are verified against MATH-001 by inspection. Pre-commit traceability checks verify that all source files reference their governing specification section. | All repos | `{PREFIX}-STRUCT-001` |
| **SW-071** Verification of coding and integration | Table A-7 | Static analysis via cppcheck runs on every commit. Integration verification via `certifiable-verify --full-replay` re-executes the complete pipeline and compares all commitments bit-for-bit. The verification report (JSON) is itself committed to the audit chain. | All repos, `certifiable-verify` | CV-MATH-001, SRS-008-REPORT |
| **SW-082** Independence of verification | Table A-7 | `certifiable-verify` is an independent repository with no build dependency on any other certifiable-* component. It consumes only the artefact bundle — not source code — and verifies all commitments without access to the original computation context. | `certifiable-verify` | CV-MATH-001 |
| **SW-091** Configuration management | Table A-8 | Every pipeline output is committed cryptographically. `M_data` commits the dataset. `H_train` commits the training run. `R` commits the deployment bundle. Any modification to any input changes every downstream hash, making configuration drift detectable by inspection of the commitment chain. | `certifiable-data`, `certifiable-training`, `certifiable-deploy`, `certifiable-verify` | CD-MATH-001, CT-MATH-001, CDP-MATH-001, CV-MATH-001 |
| **SW-111** Software quality assurance | Table A-9 | Pre-commit hooks enforce copyright headers, traceability tags, MISRA-C compliance, and YAML validity on every commit. CI (Brian Shirai's infrastructure) runs the full test suite across 50+ platform/compiler combinations. All test results are recorded in the commitment chain. | All repos | `.pre-commit-config.yaml`, CI configuration |

### Explicit Gaps — DO-178C

The following DO-178C requirements are outside `certifiable-*` scope and SHALL be addressed by the integrator:

- **Tool qualification (DO-330):** gcc, cppcheck, cmake, and SHA-256 implementations used by certifiable-* are not accompanied by tool qualification evidence per DO-330. The integrator must qualify or justify these tools independently.
- **System safety assessment (ARP4761):** certifiable-* does not perform system-level hazard analysis, fault tree analysis, or FMEA. These are integrator responsibilities.
- **Planning documents:** DO-178C Table A-1 planning objectives (software development plan, software verification plan, software configuration management plan, software quality assurance plan) are not provided as standalone documents. The MATH-001 and SRS series provide technical content but not the planning framework.
- **Airworthiness authority coordination:** certifiable-* does not interact with FAA, EASA, or other airworthiness authorities. That relationship is the integrator's responsibility.

---

## 5. IEC 62304 Alignment (Medical Devices)

**Target class:** Class C (highest risk — malfunction may cause death or serious injury)

The following table maps IEC 62304 clauses to `certifiable-*` artefacts. IEC 62304 Class C requires a complete software development lifecycle with full traceability from requirements through testing.

### IEC 62304 Clause Mapping

| IEC 62304 Clause | Requirement | What certifiable-* Provides | Repository | Document |
|------------------|-------------|-------------------------------|------------|----------|
| **5.1** Software development planning | Written plan governing development activities | MATH-001 documents establish the mathematical contract governing all implementation. The DVM specification is the normative development authority. SRS documents define the requirements baseline. | All repos | `{PREFIX}-MATH-001`, SRS series |
| **5.2** Software requirements analysis | Documented, verifiable requirements | SRS-00N documents per repository contain SHALL statements, rationale, and verification methods. Requirements reference MATH-001 sections for mathematical grounding. | All repos | SRS series per repo |
| **5.3** Software architectural design | Documented architecture showing system decomposition | The pipeline architecture is a strict directed acyclic graph: data → training → quant → deploy → inference → monitor. Each stage is an independent repository with defined inputs and outputs. STRUCT-001 documents define inter-stage data structures. | All repos | `{PREFIX}-STRUCT-001` |
| **5.4** Software detailed design | Design documentation for each software unit | STRUCT-001 documents specify every data structure. MATH-001 documents specify every algorithm. Source code is a literal transcription with `@traceability` tags linking each function to its governing specification. | All repos | `{PREFIX}-MATH-001`, `{PREFIX}-STRUCT-001` |
| **5.5** Software unit implementation and verification | Implementation and unit test evidence | Each repository provides unit tests with full coverage of DVM primitives, domain logic, and bit-identity verification. Test vectors are drawn directly from MATH-001 specification sections (e.g., CT-MATH-001 §7.2 Feistel test vectors). | All repos | Per-repo SRS, MATH-001 test vectors |
| **5.6** Software integration and integration testing | Integration of units and verification of integration | `certifiable-harness` provides end-to-end integration across all pipeline stages. `certifiable-verify` verifies cross-stage commitment bindings (SRS-007-BINDING). | `certifiable-harness`, `certifiable-verify` | CV-MATH-001, SRS-007-BINDING |
| **5.7** Software system testing | System-level testing against requirements | `certifiable-verify --full-replay` re-executes the complete pipeline from data loading through monitoring and confirms all commitments match. The verification report (SRS-008-REPORT) is the system test evidence artefact. | `certifiable-verify` | SRS-008-REPORT |
| **5.8** Software release | Release procedure with configuration baseline | `certifiable-deploy` produces a Canonical Bundle Format (CBF v1) release bundle with a 4-leaf Merkle attestation root `R`. The release bundle is target-bound, tamper-evident, and offline-verifiable. `R` is the release commitment. | `certifiable-deploy` | CDP-MATH-001 |
| **6.1** Software maintenance plan | Plan for post-release changes | Maintenance is governed by the commitment chain: any change to source data, weights, or code changes `M_data`, `H_train`, or `H_cert`, which changes `R`. Unchanged commitments prove unchanged artefacts. | `certifiable-verify` | CV-MATH-001 |
| **8.1** Configuration management | Identification and control of software items | Every pipeline stage commits its output cryptographically. The full commitment chain (`M_data → H_train → H_cert → R → H_pred → H_audit`) constitutes a cryptographic configuration baseline. certifiable-verify validates the baseline on demand. | All repos | CV-MATH-001 |
| **9.1** Problem resolution process | Process for identifying and resolving software problems | `certifiable-monitor` detects runtime anomalies (input drift, activation anomalies, output drift) and logs each event to the audit ledger `H_audit` via SHA-256 hash chain. The health FSM transitions to defined alarm states. All events are verifiable offline. | `certifiable-monitor` | CM-MATH-001, SRS-006-LEDGER |

### Explicit Gaps — IEC 62304

The following IEC 62304 requirements are outside `certifiable-*` scope:

- **Quality Management System (ISO 13485 / ISO 9001):** certifiable-* does not provide a QMS. The integrator's QMS governs development process compliance.
- **Risk management (ISO 14971):** certifiable-* does not perform hazard identification, risk estimation, or risk control decisions. These are integrator responsibilities under ISO 14971.
- **Regulatory submission:** certifiable-* does not prepare 510(k), CE marking, or other regulatory filings. The integrator manages regulatory interface.
- **Clinical evaluation:** out of scope.

---

## 6. ISO 26262 Alignment (Automotive)

**Target level:** ASIL-D (highest)

The following table maps ISO 26262 Part 6 software requirements to `certifiable-*` artefacts.

### ISO 26262 Part 6 Requirement Mapping

| ISO 26262 Requirement | Part 6 Reference | What certifiable-* Provides | Repository | Document |
|-----------------------|------------------|-----------------------------|------------|----------|
| **Software safety requirements specification** | Clause 6 | SRS documents per repository state software safety requirements as SHALL statements with rationale derived from MATH-001. Each requirement identifies its verification method. | All repos | SRS series |
| **Software architectural design** | Clause 7 | Pipeline is a strictly layered directed graph with no feedback paths between stages. STRUCT-001 documents define all inter-layer data structures. Each layer has a defined and bounded interface. | All repos | `{PREFIX}-STRUCT-001` |
| **Software unit design and implementation** | Clause 8 | All units are implemented in pure C99 with zero dynamic allocation after initialization. Every arithmetic operation uses DVM primitives (widening, saturation, round-to-nearest-even). No undefined behaviour is permitted; compiler warnings as errors. | All repos | `{PREFIX}-MATH-001`, `CMakeLists.txt` |
| **Software unit verification** | Clause 9 | Unit tests cover all DVM primitives, all domain modules, and bit-identity verification. Test vectors are drawn from MATH-001 specification sections and are deterministic — the same test run produces the same result on every platform. | All repos | MATH-001 test vectors, SRS series |
| **Software integration and verification** | Clause 10 | `certifiable-harness` integrates all seven pipeline stages and verifies cross-stage commitment bindings. Integration verification is deterministic and repeatable. | `certifiable-harness`, `certifiable-verify` | CV-MATH-001 |
| **MISRA-C compliance** | Clause 8.4.4 | cppcheck MISRA-C 2012 analysis runs via pre-commit hook on all `.c` and `.h` files. Violations fail the commit. | All repos | `.pre-commit-config.yaml` |
| **Static analysis** | Clause 9.4.3 | cppcheck runs with `--enable=all --error-exitcode=1`. Strict compiler flags (`-Wall -Wextra -Wpedantic -Werror`) apply to all builds. | All repos | `CMakeLists.txt` |
| **Structural coverage (MC/DC)** | Clause 9.4.4 (ASIL-D) | Test suites exercise all branches of DVM primitive operations including fault paths (overflow, underflow, division by zero, domain error). Fault injection tests verify fault propagation. Formal MC/DC coverage measurement tooling is not currently provided — see explicit gaps below. | All repos | Per-repo test suites |
| **Worst-case execution time (WCET)** | Clause 7.4.14 | certifiable-inference includes timing verification tests proving bounded execution time with less than 5% observed jitter at the 95th percentile under test conditions. Zero dynamic allocation guarantees O(1) memory. All loops are bounded at compile time. Formal WCET analysis tooling (e.g., AbsInt aiT) is not provided — see explicit gaps below. | `certifiable-inference` | CI-MATH-001 |

### Explicit Gaps — ISO 26262

The following ISO 26262 requirements are outside `certifiable-*` scope:

- **HARA (Hazard Analysis and Risk Assessment, Part 3):** certifiable-* does not perform hazard identification, ASIL assignment, or safety goal derivation. These are integrator responsibilities.
- **Functional safety concept (Part 4):** certifiable-* does not define functional safety requirements at the system level.
- **Hardware-software interface (HSI) specification (Part 6 clause 5):** certifiable-* does not specify the hardware environment beyond platform tuple binding in `certifiable-deploy`. The integrator must provide HSI documentation.
- **Formal WCET analysis:** certifiable-* provides timing verification tests but does not provide a WCET certificate from a qualified tool such as AbsInt aiT or Rapita RVS. The integrator must perform formal WCET analysis on their target hardware.
- **Formal MC/DC coverage measurement:** certifiable-* provides branch-covering test suites but does not provide MC/DC coverage reports from a qualified coverage tool. The integrator must instrument and measure MC/DC coverage on their toolchain.
- **Tool qualification (Part 8 clause 11):** not provided.
- **Safety case and functional safety assessment (Part 2):** not provided.

---

## 7. Cryptographic Artefact Map

The cryptographic artefact map below is the central evidence structure of `certifiable-*`, defining the verifiable commitments that form the pipeline's chain of custody.

Artefact identifiers follow this convention throughout specifications, repositories, and verification output:

- `M_*` — Merkle commitments (tree roots)
- `H_*` — chain hashes (sequential SHA-256 chains)
- `R` — release bundle attestation root

The following table is the canonical map of all cryptographic artefacts in the `certifiable-*` pipeline.

| Artefact | Commits | Produced by | Specified in | Verified by |
|----------|---------|-------------|--------------|-------------|
| `M_data` | Merkle root of the complete dataset; commits all samples in loading order | `certifiable-data` | CD-MATH-001 §6 (Merkle provenance construction) | `certifiable-verify` (SRS-001-PROVENANCE) |
| `H_train` | Training chain hash; commits all weight tensors at every training step across all epochs | `certifiable-training` | CT-MATH-001 §9 (Merkle audit chain) | `certifiable-verify` (SRS-002-TRAINING) |
| `H_cert` | Quantization certificate; commits analysis, calibration, conversion, and verification digests for FP32→Q16.16 conversion | `certifiable-quant` | CQ-MATH-001 §8 (certificate construction) | `certifiable-verify` (SRS-003-QUANT) |
| `R` | Release bundle attestation root; 4-leaf Merkle tree binding manifest, weights, certificates, and inference artefacts | `certifiable-deploy` | CDP-MATH-001 §5 (CBF v1 attestation) | `certifiable-verify` (SRS-004-BUNDLE) |
| `H_pred` | Inference output hash; SHA-256 commitment over all predictions for a defined input set | `certifiable-inference` | CI-MATH-001 §7 (output commitment) | `certifiable-verify` (SRS-005-INFERENCE) |
| `H_audit` | Monitor chain hash; SHA-256 hash chain over all runtime monitoring events in sequence | `certifiable-monitor` | CM-MATH-001 §6 (audit ledger chain) | `certifiable-verify` (SRS-006-LEDGER) |

Cross-artefact bindings — the cryptographic links asserting that `M_data` was the input to the training run that produced `H_train`, and so on — are specified in `certifiable-verify` SRS-007-BINDING and verified by `certifiable-verify --artifacts`.

The complete verification report, including all stage results and binding results, is itself committed as a JSON artefact with a report hash. This report is the single deliverable presented to an assessor as evidence of pipeline integrity. It is specified in CV-MATH-001 and SRS-008-REPORT.

---

## 8. The Three Theorems — Certification Relevance

`certifiable-*` is built on three formal theorems. Each theorem maps directly to one or more certification requirements across DO-178C, IEC 62304, and ISO 26262.

### Theorem 1 — Bit Identity

**Statement:** For any two DVM-compliant platforms A and B and any input state s, the pipeline function F satisfies F\_A(s) = F\_B(s).

**Proved by:** The DVM specification constrains all arithmetic to integer operations with explicit widening, saturation at defined bounds, and round-to-nearest-even. These properties are platform-independent by construction. Cross-platform bit-identity tests across x86, ARM, and RISC-V platforms verify this property empirically.

**Certification relevance:**

| Standard | Objective | How Bit Identity Satisfies It |
|----------|-----------|-------------------------------|
| DO-178C | SW-082 Independence of verification | A verifier on any platform produces the same result as the original certifier. Independence is genuine — not a claim. |
| DO-178C | SW-071 Verification of coding/integration | Replay on a different platform is a valid independent verification act. |
| IEC 62304 | 5.6 Integration testing | Integration tests are repeatable on any DVM-compliant platform. |
| ISO 26262 | Part 6 clause 9 Unit verification | Test results are deterministic; the same test run produces the same pass/fail verdict everywhere. |

### Theorem 2 — Bounded Error

**Statement:** Quantization error for any weight w is bounded: |Q(w) − w| ≤ ε, where ε is the Q16.16 unit of least precision (ULP = 2^{-16} ≈ 1.53e-5). Errors saturate at defined bounds; they do not accumulate silently.

**Proved by:** `certifiable-quant` computes theoretical error bounds before quantization (operator norm analysis, range propagation, overflow proofs) and then verifies empirically that all converted weights are within the certified bounds. The quantization certificate `H_cert` commits both the theoretical bound and the empirical verification result. The DVM fault model propagates overflow flags through the computation; there are no silent failures.

**Certification relevance:**

| Standard | Objective | How Bounded Error Satisfies It |
|----------|-----------|-------------------------------|
| DO-178C | SW-051 Verification of requirements | Worst-case behaviour is analysable; the error bound is a formal property, not a statistical estimate. |
| ISO 26262 | WCET / worst-case behaviour (clause 7.4.14) | Saturation bounds are fixed. No error term grows without bound. |
| IEC 62304 | 5.5 Unit verification | Each unit's fault behaviour is specified and tested. |
| All | Formal error bounds | The quantization certificate `H_cert` is the evidence artefact for numerical correctness claims. |

### Theorem 3 — Auditability

**Statement:** Any operation in the pipeline is verifiable in O(1) time given the commitment chain and the artefact bundle.

**Proved by:** SHA-256 Merkle trees provide O(log N) proof paths for dataset samples and training steps; SHA-256 hash chains provide O(1) verification of any chain link. `certifiable-verify` implements both modes: hash-only verification (seconds) and full deterministic replay (minutes). Both produce the same verification report.

**Certification relevance:**

| Standard | Objective | How Auditability Satisfies It |
|----------|-----------|-------------------------------|
| DO-178C | SW-091 Configuration management | Any release can be independently verified at any time after the fact. Configuration drift is detectable by commitment comparison. |
| IEC 62304 | 8.1 Configuration management | Cryptographic configuration baseline is verifiable on demand. |
| IEC 62304 | 5.8 Software release | The release artefact `R` is independently verifiable offline. |
| ISO 26262 | Part 6 clause 10 Integration verification | Cross-stage bindings are cryptographically verifiable, not asserted. |
| All | Traceability | Every output traces to every input via the commitment chain. The chain does not require source code to verify. |

Together, these three theorems establish the deterministic and verifiable behaviour of the `certifiable-*` pipeline. The commitment chain guarantees artefact identity, bounded error guarantees numerical stability, and auditability guarantees traceability. These properties form the technical basis upon which certification evidence for ML computation can be constructed.

---

## 9. Document Inventory

The following specification documents exist in the `certifiable-*` ecosystem. Each document is a normative authority; code is derived from these documents, not the reverse.

These documents constitute the normative specification set for the `certifiable-*` ecosystem. Source code implementations SHALL conform to these specifications.

### certifiable-data

| Document ID | File | Contents |
|-------------|------|----------|
| CD-MATH-001 | `docs/CD-MATH-001.md` | Mathematical specification: Q16.16 fixed-point formats, Feistel shuffle bijection (CT-MATH-001 §7.2 test vectors), counter-based PRNG, Merkle provenance construction, SHA-256 domain separation |
| CD-STRUCT-001 | `docs/CD-STRUCT-001.md` | Data structure specification: `ct_sample_t`, `ct_dataset_t`, `ct_batch_t`, `ct_provenance_t`, `ct_fault_flags_t` |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: data loading, normalisation, augmentation, shuffle, batch construction, Merkle chain |

Standards cross-reference: CD-MATH-001 is cited by DO-178C SW-091, IEC 62304 8.1, ISO 26262 Part 6 clause 8.

### certifiable-training

| Document ID | File | Contents |
|-------------|------|----------|
| CT-MATH-001 | `docs/CT-MATH-001.md` | Mathematical specification: Q16.16 weights, Q8.24 gradients, Q32.32 accumulators, Neumaier compensated summation, fixed-topology reduction, Merkle audit chain, Feistel data permutation |
| CT-STRUCT-001 | `docs/CT-STRUCT-001.md` | Data structure specification: weight tensors, gradient tensors, training state, checkpoint structure |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: forward pass, backward pass, optimisers (SGD/Momentum/Adam), Merkle chain, bit-identity |

### certifiable-quant

| Document ID | File | Contents |
|-------------|------|----------|
| CQ-MATH-001 | `docs/CQ-MATH-001.md` | Mathematical specification: theoretical error analysis (operator norms, range propagation, overflow proofs), calibration statistics, FP32→Q16.16 conversion, verification bounds, Merkle certificate construction |
| CQ-STRUCT-001 | `docs/CQ-STRUCT-001.md` | Data structure specification: analysis result, calibration result, conversion result, certificate |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: analyse, calibrate, convert, verify, certify |

Standards cross-reference: CQ-MATH-001 §8 is the normative specification for `H_cert`.

### certifiable-deploy

| Document ID | File | Contents |
|-------------|------|----------|
| CDP-MATH-001 | `docs/CDP-MATH-001.md` | Mathematical specification: CBF v1 bundle format, 4-leaf Merkle attestation (`L_M, L_W, L_C, L_I`), JCS canonical JSON (RFC 8785), target tuple encoding, CD-LOAD state machine |
| CDP-STRUCT-001 | `docs/CDP-STRUCT-001.md` | Data structure specification: bundle header, TOC entry, attestation structure, manifest schema |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: bundle build, attestation, manifest, target binding, offline verification, runtime loader |

Standards cross-reference: CDP-MATH-001 §5 is the normative specification for `R`.

### certifiable-inference

| Document ID | File | Contents |
|-------------|------|----------|
| CI-MATH-001 | `docs/CI-MATH-001.md` | Mathematical specification: Q16.16 matrix operations, 2D convolution (zero dynamic allocation), activation functions (ReLU, deterministic thresholding), max pooling, timing model (O(1) per-pass), output commitment |
| CI-STRUCT-001 | `docs/CI-STRUCT-001.md` | Data structure specification: weight tensors (static allocation), activation buffers (static allocation), output commitment |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: matrix multiply, convolution, activations, pooling, timing verification, output hashing |

Standards cross-reference: CI-MATH-001 §7 is the normative specification for `H_pred`.

### certifiable-monitor

| Document ID | File | Contents |
|-------------|------|----------|
| CM-MATH-001 | `docs/CM-MATH-001.md` | Mathematical specification: Total Variation (TV), Jensen-Shannon Divergence (JSD), Population Stability Index (PSI) in fixed-point (Q0.32, Q16.16), SHA-256 audit ledger chain, health FSM transitions, COE policy schema |
| CM-STRUCT-001 | `docs/CM-STRUCT-001.md` | Data structure specification: drift detector state, ledger entry, health FSM state, COE policy, reaction descriptor |
| SRS-001 through SRS-00N | `docs/requirements/` | Software requirements: drift detection, audit ledger, input/activation/output monitoring, health FSM, reaction handling, ledger verification |

Standards cross-reference: CM-MATH-001 §6 is the normative specification for `H_audit`.

### certifiable-verify

| Document ID | File | Contents |
|-------------|------|----------|
| CV-MATH-001 | `docs/CV-MATH-001.md` | Mathematical specification: hash-only verification algorithm, full replay algorithm, cross-artefact binding verification, report hash construction |
| CV-STRUCT-001 | `docs/CV-STRUCT-001.md` | Data structure specification: verification report schema, stage result, binding result |
| SRS-001-PROVENANCE | `docs/requirements/SRS-001-PROVENANCE.md` | Requirements: data Merkle verification |
| SRS-002-TRAINING | `docs/requirements/SRS-002-TRAINING.md` | Requirements: training proof verification |
| SRS-003-QUANT | `docs/requirements/SRS-003-QUANT.md` | Requirements: quantization certificate verification |
| SRS-004-BUNDLE | `docs/requirements/SRS-004-BUNDLE.md` | Requirements: bundle attestation verification |
| SRS-005-INFERENCE | `docs/requirements/SRS-005-INFERENCE.md` | Requirements: inference hash verification |
| SRS-006-LEDGER | `docs/requirements/SRS-006-LEDGER.md` | Requirements: ledger chain verification |
| SRS-007-BINDING | `docs/requirements/SRS-007-BINDING.md` | Requirements: cross-artefact binding verification |
| SRS-008-REPORT | `docs/requirements/SRS-008-REPORT.md` | Requirements: JSON verification report generation |

`certifiable-verify` SRS documents are the primary traceability artefacts for DO-178C SW-051 and IEC 62304 5.7.

---

## 10. What certifiable-* Does Not Cover

This section is not a list of weaknesses. It is a precise boundary statement. Projects that claim to cover all certification requirements cover none credibly. The artefacts in `certifiable-*` cover the software computation layer. Everything below is an explicit integrator responsibility.

### Tool Qualification

`certifiable-*` uses gcc (C99), cppcheck, cmake, and SHA-256 implementations. None of these tools carry DO-330 qualification evidence. For DO-178C DAL-A and ISO 26262 ASIL-D, tools that generate or verify software must be qualified under DO-330 or ISO 26262 Part 8 clause 11 respectively.

The integrator must either qualify these tools or demonstrate that their outputs are verified by independent means. Note that `certifiable-verify --full-replay` may serve as independent output verification for the pipeline — but `certifiable-verify` itself would then require qualification.

### System Safety Assessment

`certifiable-*` does not perform:

- Hazard and risk assessment (HARA per ISO 26262 Part 3)
- Functional hazard assessment (FHA per ARP4754A)
- Fault tree analysis (FTA per ARP4761)
- Failure modes and effects analysis (FMEA)
- Preliminary system safety assessment (PSSA)
- System safety assessment (SSA)
- ASIL assignment or decomposition

These activities establish that the ML pipeline function is safe at the system level. `certifiable-*` proves the software computation is correct with respect to its specification. It does not establish what the specification should be to make the system safe.

### Hardware Design Assurance

`certifiable-*` is a software-only artefact. Hardware design assurance (DO-254 for airborne electronic hardware; ISO 26262 Parts 4 and 5 for automotive hardware) is outside scope. `certifiable-deploy` performs target binding — it verifies that a bundle matches a declared platform — but it does not perform hardware qualification.

### Quality Management System

`certifiable-*` does not provide a quality management system. IEC 62304 requires an IEC 13485-compliant QMS. ISO 26262 requires a safety management system. The integrator's QMS governs the development process within which `certifiable-*` artefacts are produced and used.

### Risk Management (Medical)

ISO 14971 risk management — hazard identification, risk estimation, risk evaluation, and risk control — is outside `certifiable-*` scope. The certifiable-* commitment chain provides evidence relevant to risk control measures (proving software identity), but does not perform the risk management process.

### Worst-Case Execution Time (Formal)

`certifiable-inference` provides timing verification tests that demonstrate less than 5% jitter at the 95th percentile. It does not provide a WCET certificate from a qualified WCET analysis tool (AbsInt aiT, Rapita RVS, or equivalent). For DO-178C DAL-A and ISO 26262 ASIL-D, the integrator must perform formal WCET analysis on their target hardware using qualified tooling.

### MC/DC Coverage (Formal Measurement)

`certifiable-*` test suites cover all branches of all DVM primitive operations, including fault paths. They do not produce MC/DC coverage reports from a qualified coverage measurement tool. For DO-178C DAL-A, the integrator must instrument and measure MC/DC coverage using qualified tooling.

### Airworthiness and Regulatory Interfaces

`certifiable-*` does not interface with any airworthiness authority (FAA, EASA, TCCA) or medical device regulatory body (FDA, MHRA, Notified Body). Regulatory submission and approval are integrator responsibilities.

These responsibilities remain with the integrator's certification programme and are intentionally outside the scope of the `certifiable-*` artefact set.

---

## 11. Contact and Commercial Licensing

### Contact

**William Murray**  
Founder, SpeyTech  
Visiting Scholar, Heriot-Watt University  
Email: william@fstopify.com  
Web: [speytech.com](https://speytech.com)

### Licensing

`certifiable-*` is dual licensed:

- **Open Source:** GNU General Public License v3.0 — free for open source projects
- **Commercial:** Commercial licence available for safety-critical deployments requiring certification support, proprietary integration, or compliance documentation packages

For commercial licensing, compliance documentation packages, or certification assistance engagements, contact william@fstopify.com.

### Patent

The `certifiable-*` ecosystem implements deterministic computing primitives defined by the **Murray Deterministic Computing Platform (MDCP)**, protected by UK Patent **GB2521625.0 (Murray Deterministic Computing Platform)**.

---

*Copyright © 2026 The Murray Family Innovation Trust. All rights reserved.*
