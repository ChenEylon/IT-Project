# Data Source

Chen Eylon · Ori Lisser

A single live source, chosen after professor feedback on our two original
proposals. The source publishes only a current state: ask it tomorrow about
today and the answer is gone. The history only exists if we collect it, which
is the point of the project. A second, enriching source will be added in week 7
(see "Planned second source" at the end).

---

## 1. Source at a glance

| field | value |
|-------|-------|
| **Name** | Shufersal price-transparency portal — `PriceFull` files |
| **URL** | https://prices.shufersal.co.il (listed in the Ministry of Economy index: https://www.gov.il/he/departments/legalInfo/cpfta_prices_regulations) |
| **Data format** | XML, one file per branch, published gzipped (`.gz`); we store the extracted `.xml` |
| **Key / username required** | No — public, no login |
| **License** | Published under the Promotion of Competition in the Food Industry Law (2014), which requires chains to make prices public. The portal states no explicit open-data license; we use the data for non-commercial academic research only. |
| **Sampling frequency** | Manual phase (now): one sample per day. Automated phase: three times a day per branch (morning, midday, evening). |

---

## 2. What we collect

Israel's price transparency law requires every food chain to publish the prices
of every branch and to update them within an hour of a price change. Shufersal
publishes a full price file (`PriceFull`) per branch, with every product the
branch sells.

From these files we track a **fixed basket of 25 national-brand products** in
**six Shufersal branches** (three in Tel Aviv, three in northern peripheral
cities) and record, per product and branch, its price and when Shufersal last
updated that price. This lets us answer our research question: how the pace of
price increases differs from the pace of decreases between branches of the same
chain, and which branches lead price changes and which follow.

**Current state (Exercise 3):** four manual samples of branch 001 (Shufersal
Sheli Tel Aviv – Ben Yehuda), one per day, 18–21 Nov 2026, stored as full
`PriceFull` files:

```
raw/2026-11-18/0300.xml
raw/2026-11-19/0300.xml
raw/2026-11-20/0431.xml
raw/2026-11-21/0300.xml
```

We started with one branch to confirm that downloading and storing works end to
end. Before the automated phase we will confirm with the course staff how to
name six branch files collected at the same minute, since the contract's
`raw/YYYY-mm-dd/HHMM.ext` rule allows one file per minute.

---

## 3. Schema (`dataset.csv`)

One row per product, per branch, per collection.

| column | content | unit / format |
|--------|---------|---------------|
| `collected_at_utc` | when the sample was taken | ISO 8601, UTC (e.g. `2026-11-18T01:00:00Z`) |
| `chain` | chain identifier | text, constant `shufersal` |
| `branch_id` | Shufersal branch number | 3-digit text (e.g. `001`), kept as text to preserve leading zeros |
| `barcode` | product barcode — the key that matches a product across branches | text (digits) |
| `product_name` | product name as published | text, Hebrew |
| `unit` | package size and unit | text, quantity + unit (e.g. `439 גרם`) |
| `price` | shelf price at the branch | ILS (₪), 2 decimals |
| `price_updated_at_utc` | when Shufersal last changed this product's price | ISO 8601, UTC |

Example row:

```
collected_at_utc,chain,branch_id,barcode,product_name,unit,price,price_updated_at_utc
2026-11-18T01:00:00Z,shufersal,001,7290000066318,במבה 80 גרם,80 גרם,5.90,2026-10-02T06:30:00Z
```

---

## 4. Why these columns (and not the others)

**Kept:**

- `collected_at_utc`, `branch_id`, `barcode` — together they identify one
  observation: which product, in which branch, at which moment. Without all
  three there is no time series per branch.
- `chain` — constant today, but keeps the dataset correct if we ever compare
  chains or merge in another source.
- `price` — the variable the research question is about.
- `price_updated_at_utc` — **added after inspecting the files.** It tells us
  when Shufersal itself changed the price, which is more precise than our three
  daily snapshots, and directly serves the "who leads, who follows" part of the
  question.
- `product_name`, `unit` — not needed for the analysis (the barcode is the key),
  but they make the data readable and let us check that the same barcode really
  is the same product and size in every branch.

**Dropped:**

- `on_promotion` — **removed from the original plan.** `PriceFull` files contain
  no promotion field. `AllowDiscount` only says whether a discount is *allowed*,
  not whether one applies. Promotions are published in separate `PromoFull`
  files; we may add them later.
- `ManufactureName`, `ManufactureCountry`, `ManufactureItemDescription` — fixed
  per barcode and irrelevant to price changes.
- `UnitOfMeasurePrice`, `UnitOfMeasure` — price per 100 g / per unit; derivable
  from `price` and `unit` if ever needed.
- `LastSaleDateTime` — when the item last sold; about sales, not prices.
- `ItemType`, `bIsWeighted`, `QtyInPackage`, `ItemStatus`, `SubChainID`,
  `BikoretNo` — technical fields, constant or empty in our basket.

---

## 5. Field translation (source → schema)

| source (XML) | level in file | schema column | transformation |
|--------------|---------------|---------------|----------------|
| *(path `raw/YYYY-mm-dd/HHMM.xml`)* | — | `collected_at_utc` | date from folder + time from file name, Israel time → UTC |
| `ChainID` (`7290027600007`) | file | `chain` | mapped to the constant `shufersal` |
| `StoreID` | file | `branch_id` | as is (3-digit text) |
| `ItemCode` | item | `barcode` | as is |
| `ItemName` | item | `product_name` | as is |
| `Quantity` + `UnitQty` | item | `unit` | joined: `439.00` + `גרם` → `439 גרם` |
| `ItemPrice` | item | `price` | as is |
| `PriceUpdateTime` | item | `price_updated_at_utc` | Israel time (no timezone in source) → UTC |

---

## 6. Expected size after one semester

Assumptions: about 60 days of collection remain this semester; 6 branches ×
3 collections a day = 18 files a day.

**Measured:** the four manual samples take 22 MB together (`du -sh raw`), about
**5.5 MB per full `PriceFull` file**, because each file lists every product in
the branch (thousands), while we need 25.

| | calculation | semester total |
|---|---|---|
| `raw/` — full files (not viable) | 5.5 MB × 18 × 60 | **≈ 6 GB** |
| `raw/` — filtered to our 25 products | ≈ 17 KB × 18 × 60 | **≈ 18 MB** |
| `dataset.csv` | 25 × 6 × 3 × 60 = 27,000 rows × ~100 bytes | **≈ 2.7 MB** |

Storing full files would make the repository far too large for GitHub (which
recommends staying under 1 GB) and break our own 500 MB budget. So the
automated collector (`collect.sh`) will download each full file but keep **only
our 25 basket products** in `raw/`. The four manual samples stay as full files,
for reference.

Trade-off, stated deliberately: filtering at collection time means a product we
did not collect cannot be recovered later. Since the basket is fixed for the
semester, this is an acceptable choice.

---

## Appendix A — Basket

25 **national-brand** products across categories (soft drinks, snacks, sweets,
dairy, dry goods, household, personal care). Three principles:

1. **National brands only**, so the exact same barcode exists in every branch —
   this is what makes a cross-branch comparison valid.
2. **Products whose price moves.** Price-controlled items (milk in cartons,
   white bread, eggs, salt) are excluded because their price is frozen by law.
3. **Diversity across categories**, so findings are not an artefact of one
   product family.

Verified against a real `PriceFull` file from branch 001 on 2026-11-17. Exact
single-unit barcodes are finalised when `collect.sh` first confirms each product
appears in every chosen branch.

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

## Appendix B — Branches

Six branches — **three in Tel Aviv and three in northern peripheral cities**
(e.g. Afula, Tzfat) — so the comparison is group-vs-group, not
branch-vs-branch:

- **Tel Aviv** — 001 (Sheli, Ben Yehuda), 007, 011.
- **Periphery** — three branches in northern cities, ids confirmed against the
  portal's branch list.

With one branch per side, any difference could be a single branch's quirk. With
three, we can separate within-group variation from a real between-group
difference.

## Appendix C — Limitations

- A product missing from a file may be out of stock or not carried by that
  branch. We do not fill gaps.
- Publication time varies: three of our samples were published at 03:00, one at
  04:31. The file name records the publication time, which may differ from the
  moment we download it.
- Times in the source carry no timezone; we treat them as Israel time and
  convert to UTC.
- The portal is not a formally documented API, so a layout change is a
  realistic failure mode.
- The portal refuses requests from outside Israel, so collection runs from an
  Israeli address.
- In testing, one portal answered a browser but not a plain command-line
  request, so `collect.sh` sends a normal User-Agent header and fails loudly
  rather than silently writing an empty file.
- The portal documents no limit on request frequency; three requests a day per
  branch is far below anything that could burden it.

## Planned second source (week 7)

The project timeline adds a second, enriching data source in week 7, and the
contract calls for a source that can be integrated with another to produce
richer findings. We will choose one that **explains** the branch differences
rather than just adding rows — the leading candidate is a per-locality
socioeconomic indicator (CBS), to test whether a branch's tendency to lead or
follow price changes tracks the socioeconomic profile of its town. This section
will be filled in with the chosen source, its content, limitations and schema
once it is locked.

---

## Appendix D — How this was built (AI-assisted)

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

7. **"Guide us through organizing the manual samples and preparing the data
   documentation for Exercise 3."** → Organized four daily samples of one
   branch under `raw/` according to the contract's naming rule; reviewed the
   fields in the XML files and decided which to keep; dropped `on_promotion`
   and added the price-update time; based the size estimate on measured file
   sizes; and arranged this file into the six required sections.
