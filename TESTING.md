# TESTING

```
COMPILE PASS ≠ RUNTIME PASS
SUBMITTED ≠ ACCEPTED ≠ FINALIZED ≠ EXECUTION SUCCESS ≠ POSTCONDITION PASS
```

## On-chain (GenLayer StudioNet, 2026-10-04)

One run of 10 transactions with two wallets (supplier, buyer), every contract with 3 instalments and the
label `the Buyer`, in GenLayer Studio (Normal / Full Consensus). Every row behaved as expected. Full table with
inputs, hashes, results and read-back state: `RUNTIME_EVIDENCE.md`.

| Check | Rows | Result |
|---|---|---|
| **MV-1** — sentences identical up to the last words get opposite readings | E2 → `ENTIRE`/`WHOLE`, S2 → `SEVERABLE`/`SPLIT` | PASS |
| **MV-2** — the same rejection touches three rows or one | E2 → REJECTED · RELEASED · RELEASED (ENDED); S2 → only instalment 1 REJECTED | PASS |
| the tooth | `accept_instalment(2)` reverts on E2 (*The arrangement ended when an instalment was rejected*) and succeeds on S2 | PASS |
| contest after the end | `contest_rejection` on E2 succeeds; nothing else changes | PASS |
| completion | S2 ends `COMPLETE`: rejected 1, accepted 2, released 0 | PASS |
| **MV-3** — both open with "Rejecting one delivery", opposite readings | E5 → `ENTIRE`, S5 → `SEVERABLE` | PASS |

Test cases used:

| Case | Text | Expected |
|---|---|---|
| E2 | A defect in any one shipment entitles you to refuse the rest. | ENTIRE |
| S2 | A defect in any one shipment entitles you to refuse that shipment only. | SEVERABLE |
| E5 | Rejecting one delivery ends the arrangement for every delivery still to come. | ENTIRE |
| S5 | Rejecting one delivery has no effect on the deliveries still to come. | SEVERABLE |

## Offline (before deploy)

- **genvm-linter** `lint`, `schema`, `typecheck`: pass; 9 methods (4 write, 5 view), no type errors.
- **Direct Mode tests** (`genlayer-test`, the real py-genlayer v0.2.16 SDK, model mocked): 51 passed — the
  propagation rule over 10 shape × position cells, the same-command comparison, all 20 revert strings each with a
  dedicated test, check order, id formula, fail-safe, validator function, prompt fence, views.
- **Rubric gate**: no word or word pair separates the two test classes, and the rubric shares no content word
  with any of the 10 cases.
- **Calldata**: every call used on-chain is between 105 and 167 bytes, under the 255-byte RPC limit.

## What this run does NOT prove

- Only 4 of the 10 semantic cases were labelled on-chain (the two strongest pairs), one run each; label
  stability across repeated runs or other validator sets is not measured.
- Terms longer than about 165 characters (the contract allows 600) were not sent.
- Prompt-injection resistance rests on the fence and the door check; no adversarial model run was done.
