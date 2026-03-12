# DVM-SPEC-001
## Deterministic Virtual Machine Specification

| Field | Value |
|---|---|
| Document ID | DVM-SPEC-001 |
| Version | 1.0-rc1 |
| Date | 12 March 2026 |
| Author | William Murray, SpeyTech |
| Patent | UK GB2521625.0 (Murray Deterministic Computing Platform) |
| Status | Release candidate — ready for external architectural review |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Architecture](#2-system-architecture)
3. [Determinism Contract](#3-determinism-contract)
4. [Canonical State Model](#4-canonical-state-model)
5. [Total Computation Model](#5-total-computation-model)
6. [Ingress Oracle](#6-ingress-oracle)
7. [Cryptographic Commitments](#7-cryptographic-commitments)
8. [Composition Semantics](#8-composition-semantics)
9. [Conformance Requirements](#9-conformance-requirements)
10. [Reference Implementation](#10-reference-implementation)

---

## 1. Introduction

### 1.1 Purpose

This document defines the Deterministic Virtual Machine (DVM) specification.

The DVM is a computation model designed to guarantee:

- deterministic execution
- canonical state representation
- bounded computation
- cryptographically verifiable execution history

The specification defines the execution contract under which computations are considered DVM-compliant.

The DVM specification is language-agnostic. It defines semantics and invariants that must hold for any conforming implementation. Reference implementations may exist in multiple programming languages.

The planned reference implementation is **libdvm** (C99).

### 1.2 Motivation

Modern computational systems often fail to provide reliable reproducibility across platforms due to:

- floating-point nondeterminism
- undefined behaviour
- unbounded execution semantics
- implicit external state
- non-canonical data encodings

These properties make it difficult or impossible to independently verify computational results.

The Deterministic Virtual Machine addresses this problem by defining a computational model where:

- execution is deterministic
- state representation is canonical
- computation is total
- execution history can be cryptographically committed
- results can be independently replayed and verified

The objective of the DVM is not unrestricted computational expressiveness, but **verifiable computation** suitable for safety-critical systems.

> **Turing completeness is a vulnerability. Totality is a feature.**

### 1.3 Invariants Summary

A DVM-compliant system MUST satisfy all of the following invariants. Each is fully specified in the referenced section.

| # | Category | Invariant | Section |
|---|---|---|---|
| 1 | Execution determinism | Deterministic arithmetic | §3.1 |
| 2 | Execution determinism | Mathematical closure (fault as valid state) | §3.2 |
| 3 | Execution determinism | Deterministic entropy | §3.3 |
| 4 | Execution determinism | Absence of hidden external state | §3.4 |
| 5 | State model | Canonical state encoding | §4 |
| 6 | Termination | Total computation (bounded, non-recursive, static) | §5 |
| 7 | System boundary | Explicit oracle boundary | §6 |
| 8 | Evidence model | Cryptographic commitment with domain separation | §7 |
| 9 | Evidence model | Deterministic replay theorem | §7.3 |
| 10 | Composability | Composition closure | §8.2 |
| 11 | Composability | Deterministic fault propagation | §8.5 |

### 1.4 Layered Specification Model

The DVM architecture is defined across three layers.

**Specification layer**
Defines the deterministic computation model. This document constitutes the specification layer.

**Implementation layer**
Conforming runtime implementations of the DVM contract. The planned reference implementation is libdvm (C99). Independent implementations in other languages are permitted provided they satisfy all conformance requirements in §9.

**Application layer**
Deterministic systems constructed on conforming DVM implementations. Examples include the certifiable-* ML pipeline and future domain-specific systems.

Implementations MUST conform to the specification layer. Applications MUST depend only on conforming implementations.

![DVM Layered Specification Model](dvm-01-layered-architecture.svg)

---

## 2. System Architecture

The DVM exists within a layered architecture separating nondeterministic external inputs from deterministic computation.

```
External Environment
        │
[Domain-Specific Oracles] ──┐
                             ▼
                      Ingress Oracle
                             │
                             ▼
                    Canonical DVM State
                             │
                             ▼
                       DVM Execution
                             │
                             ▼
                   Canonical Output State
                             │
                             ▼
                  Cryptographic Commitment
                             │
                             ▼
                     Application Layer
```

| Layer | Description |
|---|---|
| External Environment | Physical sensors, datasets, or external systems |
| Domain-Specific Oracles | Specialised adapters converting domain inputs to oracle-compatible form |
| Ingress Oracle | Boundary converting external input into canonical DVM state |
| Canonical State | Deterministic byte representation of computation state |
| DVM Execution | Deterministic computation engine |
| Canonical Output State | Deterministic byte representation of computation result |
| Commitment Layer | Cryptographic commitment of canonical state |
| Application Layer | Domain-specific systems built on the DVM |

The architecture enforces a strict separation between nondeterministic physical environments and deterministic mathematical computation.

Domain-Specific Oracles are permitted to adapt domain-specific input formats before they reach the Ingress Oracle boundary. All such adapters MUST produce output that satisfies the canonical state encoding rules defined in §4.

---

## 3. Determinism Contract

A DVM-compliant computation MUST satisfy all of the following invariants.

### 3.1 Deterministic Arithmetic

All arithmetic operations MUST have strictly defined, platform-independent semantics.

Implementations SHALL NOT rely on platform floating-point arithmetic or hardware rounding modes.

Arithmetic SHALL be defined using fixed-precision integer operations with explicit rounding and saturation rules.

The canonical fixed-point representation is Q16.16 as defined in §4.3.

### 3.2 Principle of Closure

The DVM execution model MUST be mathematically closed.

A DVM computation is a total function mapping any valid canonical state to another valid canonical state:

$$F : S \to S$$

Undefined behaviour is forbidden. Operations that traditionally yield undefined behaviour — including division by zero, arithmetic overflow, or invalid domain inputs — SHALL NOT trigger hardware traps or host exceptions.

Instead:

- results MUST saturate to predefined bounds
- deterministic fault vectors MUST be recorded within the canonical state

A fault condition is a **valid computation state**, not an exceptional exit.

### 3.3 Deterministic Entropy

Pseudo-randomness MAY be used within DVM computations provided it is strictly deterministic.

Hardware entropy sources — including `/dev/urandom`, `rdrand`, or system clocks — are prohibited.

Entropy MUST be derived exclusively from a counter-based pseudo-random number generator (PRNG):

$$\text{bits} = \text{PRNG}(\text{seed},\ \text{counter})$$

The PRNG implementation MUST satisfy the following constraints:

**Algorithmic Determinism**
The algorithm MUST be fully specified and platform-independent.

**Cryptographic Strength**
The PRNG MUST be based on a cryptographic-grade construction. Permitted algorithms include ChaCha20 (recommended), AES-CTR, or a formally specified Feistel network.

**Deterministic Seeding**
The seed MUST be deterministically derived from the canonical input state. The RECOMMENDED derivation is:

$$\text{seed} = \text{commit}(s_0)$$

where `commit` is the domain-separated commitment function defined in §7.1 and $s_0$ is the canonical initial state.

**Monotonic Counter**
The counter MUST be a strictly monotonically increasing integer derived from deterministic execution steps. The counter SHOULD correspond to the logical computation step, such as loop iteration index, batch index, or pipeline stage index. The counter MUST be deterministic for a given execution path and MUST NOT depend on thread scheduling or timing.

**Statelessness**
The PRNG SHALL NOT maintain hidden internal state between invocations. The output MUST be a pure function of `(seed, counter)`.

### 3.4 Absence of Hidden External State

DVM execution SHALL NOT depend on:

- system clocks
- environment variables
- host memory addresses
- OS scheduling
- hardware randomness

All external inputs MUST enter the system exclusively via the Ingress Oracle (§6).

### 3.5 Deterministic Execution Theorem

Let $S$ be the set of valid canonical states, $F$ be a DVM computation, and $A$ and $B$ be two conforming implementations. For any input state $s \in S$:

$$F_A(s) = F_B(s)$$

This property guarantees **bit identity** across platforms.

Determinism is defined over canonical state transitions, not over machine-level execution traces. Two conforming implementations may differ in internal execution path, register allocation, or memory layout, provided they produce identical canonical output states.

---

## 4. Canonical State Model

### 4.1 Overview

All DVM computations operate over canonical states.

A canonical state is a deterministic byte representation of computation state. Canonical encoding ensures that semantically equivalent states produce byte-identical representations.

Canonical state includes only explicitly declared state structures. Canonical state SHALL NOT include program counters, call stacks, heap allocators, or other execution artefacts.

Canonical states are independent of:

- CPU architecture
- compiler implementation
- host memory layout
- runtime metadata

Canonical state encoding forms the basis of cryptographic commitment and replay verification.

### 4.2 Encoding Principles

Canonical state representation MUST satisfy the following rules.

**Deterministic Field Ordering**
Composite structures MUST define fixed field ordering. Field ordering SHALL NOT depend on reflection, language metadata, or runtime behaviour. This rule applies to any composite type not explicitly listed in §4.4 — deterministic traversal order must be derivable without runtime inspection.

**Fixed-Width Types**
All primitive types MUST have explicitly defined widths. Implicit widening or narrowing is forbidden.

**Explicit Endianness**
All multibyte values MUST use fixed byte ordering. The DVM canonical representation SHALL use little-endian encoding.

**No Compiler Padding**
Canonical state SHALL NOT contain compiler-generated padding. Structure layouts MUST be explicitly defined with no implicit gaps between fields.

**Byte-Packed Encoding**
Canonical state encoding SHALL be byte-packed with no alignment gaps. Architectures with strict alignment requirements MUST NOT introduce hidden padding to satisfy hardware alignment constraints. Implementations MAY use aligned internal representations for performance or hardware compatibility, provided that canonical serialisation always produces byte-packed output conforming to this rule. The canonical encoding is the authoritative representation; internal layout is an implementation detail.

**Explicit Absence**
Optional fields MUST be explicitly encoded. Implicit omission of fields is forbidden.

**Deterministic Padding for Bounded Payloads**
Canonical structures MAY contain fixed-capacity buffers. If a logical payload occupies less than the capacity of a fixed-size structure, all remaining unused bytes MUST be explicitly initialised to zero (`0x00`) or another predefined deterministic byte value. Uninitialized stack or heap memory SHALL NEVER appear in canonical state representations.

![Canonical State Memory Layout](dvm-02-canonical-memory-layout.svg)

### 4.3 Primitive Types

| Type | Description | Width |
|---|---|---|
| `int8` .. `int64` | Signed two's complement integer | 8–64 bits |
| `uint8` .. `uint64` | Unsigned integer | 8–64 bits |

All signed integer types SHALL be encoded using two's complement representation. One's complement and sign-magnitude representations are prohibited.
| `fixed<M,N>` | Signed fixed-point: M integer bits, N fractional bits, total (M+N+1) bits | (M+N+1) bits |
| `bool` | Boolean: `0x00` = false, `0x01` = true | 8 bits |
| `enum<N>` | Integer-backed enumeration with explicitly defined mapping | N bits |
| `fault_vector` | Structured deterministic fault flags (see §4.5) | defined per domain |

**Canonical fixed-point numeric family — `fixed<M,N>`**

The DVM defines a family of signed fixed-point types parameterised by integer bits M and fractional bits N. The total representation is (M + N + 1) bits including one sign bit, stored as a two's complement integer of that width, little-endian.

Domain systems MUST declare the specific `fixed<M,N>` type used and MUST NOT mix types implicitly. Permitted standard instances include:

| Instance | Alias | Total width | Range | Resolution |
|---|---|---|---|---|
| `fixed<15,16>` | `q16.16` | 32 bits | ≈ ±32768 | $2^{-16}$ |
| `fixed<15,32>` | `q16.32` | 48 bits | ≈ ±32768 | $2^{-32}$ |
| `fixed<31,32>` | `q32.32` | 64 bits | ≈ ±2.1×10⁹ | $2^{-32}$ |
| `fixed<23,8>` | `q24.8` | 32 bits | ≈ ±8.4×10⁶ | $2^{-8}$ |

`q16.16` remains the default and RECOMMENDED type for general-purpose DVM computation. Domain systems requiring extended range (aerospace navigation, large-scale financial models) or higher resolution (ML weight gradients) SHOULD select the smallest instance sufficient for their domain and declare it explicitly in their domain specification.

**Arithmetic rules for `fixed<M,N>`:**

- **Addition / subtraction:** positive overflow saturates to the maximum representable value of the `fixed<M,N>` type, negative overflow saturates to the minimum representable value, and sets the `overflow` fault flag. For the standard `q16.16` instance these are `INT32_MAX` and `INT32_MIN` respectively.
- **Multiplication:** the double-width intermediate product is right-shifted N bits; positive overflow saturates to the maximum representable value, negative overflow to the minimum representable value, and sets the `overflow` fault flag.
- **Division:** the dividend is left-shifted N bits before integer division. Division by zero saturates to the maximum representable value and sets the `division_by_zero` fault flag. Overflow (e.g. minimum value divided by −1) saturates to the maximum representable value and sets the `overflow` fault flag.
- **Rounding:** truncation toward zero on all right-shift operations. Round-half-up and banker's rounding are prohibited.

### 4.4 Composite Types

Allowed composite structures:

| Type | Description |
|---|---|
| `struct` | Fixed-field ordered record with no padding |
| `array<T, N>` | Fixed-length homogeneous sequence |
| `tuple` | Fixed-arity ordered sequence of typed elements |

The following structures are **prohibited**:

- unordered maps
- pointer graphs
- dynamically sized objects
- reflection-dependent layouts

These structures cannot guarantee a deterministic traversal order derivable without runtime inspection, which is the property required for canonical encoding.

### 4.5 Deterministic Fault Vectors

Canonical state MUST include deterministic fault vectors describing exceptional conditions encountered during computation.

| Flag | Condition |
|---|---|
| `overflow` | Arithmetic result exceeded representable range; result is saturated |
| `division_by_zero` | Divisor was zero |
| `invalid_domain` | Input outside the defined domain of the operation |
| `saturation` | Result was clamped to a boundary value for reasons other than arithmetic overflow |

Note: overflow always implies saturation, but saturation does not necessarily imply overflow. The `saturation` flag MAY additionally be set in cases where clamping occurs without overflow, such as explicit range enforcement at domain boundaries. Implementations MUST NOT set `overflow` without also setting `saturation`.

Fault conditions SHALL NOT terminate execution. They MUST be propagated within canonical state according to the fault propagation semantics defined in §8.5.

### 4.6 Canonical Representation Theorem

Let $S$ be the set of semantic states and $C$ be the canonical encoding function. Where $\equiv$ denotes semantic equivalence under the DVM state model:

$$\forall\, s_1, s_2 \in S,\quad s_1 \equiv s_2 \implies C(s_1) = C(s_2)$$

This guarantees that semantically equivalent states produce identical cryptographic commitments.

### 4.7 Canonical State Versioning

Canonical encodings are versioned. The version is embedded in the commitment tag (§7.1).

Stability guarantee:

- Canonical encodings defined within a major version SHALL NOT change.
- Breaking changes to canonical encoding require a new major version and a new tag value.
- A commitment chain produced under `DVM:STATE:v1` MUST remain independently verifiable under any future version of this specification. Cross-version verification requires the original version tag to be preserved within the commitment record.

---

## 5. Total Computation Model

The DVM enforces total computation. Every valid DVM computation MUST terminate.

### 5.1 Bounded Loops

All loops MUST have statically determinable bounds. Unbounded iteration is forbidden.

### 5.2 No Recursion

Recursive function calls are prohibited. The call graph of a DVM-compliant program MUST be a directed acyclic graph (DAG), with call depth statically determinable at compile time.

### 5.3 Static Allocation

Dynamic memory allocation during execution is forbidden. All memory MUST be allocated during initialisation.

### 5.4 Termination Guarantee

Let $P$ be a valid DVM program and $s$ a valid canonical state.

The combination of §5.1–5.3 ensures all execution paths reach a final state in time bounded by the product of loop bounds and call depth, both of which are statically determinable. Execution therefore evaluates to a final state $s'$ in finite time for all valid inputs.

The canonical state space $S$ is finite. Fixed-width primitive types (§4.3) and statically bounded composite structures (§4.4) together bound the total number of representable states. There is no mechanism by which a DVM computation can produce a state outside this bounded space.

This property permits WCET analysis by construction, without requiring post-hoc static analysis tooling.

---

## 6. Ingress Oracle

### 6.1 Purpose

The physical world and host environments are inherently nondeterministic.

The Ingress Oracle defines the strict boundary between external entropy and deterministic computation. Its role is to:

- capture external input
- normalise and quantise it to primitive types defined in §4.3. Quantisation MUST produce identical canonical values when applied to identical external inputs.
- convert it into canonical DVM state conforming to §4
- establish cryptographic provenance via an ingress commitment

### 6.2 Boundary Theorem

Let $E$ be the set of external states and $S$ be the set of canonical states. The Oracle defines a projection:

$$O : E \to (s \in S,\ L_{\text{ingress}})$$

Where $s$ is the canonical state and $L_{\text{ingress}}$ is the commitment anchoring the input.

Once the projection occurs, all subsequent execution depends solely on $s$. The ingress commitment $L_{\text{ingress}}$ preserves evidence of what the external world provided, independently of what the DVM computed.

This split provides two independently auditable records:

| Record | What it proves |
|---|---|
| $L_{\text{ingress}}$ | What the physical world provided |
| Commitment chain | What the DVM computed from that input |

The Ingress Oracle operates outside the deterministic domain of the DVM. Its behaviour must therefore be independently specified for each deployment to ensure reproducibility of canonical state generation. Deployments SHOULD provide an Oracle Specification document describing the oracle's normalisation and quantisation rules, their deterministic properties, and the verification strategy for oracle conformance.

![Ingress Oracle Boundary](dvm-03-ingress-oracle-boundary.svg)

### 6.3 Domain-Specific Oracles

Real-world systems present inputs in domain-specific formats — sensor packets, market data feeds, imaging acquisition streams, control law parameter sets. Domain-Specific Oracles are permitted adapters that normalise such formats before they reach the Ingress Oracle boundary.

A Domain-Specific Oracle MUST:

- produce output that satisfies the canonical state encoding rules of §4
- apply no computation that depends on hidden external state
- be fully specified and independently verifiable

Domain-Specific Oracles do not extend the DVM boundary. They are pre-oracle adapters. The Ingress Oracle commitment marks the boundary; domain adapter output is not committed until it passes through the Ingress Oracle.

### 6.4 Time as an Oracle Input

Real-time control systems frequently require measurements of elapsed time ($\Delta t$).

Because system clocks and oscillator drift are nondeterministic, direct clock access from within the DVM execution environment is prohibited.

Time MUST be treated as an external sensor input. The Ingress Oracle is the sole entity permitted to read the hardware clock. The Oracle MUST convert timestamps into deterministic integer or fixed-point representations, conforming to §4.3, and inject them into the canonical state.

Within the DVM execution environment, time is treated exclusively as immutable state data. There is no concept of "current time" inside DVM execution.

---

## 7. Cryptographic Commitments

Canonical states MUST be cryptographically committed using SHA-256 (FIPS 180-4). SHA-256 is the required hash function for version 1 of this specification. Future revisions of this specification MAY define additional approved algorithms; implementations claiming conformance to a specific version MUST use only the algorithm defined for that version.

### 7.1 Domain-Separated and Versioned Commitment Function

To prevent cross-protocol collisions and ensure forward compatibility, all commitments MUST use domain separation with explicit encoding versioning.

The commitment function is defined as:

$$\text{commit}(\text{state}) = H(\text{tag} \parallel \text{len} \parallel \text{state})$$

| Field | Description |
|---|---|
| $H$ | Cryptographic hash function (SHA-256 or equivalent) |
| `tag` | State type identifier and encoding version (see format below) |
| `len` | Byte length of canonical state encoding, encoded as `uint64` little-endian |
| `state` | Canonical byte encoding of the state |

**Tag format:**

```
tag = "DVM:" || domain || ":v" || version_integer
```

| Tag | Meaning |
|---|---|
| `DVM:STATE:v1` | Canonical computation state, encoding version 1 |
| `DVM:INGRESS:v1` | Ingress oracle commitment, encoding version 1 |
| `DVM:LEDGER:v1` | Ledger chain digest, encoding version 1 |

The version integer in the tag corresponds to the major version of the canonical encoding under which the commitment was produced. This is independent of the DVM-SPEC-001 document version.

### 7.2 Commitment Chains

Sequential computations MAY form commitment chains.

Let $s_0 \to s_1 \to \cdots \to s_n$ be a sequence of canonical states.

The chain is initialised as:

$$L_0 = H(\texttt{DVM:LEDGER:v1} \parallel \text{commit}(s_0))$$

where `commit(s₀)` uses tag `DVM:STATE:v1`. The outer hash with tag `DVM:LEDGER:v1` distinguishes the ledger root from the bare state commitment.

The ledger digest at step $t > 0$ is:

$$L_t = H(\text{tag}_{\text{ledger}} \parallel L_{t-1} \parallel \text{commit}(s_t))$$

where $\text{tag}_{\text{ledger}}$ = `DVM:LEDGER:v1`.

This creates a deterministic computational ledger. Any step in the chain is independently verifiable by replaying the computation from $s_0$ and recomputing the commitment sequence.

### 7.3 Deterministic Replay Theorem

Let $s_0$ be a canonical initial state, $P$ a DVM program, and $C = (L_0, L_1, \ldots, L_n)$ a commitment chain produced by a conforming implementation. For any other conforming implementation replaying $P(s_0)$:

$$C' = (L_0', L_1', \ldots, L_n') \implies C' = C$$

That is, any conforming implementation given the same initial canonical state and program MUST produce a commitment chain that is byte-identical at every position.

**Proof sketch:** Each $L_t$ is a deterministic function of $\text{tag}_{\text{ledger}}$, $L_{t-1}$, and $\text{commit}(s_t)$. Each $\text{commit}(s_t)$ is a deterministic function of $s_t$. Each $s_t$ is produced by $F_t(s_{t-1})$, which by the determinism contract (§3.5) is bit-identical across conforming implementations. Induction from $L_0$ therefore guarantees $C' = C$.

![Cryptographic Commitment Chain and Replay Theorem](dvm-04-commitment-chain.svg)

This theorem is the core verifiability guarantee of the DVM. It states that the system's computational history is independently auditable: any party holding $s_0$ and $P$ can verify any commitment chain produced by any conforming implementation.

### 7.4 Computational Ledger Property

A DVM commitment chain constitutes a cryptographically verifiable ledger of deterministic computation.

This property has two consequences:

**Tamper evidence.** Any modification to any intermediate state $s_k$ produces a different $\text{commit}(s_k)$, which invalidates $L_k$ and every subsequent digest $L_{k+1} \ldots L_n$. Tampering is detectable without access to any secret.

**Independent auditability.** The ledger requires no trusted third party. Any conforming implementation can verify the chain by replaying $P(s_0)$ and comparing the resulting commitment sequence. This property supports independent certification review, regulatory audit, and multi-party verification without shared secrets or trusted intermediaries.

The DVM therefore provides not merely a deterministic runtime but a **verifiable computation ledger** — a cryptographic record of what was computed, from what starting state, producible and verifiable by any conforming implementation.

---

## 8. Composition Semantics

### 8.1 Computation Mapping

Let $S$ be the canonical state space. Two DVM computations are defined as:

$$F_1 : S \to S \qquad F_2 : S \to S$$

### 8.2 Composition Theorem

The composed computation is defined as:

$$F_2 \circ F_1(s) = F_2(F_1(s))$$

If both $F_1$ and $F_2$ satisfy the DVM determinism contract, then their composition also satisfies the contract.

**Proof sketch:** Since $F_1$ produces canonical state by the determinism contract, and $F_2$ accepts canonical state and produces canonical state by the same contract, the composition is closed over $S$. Determinism of the output follows from the determinism of each stage and the absence of hidden external state (§3.4).

### 8.3 Pipeline Execution

A pipeline of computations produces deterministic state transitions:

$$s_n = (F_n \circ \cdots \circ F_1)(s_0)$$

This sequence is fully replayable. Given $s_0$ and the sequence of computations, any conforming implementation produces $s_n$ identically.

### 8.4 Parallelism

The DVM specification defines sequential deterministic semantics as the normative execution model.

Parallel execution — including GPU, SIMD, and distributed compute — is permitted as an implementation strategy, subject to the following constraint:

> A parallel implementation is DVM-compliant if and only if the canonical state it produces at each step is bit-identical to the canonical state that would be produced by sequential evaluation as defined by this specification.

The burden of proving this equivalence rests with the implementation. Parallel implementations MUST provide conformance evidence demonstrating bit-identity against the sequential reference on the test suite defined in §9. The canonical state, not the execution path, is the normative output.

### 8.5 Fault Propagation Semantics

Fault conditions captured in the canonical state fault vector MUST propagate deterministically through computation chains.

Let $s_{\text{faulted}}$ be a canonical state containing an active fault vector. For any computation $F_n$ applied to $s_{\text{faulted}}$, the implementation MUST explicitly define one of the following deterministic behaviours:

**1. Identity Propagation**

$$F_n(s_{\text{faulted}}) = s_{\text{faulted}}$$

**2. Deterministic Degraded Mode**
$F_n$ executes a predefined fallback computation designed to operate safely under the active fault condition. This mode is appropriate for safety-critical domains — such as aerospace flight control — where forward computation must continue under defined constraints rather than halting.

**3. Explicit Fault Handling**
$F_n$ may transform the fault vector according to documented deterministic rules.

Under no circumstances SHALL a computation silently clear or ignore a fault vector. Fault state transitions MUST be observable within canonical state.

![Mathematical Closure and Fault Propagation](dvm-05-fault-propagation.svg)

The chosen fault propagation strategy for each computation MUST be declared in the implementation documentation and is subject to conformance verification (§9).

---

## 9. Conformance Requirements

An implementation is DVM-compliant if it satisfies all of the following requirements. For each requirement, the verification method is stated.

| Requirement | Section | Verification method |
|---|---|---|
| Deterministic arithmetic | §3.1 | Static analysis confirming absence of floating-point operations; arithmetic unit tests with known fixed-point values and expected saturation behaviour |
| Mathematical closure | §3.2 | Fault injection tests confirming no hardware trap, no undefined behaviour, deterministic fault vector in output |
| Deterministic entropy | §3.3 | Code review confirming no hardware entropy source; PRNG output reproducibility tests with fixed seed and counter |
| Absence of hidden external state | §3.4 | Static analysis; code review confirming no clock, environment variable, or hardware randomness access outside the Ingress Oracle |
| Canonical state encoding | §4 | Round-trip serialisation tests; byte-identity tests across platforms and compiler versions |
| Deterministic fault vectors | §4.5 | Fault injection test suite confirming all defined conditions produce deterministic canonical output |
| Total computation model | §5 | Static analysis confirming bounded loops, no recursion, no dynamic allocation post-initialisation |
| Ingress Oracle rules | §6 | Oracle test suite confirming canonical output for defined input classes; no direct clock or entropy access |
| Cryptographic commitment rules | §7 | Commitment test vectors; cross-implementation commitment equality tests; chain initialisation and extension tests |
| Deterministic replay theorem | §7.3 | Cross-implementation replay test: two independent conforming implementations given identical $s_0$ and $P$ must produce byte-identical commitment chains |
| Composition closure | §8.2 | Pipeline integration tests; composed pipeline output must be byte-identical to sequential application |
| Parallelism equivalence | §8.4 | Where parallel execution is used: bit-identity tests against sequential reference on full conformance test suite |
| Fault propagation | §8.5 | Test suite confirming declared propagation strategy is applied; no silent fault clearing across pipeline stages |

Conformance evidence for the certifiable-* application layer is produced by the `certifiable-verify` repository and its associated test harness.

---

## 10. Reference Implementation

The planned reference implementation of the DVM is **libdvm** (C99).

libdvm will expose the `dvm_*` family of primitives implementing:

- Q16.16 fixed-point arithmetic with saturation and fault flags
- Canonical state serialisation
- SHA-256 commitment construction with domain separation and versioned tagging
- Commitment chain ledger operations including chain initialisation
- Counter-based PRNG (ChaCha20) with `commit(s₀)` seed derivation

Domain systems such as the certifiable-* pipeline build on libdvm to construct deterministic computational pipelines targeting DO-178C, IEC 62304, and ISO 26262 certification evidence.

Built on the Murray Deterministic Computing Platform (MDCP), UK Patent GB2521625.0.

---

*End of DVM-SPEC-001 v1.0-rc1*
