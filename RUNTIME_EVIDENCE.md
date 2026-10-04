# RUNTIME_EVIDENCE

Network: GenLayer StudioNet, chain 61999. Source SHA-256: `f5181ea8331abbfe69c7765705afdb7b43ba419a512696e956f762dcc640e63a`.
Every row: wallet, method, exact input, expected, tx hash, execution result, post-state from a view.
"EP" is the equivalence-principle output (the label the validators agreed on). A screenshot or a FINALIZED status alone is not evidence of execution.

## The two pairs that carry the concept

| Pair | What it shows | Status |
|---|---|---|
| **#2 vs #6** | the same `reject_instalment(1)` by the same buyer: three rows change on E2, one row on S2 | PASS — `0xe3d13706…0ba6` vs `0x4eb2c0c2…01de` |
| **#3 vs #7** | the same `accept_instalment(2)`: reverts on E2, succeeds on S2 | PASS — `0xad05a870…f399` vs `0x839c8fcc…3f2d` |

## Intelligent Contract — 9 rows, 10 transactions

Contract address: [`0x4552f56bA6BcEB14f5b2a68B1B7214fa00D24439`](https://explorer-studio.genlayer.com/address/0x4552f56bA6BcEB14f5b2a68B1B7214fa00D24439) · deploy tx `0xefd3a085c6c25dc3daf596d47024f278034a8108bcf0153c88da5d9f05aab513` (FINALIZED · SUCCESS)
Supplier wallet `0x923a09d0D6e5C242e36C3c1D2071835917cC0bDF` · buyer wallet `0x76DD809f34e0B72d9339bc509e1E19FaFEB445c2` · every contract
`part_count = 3`, label `the Buyer`.

| # | Wallet | Method and input | Expected | Tx hash | Result | Post-state | Status |
|---|---|---|---|---|---|---|---|
| 1 | supplier | `open_contract(buyer, "the Buyer", 3, E2)` | ENTIRE, WHOLE, 3 × PENDING (MV-1a) | `0x99c29e5b46bf737c2f3012c90916b6d38135c9b0b639344dff184037e19986d6` | FINALIZED · SUCCESS · EP `ENTIRE` | `get_contract`: ENTIRE, WHOLE, OPEN, next_index 1, pending 3 | PASS |
| 2 | buyer | `reject_instalment(<E2 id>, 1, "Damaged")` | `[REJECTED, RELEASED, RELEASED]`, ENDED (MV-2a) | `0xe3d13706c74815ed7c3c76bcd86a1738b98b25fa75b958cb5c5addc388b80ba6` | FINALIZED · SUCCESS | `get_instalments`: REJECTED ("Damaged"), RELEASED, RELEASED | PASS |
| 3 | buyer | `accept_instalment(<E2 id>, 2)` | revert *The arrangement ended when an instalment was rejected* | `0xad05a870f76893b73e0f5a8147ad8a8c86b31b7fa221d81ad6b8130912d1f399` | ERROR · `[rollback] The arrangement ended when an instalment was rejected` (validators agree) | | PASS (expected revert) |
| 4 | supplier | `contest_rejection(<E2 id>, 1, "Goods were fine")` | success; no state changes | `0x10fb396f28f24ca34f23384c5df6322281db26addff05d29e79e1a52cb0923d2` | ACCEPTED · SUCCESS | | PASS |
| 5 | supplier | `open_contract(buyer, "the Buyer", 3, S2)` | SEVERABLE, SPLIT (MV-1b) | `0xfbaaa777470cf099bb2ac368557df2fbdd37f1218284d8e25cdebc473ed35761` | ACCEPTED · SUCCESS · EP `SEVERABLE` | `get_contract`: SEVERABLE, SPLIT, OPEN, pending 3 | PASS |
| 6 | buyer | `reject_instalment(<S2 id>, 1, "Damaged")` | `[REJECTED, PENDING, PENDING]`, OPEN (MV-2b) | `0x4eb2c0c2a7d937b1af6efd69f7346b588b65939de1114542694f1f7334ba01de` | FINALIZED · SUCCESS | only instalment 1 changed (confirmed by read R1 below) | PASS |
| 7 | buyer | `accept_instalment(<S2 id>, 2)` | success, instalment 2 ACCEPTED | `0x839c8fcc62f32b8f40f80340dd7de903bb395e9a33dfa3fab06930cb39cc3f2d` | FINALIZED · SUCCESS | | PASS |
| 8 | buyer | `accept_instalment(<S2 id>, 3)` | instalment 3 ACCEPTED, COMPLETE | `0x0ad2c2e0c7ab17c33e8fc46a6747e454f0d33aa774c6ca412cbaf4ae7af66387` | ACCEPTED · SUCCESS (5/5 agree) | `get_contract`: COMPLETE, next_index 4, pending 0, accepted 2, rejected 1, released 0 | PASS |
| 9 | supplier | `open_contract` E5, then `open_contract` S5 | ENTIRE, then SEVERABLE (MV-3) | E5 `0xd0e7cec5dc6eec9eaaaa3d4e9ca44d4acd725a15ac99b6db676509033884c8a6` · S5 `0xe5cc687168c1d590153b03c4b94fb1330d525a43a7f588f46ba27efddafa9668` | both ACCEPTED · SUCCESS · EP `ENTIRE` / `SEVERABLE` | | PASS |

E2 = `A defect in any one shipment entitles you to refuse the rest.`
S2 = `A defect in any one shipment entitles you to refuse that shipment only.`
E5 = `Rejecting one delivery ends the arrangement for every delivery still to come.`
S5 = `Rejecting one delivery has no effect on the deliveries still to come.`

Must-verify: MV-1 PASS · MV-2 PASS · MV-3 PASS. Run date 2026-10-04, GenLayer Studio, Normal (Full Consensus).

## Read-back after the run (views, no transaction)

| # | View | Input | Returned |
|---|---|---|---|
| R1 | `get_instalments` | S2 id, 0, 10 | `REJECTED` ("Damaged") · `ACCEPTED` · `ACCEPTED` — the rejection on S2 touched one row |
| R2 | `get_instalment` | E2 id, 1 | `REJECTED`, reject_note "Damaged", contest_note "Goods were fine" — the contest is recorded and changed no state |
| R3 | `get_contract` | E2 id | ENTIRE, WHOLE, **ENDED**, next_index 2, pending 0, accepted 0, rejected 1, **released 2** |

E2 id `4b367f19bff24dec266fb72f044e62441226b241f20a5aafa61d56beef5bb724` ·
S2 id `e3b0109bbddbb37d7d96d945fd5fc8e76a019d7810bbda72a285a98922c2fa98`

## Notes

- Row #3 is an expected revert; the validators agree on the rollback reason shown.
