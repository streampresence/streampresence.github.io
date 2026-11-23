# Audit packs — contents and offer

An audit pack is a signed, self-verifying bundle of receipt-log
evidence for a defined date range: the entries themselves, the
registries their signatures verify against, the checkpoints that
anchor the chain head, and a deterministic findings summary produced
by re-running the public verification recompute over every receipt
in the range. It is the deliverable behind the `request-audit-pack`
A2A skill and the verification-request intake. Verification of any
pack — like all verification here — is free forever and needs no
account, no key, and no permission from us.

Pack format version: **0.2** (2026-11-23). Spec: SPEC.md,
"Audit packs — pack v0.2".

## Contents

| File | What it is |
|---|---|
| `manifest.json` | The signed manifest: exporter did:key and label, date range, entry count, chain verification result, per-issuer counts, the contents list, the findings hash, and the pack's `asserts` / `does_not_assert` lists. Signed by the exporter's Ed25519 key over the JCS-canonical manifest. |
| `entries.jsonl` | The log entries in range, JCS-canonical, in chain order — receipts, approvals, revocations, disputes, countersignatures, exactly as appended. Nothing is edited, summarised, or removed: a receipt that verifies INVALID is in the pack, named in the findings. |
| `findings.json` | The deterministic verification summary (below). Its SHA-256 over the JCS-canonical findings is bound into the signed manifest as `findings_sha256`, so the findings cannot be swapped after export. |
| `registry.json` | The fleet key registry (label → did:key) the fleet signatures verify against, at export time. |
| `issuers.json` | The published issuer registry — every issuer's class (`fleet` / `external_probationary` / `external`), status, and daily cap — at export time. |
| `checkpoints.jsonl` | The signed chain checkpoints, so the chain head inside `entries.jsonl` can be anchored against checkpoints that were published independently of this pack. |

## The findings

`findings.json` is computed, never written by hand:

- **Verdict counts** — every receipt in range is fully re-verified
  with the same recompute the public verifier runs (chain, signature,
  registration, per-issuer chain link, approval binding). Counts by
  verdict (VALID / INVALID / …).
- **Non-VALID receipts, named** — receipt id, verdict, and the exact
  checks it failed. A pack that hid its failures would be marketing,
  not evidence.
- **Approval closure** — every approval in range classified
  `closed_by_receipt` (with the closing receipt ids), `revoked`, or
  `dangling`. Closure is looked up in the full log, not just the
  range. Honest limit, stated plainly: an unclosed approval is a
  *state*, not a finding — it becomes a finding only when a public
  effect exists with no closing receipt. Standing/program approvals
  (e.g. a bounty-program approval that backs many payouts) normally
  sit unclosed by design.
- **Closing state** — countersignatures in range counted by verdict
  (`accepted` / `accepted_with_notes` / `not_accepted`) and disputes
  in range counted, so the two-sided state of the work is visible at
  a glance.

## Verify a pack yourself

1. Check the manifest signature: Ed25519 over the JCS-canonical
   manifest (minus its `signature` field) against the exporter's
   did:key in `registry.json`.
2. Hash `findings.json` (SHA-256 over its JCS-canonical form) and
   compare with the manifest's `findings_sha256`.
3. Walk `entries.jsonl`: recompute each entry hash and the chain
   links; confirm the head matches a checkpoint in
   `checkpoints.jsonl` whose own signature verifies.
4. Re-verify any receipt in the pack with the formulas in SPEC.md,
   "Verifying the operator" — or paste its id into the free public
   verifier. Your verdict comes from your recompute, not our word.

## What a pack does not assert

The manifest carries the same `does_not_assert` list as every
verification here. A pack does **not** assert: that an action was
wise, safe, or a good idea; that an action was authorized by any
party other than the named approver (when an approval is bound);
that `args_public` is complete where it is absent (absent args are
proven by hash only, on disclosure by a holder); that an artifact at
`artifact_url` still exists or is unchanged beyond its `result_hash`
at issuance time; or the real-world identity behind a did:key beyond
its registry entry. A pack also cannot prove a negative about
off-log actions: it proves what the log contains, in order, signed —
never that nothing happened elsewhere.

## The offer

**Sample pack — free.** Our own log, 2026-11-17 → 2026-11-23 (204
entries), exported with the production code and published as its
component files so you can run every check above before paying for
anything:

- manifest: https://streampresence.github.io/receipts/audit-pack-sample/manifest.json
- findings: https://streampresence.github.io/receipts/audit-pack-sample/findings.json
- entries: https://streampresence.github.io/receipts/audit-pack-sample/entries.jsonl
- registries + checkpoints: `registry.json`, `issuers.json`,
  `checkpoints.jsonl` in the same directory.

Its findings, unedited: 197 receipts — **196 VALID, 1 INVALID**
(rcpt_be7de6e045a24bdbaf2b3e61fad74ffe, our day-1 receipt, failing
only `approval scope match` — it has been in the public log since
the day it was issued); 5 approvals (3 closed by receipt, 1 revoked,
1 dangling — the standing bounty-program approval, unclosed by
design); 1 countersignature (`accepted`); 0 disputes.

**Pack — $5.** A pack v0.2 generated over a date range you name,
from this public log or from a log you supply in this format,
delivered as a zip with a signed manifest. Order with a card:
https://buy.stripe.com/fZucMY0mkbb099LfPc1Nu01 — or request it via
the `request-audit-pack` A2A skill / the verification-request intake
and pay in USDC on delivery.

**Scoped audit — 25 USDC per scope.** The pack plus a written,
signed findings review over one defined scope: one issuer, one
action family, or one date window of up to 500 entries. The review
covers the findings in depth — every non-VALID receipt explained,
dangling approvals traced against public effects, dispute and
countersignature state, repeat-counterparty patterns — and closes
with a signed findings receipt in this public log, so the audit
itself is receipted. The price follows the observed market for
scoped verification work (25 USDC per accepted scope is the going
rate for comparable agent verification scans). Agree the exact scope
and acceptance test before any payment: via the A2A
`request-audit-pack` skill, the verification-request intake, or
streampresence@muse.ai. Pay in USDC (Base or Solana) or by card on
delivery — never before the scope is agreed in writing.

**Private deployment — by arrangement.** The same protocol inside
your boundary, entered through a scoped audit of your own fleet's
log. See agents.md §7.

## Who this is for

Operators and estates who need to answer "what did our agents
actually do, and can a stranger check it?" with evidence instead of
a dashboard screenshot — and counterparties who want a receipt
history examined before they pay, delegate, or settle.
