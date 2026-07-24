# How we machine-verify every claim in our README

*Actenon engineering — 2026-07-24*

Every repo in the Actenon ecosystem carries this badge:

[![claims: machine-verified](https://img.shields.io/github/actions/workflow/status/Actenon/actenon-kernel/verify-claims.yml?branch=main&label=claims%3A%20machine-verified)](https://github.com/Actenon/actenon-kernel/actions/workflows/verify-claims.yml)

It is not decoration. It links to a CI workflow named `verify-claims.yml`
that fails — on every PR, every push to `main`, and once a day — if any
factual claim the README makes stops being true. This post explains why we
built it, what it checks, and what it has already caught.

## Why

Actenon's product is a verifiable claim: *an agent should not be able to
take a consequential action without signed authority for that exact action,
and every action should leave proof a third party can check without
trusting us.* A project whose pitch is "verifiable claims" cannot ask you
to take its README on faith. An early external review of this ecosystem
found exactly the failure you would predict: prose that had drifted ahead
of the engineering. The fix was not better prose discipline. Prose
discipline does not survive contact with a Tuesday afternoon. The fix was
making the README a *tested surface*, the same way the code is.

The principle: **if you can state it, you can usually assert it.** A claim
that cannot be asserted by a machine gets rewritten until it can — or
deleted.

## What the gates check

Each repo verifies its own claims; here are the ones a security reviewer
should care about.

**"Zero runtime dependencies."** `actenon-scan` and `actenon-protocol` both
badge this. The gate reads `[project].dependencies` out of `pyproject.toml`
and fails if the list is non-empty. Nobody gets to add "just one small dep"
without turning a public badge red.

**"Zero network calls during verification."** The kernel's core neutrality
claim. There is a test file (`tests/test_neutrality.py`) that proves it,
and the gate *runs it* — a badge that links to a test nobody executes is
just prose with extra steps.

**"51 conformance vectors."** Counts rot faster than any other claim. The
kernel's gate does not trust the number in the README: it runs
`actenon-kernel conformance run`, parses "Ran N test(s)" from the runner's
own output, and compares N against both the badge and the prose. The
protocol repo does the same for its "129 hash-locked test vectors" — by
counting the vector files on disk. Conformance counts are generated, never
hand-written.

**"pip install actenon-kernel" works.** Every install command in every
README is resolved against the live registry, daily — because PyPI state
can change with no PR to catch it.

**The ecosystem table.** The cross-repo table that appears in every README
is rendered from one `ecosystem.yaml` in the protocol repo and checked
byte-for-byte in every consumer. Five READMEs, one source of truth.

**Readiness honesty.** The most unusual one. `actenon-cloud`'s readiness
doc rates production deployment **Red**, and its gate fails if that rating
is ever changed — moving out of Red requires editing the gate itself in the
same PR, in public. We machine-enforce our own *negative* claims, because a
trust vendor that quietly deletes its known limitations is worse than one
that never listed them.

**Version coherence.** A sibling gate asserts that `pyproject.toml`, the
newest release tag, and the version PyPI actually serves agree in every
repo. Its failure message is a rule, not a shrug: *"Never lower a version
to pass a check — bump forward instead."*

## What it has caught

This is not theoretical. In the last review cycle alone:

- A PR "fixed" a version mismatch by **rolling the package version
  backwards** past what PyPI already served. The old one-directional gate
  missed it; the three-way gate now fails that direction with the message
  above.
- The base install of the kernel — the verifier — **could not verify the
  Ed25519 proofs the reference broker mints in production**. 15 of 33
  conformance tests silently skipped. The zero-skip assertion in the
  kernel's invariants gate now makes that class of gap a hard CI failure.
- A dependency restructure upstream broke three consumer repos' CI within
  minutes of publishing — and the gates, not a user, were what caught it.
- The badge itself. The first version of this badge was a *static* green
  image sitting above the sentence "if a claim drifts, the badge goes red."
  A hardcoded badge can never go red. It now renders live workflow status.
  We caught it by checking our own advert against its own standard, which
  is the only reason any of this works.

Each of these is written up, unedited, in the repos' `FINDINGS.md` files.
Findings are append-only: when a conclusion is reversed, a resolution block
is added and the original reasoning stays. The trail is the point.

## How to do this in your project

1. **One workflow, one copy of each check.** Ours is
   [`verify-claims.yml`](https://github.com/Actenon/actenon-kernel/blob/main/.github/workflows/verify-claims.yml)
   in every repo. Two gates with different rules is how drift survives.
2. **Parse reality, compare to prose.** Read the dependency list from the
   package metadata. Ask the test runner how many tests it ran. Resolve the
   install command against the registry. Never grep the README for the
   claim and call it verified.
3. **Schedule it daily.** Registries, sibling repos, and published packages
   change without sending you a PR.
4. **Make failure messages teach the rule.** "pyproject.toml is BEHIND
   PyPI. Never lower a version to pass a check — bump forward instead" has
   already stopped one bad fix. "Check failed" teaches nothing.
5. **Gate your negative claims too.** The checks that keep our limitations
   documented are worth more to a reviewer than the ones that keep our
   features advertised.

If the badge is green, the README is telling the truth. If a claim drifts,
the badge goes red before a human notices — including us.
