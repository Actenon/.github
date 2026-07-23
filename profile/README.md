# Actenon

> Authority enforcement and verifiable evidence for autonomous systems. An agent shouldn't be able to take a consequential action without scoped, signed authority for that exact action — and every action should leave proof a third party can verify.

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](https://github.com/Actenon/actenon-kernel/blob/main/LICENSE)
[![Ecosystem: 5 repos](https://img.shields.io/badge/Ecosystem-5%20repos-orange.svg)](#the-actenon-ecosystem)
[![Languages: Py · TS · Go · Rust](https://img.shields.io/badge/Languages-Py%20%C2%B7%20TS%20%C2%B7%20Go%20%C2%B7%20Rust-blue.svg)](#multi-language-sdks)
[![Stance: open & vendor-neutral](https://img.shields.io/badge/Stance-open%20%26%20vendor--neutral-2ea44f.svg)](#principles)

---

## Start here

```bash
pip install actenon-scan && actenon-scan scan .
```

Find the execution gap in your own code in 10 seconds. Then:

- **[`actenon-permit`](https://github.com/Actenon/actenon-permit)** — the reference enforcement demo. `pip install actenon-permit` → `permit demo`.
- **[`actenon-kernel`](https://github.com/Actenon/actenon-kernel)** — the proof gate. The trust anchor that verifies every action at the execution edge.
- **[`actenon-protocol`](https://github.com/Actenon/actenon-protocol)** — the wire contract. Zero dependencies, any language, any framework.
- **[`actenon-cloud`](https://github.com/Actenon/actenon-cloud)** — the hosted control plane. Optional.

## The problem

Modern agent stacks answer the upstream question — *should this requester be allowed to do this kind of thing?* — with authentication, policy engines, approval workflows, and audit logs. They still leave open the question the execution edge needs to answer:

> Is the exact action about to execute still the exact action that was authorized — for this endpoint, this tenant, this subject, this target, this scope, this time window, and this single execution attempt?

That unanswered question is the **execution gap**. Actenon closes it.

## The Actenon ecosystem

| Repo | Role | Depends on | Badges |
|---|---|---|---|
| [**`actenon-protocol`**](https://github.com/Actenon/actenon-protocol) | The neutral wire contract. Zero dependencies. | *nothing* | [![PyPI](https://img.shields.io/pypi/v/actenon-protocol?label=PyPI)](https://pypi.org/project/actenon-protocol/) [![npm](https://img.shields.io/npm/v/@actenon/protocol-types?label=npm)](https://www.npmjs.com/package/@actenon/protocol-types) |
| [**`actenon-kernel`**](https://github.com/Actenon/actenon-kernel) | The open verifier. Verifies proofs at the execution edge; issues no grants; runs no policy decisions. | `actenon-protocol` | [![PyPI](https://img.shields.io/pypi/v/actenon-kernel?label=PyPI)](https://pypi.org/project/actenon-kernel/) [![Conformance](https://img.shields.io/badge/Conformance-1.0.0-51%20vectors-success.svg)](https://github.com/Actenon/actenon-kernel/blob/main/docs/CONFORMANCE.md) |
| [**`actenon-permit`**](https://github.com/Actenon/actenon-permit) | The developer on-ramp + authority broker. Issues signed grants, runs the PDP, brokers credentials. | `actenon-kernel`, `actenon-protocol` | [![PyPI](https://img.shields.io/pypi/v/actenon-permit?label=PyPI)](https://pypi.org/project/actenon-permit/) [![npm](https://img.shields.io/npm/v/@actenon/sdk?label=npm)](https://www.npmjs.com/package/@actenon/sdk) |
| [**`actenon-cloud`**](https://github.com/Actenon/actenon-cloud) | The optional managed control plane. Multi-tenant, hosted, 9-layer evidence bundles. | `actenon-kernel`, `actenon-permit` | [![Optional](https://img.shields.io/badge/Deployment-optional-2ea44f.svg)](https://github.com/Actenon/actenon-cloud) |
| [**`actenon-scan`**](https://github.com/Actenon/actenon-scan) | The independent scanner. Finds the execution gap in any codebase. Zero dependencies. | *nothing* | [![PyPI](https://img.shields.io/pypi/v/actenon-scan?label=PyPI)](https://pypi.org/project/actenon-scan/) [![GitHub Action](https://img.shields.io/badge/GitHub%20Action-v1-blue.svg)](https://github.com/Actenon/actenon-scan#github-action) |

### How to choose where to start

| If you are... | Start here | Why |
|---|---|---|
| An engineer evaluating Actenon for the first time | [`actenon-permit`](https://github.com/Actenon/actenon-permit) | The on-ramp. `Actenon.local()` gives you the full feature set in-process — no Cloud, no login. |
| A security reviewer or auditor | [`actenon-kernel`](https://github.com/Actenon/actenon-kernel) | The trust anchor. 51 conformance vectors, public threat model, audit responses. |
| A platform team deciding on a wire format | [`actenon-protocol`](https://github.com/Actenon/actenon-protocol) | The neutral contract. Zero dependencies. Any language, any framework, any cloud. |
| A DevSecOps engineer who wants CI integration today | [`actenon-scan`](https://github.com/Actenon/actenon-scan) | Zero-dependency scanner with a GitHub Action that emits SARIF to the Security tab. |
| A platform team that wants a hosted control plane | [`actenon-cloud`](https://github.com/Actenon/actenon-cloud) | Multi-tenant, 9-layer evidence bundles, insurer-facing clarity. **Optional** — everything else runs without it. |

## Multi-language SDKs

| Language | Package | Use case |
|---|---|---|
| **Python** | `pip install actenon-permit` · `pip install actenon-kernel` · `pip install actenon-protocol` · `pip install actenon-scan` (all on PyPI) | Reference implementation. Full kernel: minter, verifier, executor, CLI, conformance. |
| **TypeScript** | `npm install @actenon/sdk` (v1.4.0) · `npm install @actenon/protocol-types` (v1.0.0) | Discriminated result types, receipt verification, protocol parity with Python. |
| **Go** | [sdk/go/](https://github.com/Actenon/actenon-kernel/tree/main/sdk/go) | Verifier-edge proof checking in Go HTTP services. |
| **Rust** | [sdk/rust/](https://github.com/Actenon/actenon-kernel/tree/main/sdk/rust) | Verifier-edge proof checking in systems components. |

Every SDK runs against the same 51 conformance vectors in the Kernel. Conformance, not pedigree, decides validity.

## Principles

1. **The execution edge is the only trustworthy trust boundary.** Upstream controls matter; they are not the missing boundary.
2. **Proof must be bound to the exact action.** Action name, target, tenant, subject, audience, scope, time window, single-use nonce. Any mutation is detected at the edge.
3. **The Kernel does not issue grants or make policy decisions.** That's Permit's job. The Kernel verifies proofs at the edge and refuses on any failure.
4. **The agent never holds the production credential.** The broker resolves it server-side after verification passes.
5. **Submission is not execution.** A `submitted` state is non-final. `succeeded` requires a cryptographically verified receipt.
6. **Cryptography proves execution integrity and authority-process integrity.** It does not prove business decision correctness. We are honest about which is which.
7. **Every repo is independently adoptable.** The Kernel runs without Permit, Cloud, or Scan. Scan runs without anything. Protocol runs without anything. Cloud is optional. Permit runs without Cloud.
8. **Conformance, not pedigree.** A third-party proof that conforms to the Kernel's conformance vectors will be accepted by the Kernel verifier, regardless of who issued it.
9. **Vendor-neutral by design.** Scan recognises 30+ non-Actenon guard patterns. The Protocol is implementable by any vendor. Cloud is optional. No lock-in.
10. **Source-disciplined incident reconstruction.** Pattern language only. No uncited facts about named incidents.

## See also

- [The Execution Gap](https://github.com/Actenon/actenon-kernel/blob/main/docs/THE_EXECUTION_GAP.md) — the canonical problem statement
- [Kernel Guarantees](https://github.com/Actenon/actenon-kernel/blob/main/docs/KERNEL_GUARANTEES.md) — what the OSS kernel does and does not guarantee
- [Compliance Mapping](https://github.com/Actenon/actenon-kernel/blob/main/docs/COMPLIANCE_MAPPING.md) — OWASP LLM/Agentic + NIST AI RMF mappings
- [Insurer Clarity](https://github.com/Actenon/actenon-permit/blob/main/docs/INSURER_CLARITY.md) — three separate questions, honestly answered

## License

Apache-2.0 across all five repositories.
