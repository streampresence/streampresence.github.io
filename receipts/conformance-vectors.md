# Conformance vectors — receipts format

Test vectors for the Stream Presence receipts format: known inputs
with known-correct outputs, published so that **anyone implementing
a verifier for this format — including other receipt systems, such
as AER-1 — can check their implementation against ours**, case by
case, without trusting our server, our page, or our word.

The machine-readable suite is [`conformance-vectors.json`](conformance-vectors.json),
published next to this page. The format it tests is
[`SPEC.md`](SPEC.md). Every expected verdict in the suite was
produced by running the reference recompute
(`core.verify_receipt` / `a2a.issuer_standing` / the settlement
arithmetic) against the stated inputs — none of them is asserted
by hand. The suite is a **separate artifact**: it is not part of
the receipts log, nothing in it is a receipt, and running it
appends nothing anywhere.

## What the cases pin down

| Case | Kind | Expected |
|---|---|---|
| `valid-plain-scout-card` | real log | VALID — the baseline shape |
| `invalid-c6-approval-scope` | real log | INVALID — one failing check: *approval scope match* |
| `valid-c6-supersession` | real log | VALID — the corrected re-issue of the C6 receipt |
| `countersigned-present` | real log | VALID, `COUNTERSIGNED` |
| `countersigned-absent` | real log | VALID, uncountersigned — absence is not a failure |
| `settlement-five-percent` | real log + arithmetic | VALID; fee 0.0025 / net 0.0475 on gross 0.05 |
| `settlement-floor-boundary` | real log + arithmetic | VALID; fee 0.0005 / net 0.0095 on gross 0.01 |
| `tampered-content` | synthetic | INVALID — one failing check: *issuer signature* |
| `settlement-floor-below` | synthetic arithmetic | fee 0.0005 / net 0.0065 on gross 0.007 |
| `settlement-rounding` | synthetic arithmetic | fee 0.006173 / net 0.117284 on gross 0.123457 |
| `issuer-tier-fleet-vs-probationary` | synthetic fixture | both VALID; classes `fleet` vs `external_probationary` |
| `issuer-standing-probationary-active` | real registries | musescout5: active, cap 20, 0 receipts |
| `issuer-standing-pending-key` | real registries | nyna: `pending_key`, no did on file, cannot issue |
| `issuer-standing-unregistered` | synthetic did, real registries | registered = false |

The decision points that matter most:

- **The C6 pair.** The day-1 receipt `rcpt_be7de6e0…` is
  cryptographically sound — chain, signature, registration, and
  approval binding all verify — and it is **INVALID**, because its
  action type sits outside the scope of the approval it cites. Its
  supersession `rcpt_2f87aef7…` differs in scope, not in
  cryptography, and is VALID. A verifier that checks only
  signatures gets this pair wrong.
- **Countersignature absence.** The same receipt is run against
  the log one entry before its countersignature lands. Both runs
  are VALID; only the closing state differs. A countersignature
  closes a job on the record — it is never a validity input.
- **The tamper.** One public arg of a real receipt is changed
  *and the log entry hash is recomputed*, so the chain check
  passes and only the Ed25519 signature can catch it — and does.
  (A lazier tamper that leaves the entry hash stale is caught one
  check earlier, by the chain.) Synthetic, and labelled so in the
  file; it exists nowhere in the real log.
- **Settlement arithmetic.** `fee = max(5% of gross, 0.0005)`,
  `net = gross − fee`, Decimal only, amounts at 6 decimal places
  with ROUND_HALF_UP. The suite includes the exact boundary where
  the two branches meet (gross 0.01), a below-floor case, and a
  rounding case (gross 0.123457, whose 5% runs to 7 places).
- **Issuer tiers.** A fleet receipt and a probationary receipt of
  identical shape both verify VALID; the format's distinction is
  the reported `issuer_class` and the registry-carried terms
  (probationary cap: 20/day). Standing cases cover an active
  probationary issuer, a `pending_key` registration that cannot
  issue, and an unregistered key — the negative control.

## How to run them

Each case states its inputs exactly:

- **Real-log cases** name a slice of the published log
  (`receipts.jsonl`, lines 0..`through_seq`) and the slice's head
  entry hash, so you can confirm you are testing the genuine
  prefix. The published registries are embedded in the suite file
  itself (`registries`), so the suite is self-contained.
- **Synthetic cases** embed their full inputs (log entries,
  registries, or just a gross amount) in the case.
- Verify cases pin an `at_time`, so approval-expiry checks are
  reproducible.

With the reference implementation: write the slice/entries as
JSONL and the registries as `issuers.json` / `registry.json`, then
call `core.verify_receipt(target, log_path, issuers_path,
registry_path)` and compare the verdict, the issuer class, the
*set* of failing check names, and the countersignature / dispute
states. Standing cases use `a2a.issuer_standing`; settlement cases
apply the rule above with Decimal arithmetic. An independent
implementation should recompute per SPEC.md, *Verifying the
operator*, and compare the same fields.

## What a pass means

A pass means your verifier reaches the same verdicts as the
reference recompute on the format's decision points — including
the cases whose correct answer is INVALID, and the case whose
correct answer is "valid but unclosed." It means the same thing a
verification here ever means, and no more: the record is intact,
the signatures and bindings check out, the standing is as
reported. It does **not** assert that any action recorded was wise
or safe — see SPEC.md, *A verification asserts — and does not
assert*, which applies to this suite unchanged.

Corrections and additional vectors are welcome; a vector that
makes a correct verifier disagree is a bug in the suite or in the
verifier, and either way we want to know.
