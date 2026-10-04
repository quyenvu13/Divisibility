Divisibility does not decide whether a delivery was faulty — the buyer says so, and the supplier can contest it. It asks whether turning down one instalment lets the buyer walk away from the rest, and it turns that answer into whether one rejection touches one row or every row still to come.

# Divisibility

GenLayer Intelligent Contract · StudioNet (chain 61999) · py-genlayer v0.2 (`# v0.2.16`)

| | |
|---|---|
| Contract | `contracts/Divisibility.py` |
| Address (StudioNet) | `0x4552f56bA6BcEB14f5b2a68B1B7214fa00D24439` ([explorer](https://explorer-studio.genlayer.com/address/0x4552f56bA6BcEB14f5b2a68B1B7214fa00D24439)) |
| Deploy tx | `0xefd3a085c6c25dc3daf596d47024f278034a8108bcf0153c88da5d9f05aab513` |
| Source SHA-256 | `f5181ea8331abbfe69c7765705afdb7b43ba419a512696e956f762dcc640e63a` |
| Tested on-chain | `RUNTIME_EVIDENCE.md` — 10 transactions, one hash per row, all as expected |
| Testing summary | `TESTING.md` |

## What it does

- A supplier opens a supply contract for a named buyer, split into 2–5 instalments. Validators read the terms
  **once**: `ENTIRE` (one indivisible bargain) or `SEVERABLE` (independent bargains). The answer is stored as
  `outcome` and frozen as a shape, `WHOLE` or `SPLIT`.
- The buyer handles instalments in order: accept or reject. **Rejecting always works; the shape decides how far
  it reaches.** In a `WHOLE` contract, rejecting instalment 1 marks it `REJECTED`, marks every later instalment
  `RELEASED` and ends the contract. In a `SPLIT` contract it touches instalment 1 only.
- After a `WHOLE` contract ends, accepting the next instalment reverts with *"The arrangement ended when an
  instalment was rejected"*. In a `SPLIT` contract the same call succeeds.
- The supplier may contest a rejection once; the contest is recorded next to it and changes no state.

**The contract holds no funds or goods** and does not judge whether a rejection was justified.

## Proven on StudioNet

| What | Transactions | Result |
|---|---|---|
| Same sentence up to the last words, opposite readings | E2 `0x99c29e5b…86d6` · S2 `0xfbaaa777…5761` | ENTIRE / SEVERABLE |
| Same `reject_instalment(1)`, same buyer | E2 `0xe3d13706…0ba6` · S2 `0x4eb2c0c2…01de` | three rows change (REJECTED · RELEASED · RELEASED) / one row changes |
| Same `accept_instalment(2)` afterwards | E2 `0xad05a870…f399` · S2 `0x839c8fcc…3f2d` | reverted *The arrangement ended when an instalment was rejected* / SUCCESS |
| Both open with "Rejecting one delivery", opposite readings | E5 `0xd0e7cec5…c8a6` · S5 `0xe5cc6871…9668` | ENTIRE / SEVERABLE |

## Methods

| Write | Who | Notes |
|---|---|---|
| `open_contract(buyer_wallet, buyer_label, part_count, text)` | the supplier | 2–5 instalments; the only model call |
| `accept_instalment(contract_id_hex, index)` | the buyer | in order; reverts once a `WHOLE` contract has ended |
| `reject_instalment(contract_id_hex, index, note)` | the buyer | in order; `WHOLE` releases every later instalment and ends the contract |
| `contest_rejection(contract_id_hex, index, note)` | the supplier | once per rejected instalment; works after the end; changes no state |

Views (JSON strings; an unknown id returns `"{}"`, the list view `"[]"`, never a revert): `get_contract`
(all fields including `outcome` and `shape`, plus pending / accepted / rejected / released counts),
`get_instalment`, `get_instalments`, `get_rubric`, `get_limits`. There is no preview or dry-run view.

Contract id: `keccak256("DIVISIBILITY:CONTRACT:V1|" + supplier_lower + "|" + len(text) + "|" + text)`, where
`text` has all whitespace collapsed (Python `" ".join(text.split())`).

## Design choices

- **Fail-safe is `ENTIRE`.** Unclear or unparseable output hands the choice to walk away to the buyer, who did
  not write the terms; nothing propagates unless the buyer rejects. A wrong `SEVERABLE` would hold the buyer to
  the rest of a deal the terms let them drop.
- **One place knows propagation.** `_after_reject` is the only code that sets `RELEASED`, and only for
  instalments after the rejected one; earlier accepted instalments keep their state.
- **No wording game.** The supplier wants `SEVERABLE`; the fail-safe runs against that, and getting
  `SEVERABLE` takes terms that clearly say each instalment stands alone — which the buyer reads first. The text
  is part of the id, so the same terms cannot be reopened for another reading.
- **Prompt fence.** The model sees only the rubric, the buyer label and the terms — not the number of
  instalments, wallets or state. Text or label containing a tag or an answer token is rejected first.

## How to try it in GenLayer Studio

Use **two wallets of your own**: supplier and buyer.

1. Supplier: `open_contract(<buyer>, "the Buyer", 3, "<terms>")` once with terms that make the instalments one
   bargain and once with terms that make each stand alone. Read `get_contract` for `outcome` and `shape`.
2. Buyer: `reject_instalment(<id>, 1, "Damaged")` on both; read `get_instalments`.
3. Buyer: `accept_instalment(<id>, 2)` on both — it reverts on the `WHOLE` contract and succeeds on `SPLIT`.

Terms and label may not contain the words `ENTIRE` or `SEVERABLE` in any letter case ("This is the entire
deal" is rejected). Notes are at most 60 characters; keep terms under about 165 characters so the calldata
stays under 255 bytes.

## Honest limitation

1. **The contract holds no money or goods.** It records the state of each instalment.
2. **A rejection is the buyer's own statement, and the contract does not judge it.** In an `ENTIRE` contract one
   rejection ends the whole contract. The supplier only has `contest_rejection`, and a contest restores nothing.
3. **A wrong `SEVERABLE` is the main risk** — the buyer would be held to a deal the terms let them leave.
4. **A wrong `ENTIRE` costs the supplier the later instalments** — it lands on whoever wrote the unclear terms.
   This is deliberate.
5. **The supplier declares the buyer's wallet;** a second wallet is cheap.
6. **Instalments are handled in order, and `MAX_PARTS = 5`.** An instalment is never accepted automatically — if
   the buyer stays silent, the contract stays `OPEN` forever.

The contract accepts terms up to 600 characters, but `open_contract` at the caps measures 764 bytes of calldata;
only terms up to about 165 characters fit the 255-byte limit, and longer terms would be refused by the RPC.

License: MIT.
