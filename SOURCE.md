# Data Source

Chen Eylon · Ori Lisser

A single live source, chosen after professor feedback on our two original
proposals. The source publishes only a current state: ask it tomorrow about
today and the answer is gone. It is not an existing dataset — the history only
exists if we collect it, which is the point of the project.

A **second, enriching source** will be added in week 7 per the project timeline
(see "Planned second source" below). This file will be updated when that source
is chosen.

---

## Primary source — Shufersal prices across its branches

### What it is

Israel's price transparency law requires every food chain to publish its prices
for every branch and to update them within an hour of a price change. Shufersal
runs its own public portal of timestamped, gzipped files, one per branch, with
no login required. We collect a fixed basket of products from a fixed set of
Shufersal branches on the same daily schedule and track how each product's price
moves, per branch, over the semester.

We chose Shufersal over the other chains for two reasons: its portal is open
(no login, unlike chains served through the shared login platform), and it
operates a large number of branches — which is exactly what our research
question needs, since the comparison is *between branches of the same chain*.

### Research question

How does the **rate of price increases differ from the rate of price
decreases** across different branches of the **same chain** — and from that,
**which branches lead** price changes and which only follow them?

Concretely, we measure three things:

1. **Asymmetry** — within each branch, do prices rise faster/sharper than they
   fall? (the "rockets and feathers" pattern)
2. **Branch comparison** — is that asymmetry stronger in some branches than
   others (e.g. central vs peripheral locations)?
3. **Lead–follow** — when a product's price changes, which branch moves first
   and which align to it a few days later?

Measuring this needs same-moment, per-branch collection over a full semester —
history that does not exist anywhere and cannot be reconstructed after the fact.

### Expected content

Per product, per branch, per collection: barcode, product name, unit, price,
and whether a promotion applies. Each source file carries the branch id and the
timestamp of publication.

### Basket

A fixed basket of 25 **national-brand** grocery products, spread across many
categories (soft drinks, snacks, sweets, dairy, dry goods, household, personal
care) so that it reflects a typical household's shopping rather than one corner
of it. Three principles drive the choice:

1. **National brands only** (not private-label / store brands), so the **exact
   same barcode** exists in every branch — this is what makes a cross-branch
   comparison valid.
2. **Products whose price moves**, since the research question is about price
   *changes* over time. We deliberately **exclude price-controlled items**
   (milk in cartons, white bread, eggs, salt), whose price is frozen by law and
   would contribute no signal.
3. **Diversity across categories**, so the findings are not an artefact of one
   product family.

The basket below was verified against a real `PriceFull` file from branch 001
(Shufersal Sheli Tel Aviv – Ben Yehuda) on 2026-11-17: every product was found
in the live catalogue with a discount-eligible flag. Exact single-unit barcodes
are finalised when `collect.sh` first runs and confirms each product appears in
every chosen branch.

| # | product | barcode | price (₪) |
|---|---------|---------|-----------|
| 1  | Coca-Cola 1.5 L            | 7290110115364 | 47.90 |
| 2  | Sprite Lime 1.5 L         | 7290110115265 | 8.90  |
| 3  | Bamba 80 g (Osem)         | 7290000066318 | 5.90  |
| 4  | Bisli Grill 70 g          | 7290000066141 | 5.50  |
| 5  | Tapuchips natural 50 g    | 7290000178707 | 5.20  |
| 6  | Doritos Grill 70 g        | 7290100850923 | 5.50  |
| 7  | Elite Para chocolate 100 g| 7290000170053 | 9.90  |
| 8  | Klik biscuit 65 g         | 7290112494221 | 8.90  |
| 9  | Nescafé Red Mug 200 g     | 7290000061764 | 29.90 |
| 10 | Cornflakes honey (Dgani Boker) | 7290116538686 | 25.50 |
| 11 | Osem pasta macaroni 500 g | 7290000060217 | 7.90  |
| 12 | Osem ketchup 750 g        | 7290000072623 | 16.50 |
| 13 | StarKist tuna 4×160 g     | 7290005287206 | 26.90 |
| 14 | Emek cheese 15% sliced    | 7290000052311 | 19.90 |
| 15 | Cottage cheese 5% 250 g   | 7290000041445 | 6.40  |
| 16 | Milky 141 g               | 7290005839313 | 5.80  |
| 17 | Danone Bio yogurt 3%      | 7290000408316 | 4.90  |
| 18 | Canola oil 1 L            | 7290000144474 | 17.90 |
| 19 | Round rice 1 kg (Sugat)   | 7290000211985 | 13.90 |
| 20 | Goldstar can 500 ml       | 7290008464024 | 11.90 |
| 21 | Ice cream tub (chocolate) | 4810168069018 | 39.90 |
| 22 | Ariel laundry powder 1.25 kg | 5413149655522 | 19.90 |
| 23 | Fairy dish soap 650 ml    | 4084500853553 | 14.50 |
| 24 | Lily toilet paper 32 rolls| 7290000187938 | 43.90 |
| 25 | Johnson's shampoo 750 ml  | 3574669907729 | 22.90 |

### Branches

Six Shufersal branches — **three in Tel Aviv and three in one peripheral city**
(e.g. Be'er Sheva) — so the comparison is group-vs-group, not branch-vs-branch:

- **Tel Aviv** — branch 001 (Sheli, Ben Yehuda, verified) plus two more Tel Aviv
  branches (e.g. 007 Arlozorov, 011 Nordau).
- **Periphery** — three branches in one southern city, ids confirmed against the
  portal's branch list.

Three branches per group is deliberate: with one branch per side, any difference
could just be that single branch's quirk. With three, we can separate
*within-group* variation (one odd branch) from a real *between-group* difference
(Tel Aviv as a whole behaving differently from the periphery), which is what
keeps the finding unbiased.

### URL

The source of record is the Ministry of Economy's index, which lists the
statutory price portal of every chain:

- https://www.gov.il/he/departments/legalInfo/cpfta_prices_regulations

The Shufersal portal itself:

- https://prices.shufersal.co.il

### Update frequency

The law requires an update within an hour of any price change. We collect
**three times a day** per branch (e.g. morning, midday, evening), so we can see
intra-day timing — which matters for the lead/follow part of the research
question — not just day-to-day movement. Each collection downloads the full
`PriceFull` snapshot (so all basket products are always present) but keeps only
our 25 basket products — see below.

### Expected size

`collect.sh` downloads the full `PriceFull` file per branch but stores **only
the 25 basket products** as a small timestamped file in `raw/`. That is roughly
25 products × 6 branches × 3 times a day ≈ 450 rows per day, a few hundred KB —
so the whole semester stays at a handful of MB, far inside the 500 MB budget.

Trade-off, stated deliberately: filtering at collection time means the basket is
fixed — a product we did not collect cannot be recovered later, because the
unfiltered file is not kept. Since the basket is chosen on purpose and held
constant for the semester, this is an acceptable and intentional choice.

### Key or password

None. Shufersal publishes without a login.

### Limitations

- A product missing from a file may be out of stock or may simply not be carried
  by that branch. We treat the two as different and do not fill gaps.
- Branch coverage and assortment differ between branches, so the basket is
  defined by barcode and any product absent from a branch stays a gap.
- Promotions are expressed in the chain's own fields and need normalising into
  one boolean / price schema.
- The portal is not a formally documented API, so a layout change on their side
  is a realistic failure mode we monitor.
- The portal refuses requests from outside Israel, so collection runs from an
  Israeli address.
- In testing, one portal answered a browser but not a plain command-line
  request, so `collect.sh` sends a normal User-Agent header and fails loudly
  rather than silently writing an empty file.

### Expected record shape

```
collected_at_utc,chain,branch_id,barcode,product_name,unit,price,on_promotion
2026-11-17T13:05:00Z,shufersal,001,7290000066318,Milk 3% 1L,1 L,6.90,false
```

### Schema (dataset.csv columns)

| column           | meaning                                                      |
|------------------|-------------------------------------------------------------|
| collected_at_utc | UTC timestamp of the collection run (ISO 8601)              |
| chain            | chain identifier (constant: `shufersal` for now)           |
| branch_id        | Shufersal branch id, as published in the source file        |
| barcode          | product barcode (unique product key across branches)        |
| product_name     | product name as published                                   |
| unit             | unit of sale (e.g. `1 L`, `500 g`)                          |
| price            | shelf price for that branch at collection time              |
| on_promotion     | `true` / `false` — whether a promotion applied              |

---

## Planned second source (week 7)

The project timeline adds a second, enriching data source in week 7, and the
contract calls for a source that can be integrated with another to produce
richer findings. We will choose one that **explains** the branch differences in
our research question rather than just adding rows — the leading candidate is a
per-locality socioeconomic indicator (CBS), to test whether a branch's tendency
to lead or follow price changes tracks the socioeconomic profile of its town.
This section will be filled in with the chosen source, its content, limitations
and schema once it is locked.

---

## Appendix — How this was built (AI-assisted)

Per the course rules on using language models, this documents the prompts and
decisions behind this file. We used Claude as a thinking and drafting aid; every
choice was reviewed and decided by us.

1. **"The professor replied on our IT project — walk me through the assignment
   in depth, the requirements in plain terms, and what we can do from here."** →
   Re-read the contract, confirmed the live-source model, and locked Proposal 1
   (supermarket prices).

2. **"Help me design the research question, step by step, so I understand it."**
   → Explored several directions, rejected ones that didn't actually depend on
   branches, and settled on: rate of price increase vs decrease across branches
   of the same chain, and which branches lead vs follow.

3. **"Pick the chain with the most convenient portal/API to work with."** →
   Checked the portals and chose Shufersal (open, no login, many branches).

4. **"Check everything we did against the project contract and tell me what
   doesn't fit."** → Found that the required second data source (week 7) was
   missing; added a note deferring it and kept the rest.

5. **"Give me a different basket — products whose prices move, and diverse — and
   pull a real sample from one central Shufersal branch to prove it works."** →
   Built a 25-product national-brand basket, excluded price-controlled items,
   and verified every product against a real `PriceFull` file from branch 001.

6. **"Three collections a day; six branches (3 Tel Aviv + 3 periphery) so it
   isn't biased; store only the filtered basket to keep the size small."** →
   Updated the collection frequency, the branch design, and the collect-and-
   filter storage approach.
