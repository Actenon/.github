# Actenon

### Control what AI agents are allowed to do - all the way to execution.

AI agents can now send messages, modify files, update databases, call APIs, deploy software, change permissions and take other actions with real-world consequences.

The problem is no longer just:

> **Can this agent call this tool?**

The harder questions are:

> **What powers does this agent actually have?**  
> **Did those powers change when the code changed?**  
> **Was this exact action authorised?**  
> **Is the action executing now still the action that was approved?**  
> **Has this consequence already happened?**

**Actenon is an open system for discovering, controlling and verifying machine authority.**

Its developer-facing product is **Airlock**.

Airlock discovers consequential powers from an agent's source code, turns those powers into enforceable authority, shows when authority changes, and controls consequential actions at runtime.

---

## The problem: the execution gap

Authentication can prove **who** is asking.

Authorization can decide **what they are generally allowed to do**.

Approval workflows can record **what somebody intended to approve**.

But there is still a gap between:

```text
WHAT WAS INTENDED OR AUTHORISED
                ↓
        agent reasoning
                ↓
        tool / API request
                ↓
WHAT IS ACTUALLY ABOUT TO HAPPEN
```

That is the **execution gap**.

For consequential systems, the execution boundary needs to answer a stricter question:

> **Is this exact action, against this exact target, with these exact constraints, still within the authority that was reviewed and approved?**

Actenon carries authority from source code and human review all the way to the execution boundary.

---

# Airlock

**Airlock is the product developers use.**

It connects five things that are normally separate:

```text
SOURCE CODE
    ↓
DISCOVER POWERS
    ↓
REVIEW AUTHORITY
    ↓
ENFORCE AT RUNTIME
    ↓
VERIFY THE EXACT ACTION
    ↓
RECORD WHAT HAPPENED
```

The goal is simple:

> **A consequential action should not happen simply because an AI agent decided to attempt it.**

---

## 1. Discover what the agent can do

Airlock uses **Actenon Scan** to analyse source code and identify consequential capabilities.

For example:

```text
http.post
    api.example.com/orders

github.issue.comment
    acme/support

filesystem.write
    ./reports/**

database.delete
    production/customers
```

Where possible, authority is tied back to the source location that introduced it.

If a target cannot be determined safely, Actenon does not silently widen it to `*`.

Unknown authority remains unresolved or constrained until it can be reviewed safely.

---

## 2. Show when an agent gains new powers

Ordinary code review answers:

> What code changed?

Airlock adds another question:

> **What can the agent do now that it could not do before?**

Example:

```diff
AIRLOCK AUTHORITY DIFF

+ github.repo.delete
  acme/payments

+ database.delete
  production/customers

- github.issue.read
  acme/support
```

A code change that silently increases an agent's authority becomes visible before deployment.

This can also run in CI so a pull request can answer:

> **Does this change give the agent new consequential powers?**

---

## 3. Turn discovered powers into authority

Discovered capability is not automatically permission.

Airlock turns reviewed powers into explicit authority.

Authority can be limited by things such as:

```text
action
target
resource
parameters
time
principal
budget
environment
```

For dynamic applications, authority can be bounded rather than reduced to an unsafe wildcard.

For example:

```text
filesystem.write

allowed:
    ./reports/**
```

Then:

```text
./reports/result.md
    → allowed

/etc/passwd
    → refused
```

The important rule is:

> **Dynamic does not mean unlimited.**

---

## 4. Decide how much autonomy an agent gets

Not every authorised action needs the same level of autonomy.

A policy can distinguish between:

```text
ALLOW
REQUIRE APPROVAL
DENY
```

For example:

```text
refund ≤ £500
    → autonomous

refund £500–£10,000
    → human approval

refund > £10,000
    → deny
```

Authority answers:

> **May this agent ever do this?**

Policy answers:

> **May it do this autonomously right now?**

Both must pass.

---

## 5. Bind approval to the exact action

A human approval should not become a reusable blank cheque.

If somebody approves:

```text
action:
    refund

payment:
    txn_123

amount:
    £5,000
```

then changing the attempted action to:

```text
amount:
    £9,000
```

requires different authority.

Actenon binds execution proof to the consequential action being attempted rather than merely to the fact that somebody approved something earlier.

---

## 6. Verify at the execution boundary

This is where **Actenon Kernel** operates.

Before a protected side effect executes, the Kernel verifies the proof against the exact execution request.

The proof can bind:

```text
action
target
tenant
subject
audience
scope
parameters
time window
single-use nonce
authority reference
```

If the proof does not match the action about to execute:

```text
REFUSE
```

The Kernel does not make the business decision.

It does not decide whether deleting a database row is sensible.

Its job is narrower and more important:

> **Verify that the action reaching the execution edge is the action that was actually authorised.**

---

## 7. Keep production credentials away from the agent

Where Airlock brokers a protected resource, the agent does not need to hold the real production credential.

Instead:

```text
AGENT
  ↓
requests consequential action
  ↓
AIRLOCK / PERMIT
  ↓
authority checked
  ↓
KERNEL
  ↓
proof verified
  ↓
credential released to protected boundary
  ↓
exact side effect
```

A denied action should therefore mean:

```text
credential released: NO
execution occurred: NO
```

This reduces the value of simply persuading the model to make a different decision.

---

## 8. Prevent duplicate consequential effects

An action can be authorised and still be unsafe to execute twice.

For example:

```text
refund customer
      ↓
provider commits refund
      ↓
network response is lost
      ↓
agent retries
```

The second request must not automatically become a second refund.

Actenon is designed to treat the real-world **effect** separately from the API call that requested it.

Protected effects can be reserved before execution so concurrent or repeated attempts do not knowingly perform the same consequence twice.

---

## 9. Do not pretend an unknown outcome is a failure

Distributed systems cannot always know whether a remote effect happened.

After execution begins, a timeout may mean:

```text
the action failed
```

or:

```text
the action succeeded
but the response was lost
```

Those are not the same state.

Actenon's execution model distinguishes outcomes such as:

```text
COMMITTED

NOT_EXECUTED

AMBIGUOUS
```

An ambiguous effect must not be blindly retried until its real-world state has been reconciled.

---

## 10. Leave verifiable evidence

Consequential attempts produce structured evidence.

Depending on the boundary, evidence can include:

```text
source version

authority manifest

principal

approval

action

target

constraints

proof

credential decision

execution outcome

timestamp
```

Actenon's proof and receipt architecture is designed so evidence can be verified independently rather than requiring blind trust in the agent that performed the action.

---

# How Actenon fits together

```text
                         ACTENON AIRLOCK
                               │
              ┌────────────────┼────────────────┐
              │                │                │
           DISCOVER          CONTROL          EXECUTE
              │                │                │
              ▼                ▼                ▼
            SCAN             PERMIT           KERNEL
              │                │                │
              └──────────────┬─┴────────────────┘
                             │
                          PROTOCOL
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
              Python         Go          Rust
                        TypeScript
```

### Airlock

The developer-facing product.

Discovers authority, shows authority changes, coordinates approval and runtime enforcement, and produces execution evidence.

### Scan

The authority discovery engine.

Analyses source code to identify consequential actions, targets and provenance.

### Permit

The authority broker and policy decision layer.

Handles grants, constraints, approvals, delegation, budgets and credential brokering.

### Kernel

The execution-edge verifier.

Checks whether the proof presented for an action actually authorises the action reaching the protected boundary.

### Protocol

The vendor-neutral wire contract.

Defines portable proof, receipt, refusal and execution semantics independently of a particular framework or programming language.

### SDKs

Independent implementations allow protected services written in languages such as Python, TypeScript, Go and Rust to verify Actenon proofs at their own execution boundary.

---

# The chain Actenon protects

The central Actenon invariant is:

```text
POWER FOUND IN CODE
        =
POWER REVIEWED
        =
AUTHORITY APPROVED
        =
ACTION POLICY EVALUATED
        =
ACTION PROOF AUTHORISED
        =
ACTION RESOURCE ACCEPTED
        =
CONSEQUENCE ACTUALLY EXECUTED
```

If those diverge, the protected action should not proceed.

---

# Example

Suppose an agent currently has:

```text
github.issue.read
github.issue.comment
```

A pull request adds:

```text
github.repo.delete
```

Airlock can surface:

```text
AIRLOCK AUTHORITY DIFF

NEW POWER

+ github.repo.delete
  acme/payments

Status:
BLOCKED UNTIL APPROVED
```

If the running agent later attempts that operation without the necessary authority:

```text
decision:
    DENY

credential released:
    NO

execution:
    NOT_EXECUTED
```

The fact that the model decided to delete the repository is not sufficient authority to delete it.

---

# What Actenon is designed for

Actenon matters when software can create real-world consequences.

Examples include agents that can:

- modify or delete data
- send email or messages
- create or modify GitHub resources
- deploy software
- execute shell commands
- change infrastructure
- modify files
- grant or revoke access
- interact with production APIs
- issue payments or refunds
- perform database mutations
- trigger business workflows

If an agent only reads information and produces text, much of this machinery may be unnecessary.

The value appears when an agent is allowed to **act**.

---

# Trust boundary

Actenon does not claim that a Python library can magically contain hostile arbitrary code.

The strongest execution guarantee requires the protected execution edge to actually be the route to the resource.

In particular:

> **The protected edge must be the only permitted route to the resource, production credentials must remain outside the agent and be released only after verification, and the agent must not have an alternate path that bypasses enforcement.**

If an agent already possesses an unrestricted production credential or another direct route to the resource, it may bypass any upstream enforcement system.

For stronger threat models, execution therefore needs appropriate process, container, operating-system or resource-boundary isolation in addition to authority enforcement.

Actenon is explicit about that boundary.

---

# Principles

1. **Consequential capability is not authority.**  
   Being technically able to invoke something does not mean an agent is permitted to invoke it.

2. **Code changes can be authority changes.**  
   A pull request can silently give an agent new powers even if nobody intended to change its permissions.

3. **Dynamic authority must still be bounded.**  
   Unknown values must never silently become wildcards.

4. **Approval must bind to the action.**  
   Changing the consequential action after approval requires different authority.

5. **The execution edge is the final trust boundary.**  
   Upstream controls matter, but the real side effect must still be independently checked.

6. **The agent should not hold standing production credentials.**

7. **Unknown is not failed.**  
   An uncertain execution outcome must not trigger a blind retry.

8. **One real-world effect should not knowingly happen twice.**

9. **Refusal is better than false confidence.**

10. **Evidence should be independently verifiable.**

11. **Open protocols and conformance matter more than vendor pedigree.**

---

# Start here

The long-term developer experience is deliberately simple:

```bash
airlock init
airlock diff
airlock run
```

Airlock is currently under active development.

Repository:

**[`Actenon/actenon-airlock`](https://github.com/Actenon/actenon-airlock)**

You can also explore the underlying open components:

- **[`actenon-scan`](https://github.com/Actenon/actenon-scan)** — discover consequential authority in source code
- **[`actenon-permit`](https://github.com/Actenon/actenon-permit)** — authority, policy and credential brokering
- **[`actenon-kernel`](https://github.com/Actenon/actenon-kernel)** — execution-edge proof verification
- **[`actenon-protocol`](https://github.com/Actenon/actenon-protocol)** — vendor-neutral proof and execution contracts
- **[`sdk-go`](https://github.com/Actenon/sdk-go)** — Go protected-boundary verifier
- **[`sdk-rust`](https://github.com/Actenon/sdk-rust)** — Rust protected-boundary verifier

---

## The idea in one sentence

> **Git tells you what code changed. Actenon tells you what your agent can do now — and carries that authority all the way to the consequential action.**

---

## License

Actenon's open components are released under the Apache-2.0 licence unless otherwise stated.
