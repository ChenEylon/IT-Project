# Data Source Proposals

Chen Eylon · Ori Lisser

Two candidate live sources. Both publish only a current state: ask them tomorrow
about today and the answer is gone. Neither is an existing dataset — the history
only exists if we collect it, which is the point of the project.

**Preference: Proposal 1.**

---

## Proposal 1 — Supermarket prices across competing chains *(preferred)*

### What it is

Israel's price transparency law requires every food chain to publish its prices
for every branch, and to update them within an hour of a price change. Each
chain runs its own portal of timestamped files. We collect a fixed basket of
products from two competing chains on the same schedule and track the gap
between them.

One chain alone says little. Two chains, measured at the same moment over a
semester, say who is actually cheaper, on what, and when that changes.

### Expected content

Per product, per branch, per collection: barcode, product name, unit, price,
and whether a promotion applies. Each file carries the branch id and the
timestamp of publication.

### Research question

Which chain is cheaper on a fixed basket, how does the gap move over the
semester, and do prices rise before holidays?

### URL

The source of record is the Ministry of Economy's index, which lists the
statutory price portal of every chain:

- https://www.gov.il/he/departments/legalInfo/cpfta_prices_regulations

From that index we work with two chains at a time. Our intended pair:

- Shufersal: https://prices.shufersal.co.il
- Rami Levy: https://www.rami-levy.co.il/he/price-transparency

The pair is deliberately interchangeable. The comparison needs two competing
chains, not these two specifically, so if a portal is retired or becomes
unreliable we substitute another chain from the same index without changing
the research question or the schema.

### Update frequency

The law requires an update within an hour of any price change. We plan to
collect once per day per branch, which is enough to see weekly and seasonal
movement without storing near-duplicate files.

### Expected size

Individual price files are gzipped and run roughly **3–28 KB each**, measured
from the file listing on the Shufersal portal. Two chains, a few branches, once
a day for a semester stays well under 50 MB — far inside the 500 MB budget.

### Key or password

Shufersal publishes without a login. Some chains require a username, which the
project contract explicitly allows. Because the Ministry of Economy maintains a
public index of every chain's portal, swapping one chain for another would not
break the proposal.

### Limitations

- A product missing from a file may be out of stock or may simply not be
  carried by that chain. We will treat the two as different and not fill gaps.
- Branch coverage differs between chains, so comparisons are per basket, not
  per branch location.
- Promotions are expressed differently by each chain and will need normalising
  into one schema.
- The portals are not formally documented as APIs, so a layout change on their
  side is a realistic failure mode we will have to monitor. Some chains run
  more than one portal and retire the older one, so we will check our two
  against the Ministry's index periodically rather than hard-coding them once.
- Some of these portals refuse requests from outside Israel, so collection has
  to run from an Israeli address.
- Portal availability is uneven. In our own testing one portal answered a
  browser but not a plain command-line request, so `collect.sh` will need to
  send a normal User-Agent header and to fail loudly rather than silently
  writing an empty file.

### Expected record shape

```
collected_at_utc,chain,branch_id,barcode,product_name,unit,price,on_promotion
2026-11-05T13:05:00Z,shufersal,001,7290000066318,Milk 3% 1L,1 L,6.90,false
```

---

## Proposal 2 — Tel Aviv parking lot occupancy

### What it is

Tel Aviv-Yafo publishes the live status of the Ahuzot HaHof parking lots through
its public GIS service. Each lot reports one of four states — available, few
spaces, full, or closed. The service exposes the current state only; yesterday's
pattern is not retrievable from it.

### Expected content

Per lot, per poll: lot name, address, coordinates, total spaces, subscriber
spaces, tariffs, and the current status with its status timestamp. The layer
carries 29 fields in total.

### Research question

When and where does the city run out of parking, and how does that pattern
differ between weekday mornings, Friday afternoons and rainy days?

### URL

https://gisn.tel-aviv.gov.il/arcgis/rest/services/IView2/MapServer/970/query

### Update frequency

The status field is refreshed continuously; an independent project that consumes
this layer caches it for 120 seconds, which suggests that is roughly the useful
resolution. We plan to collect every 10 minutes.

### Expected size

A few dozen lots returned as JSON is a few kilobytes per poll. At one poll per
10 minutes this is a few megabytes per month — comfortably inside the budget.

### Key or password

None. The layer is served from a public ArcGIS REST endpoint and supports
`f=json`. We confirmed this with an anonymous request before submitting: the
query returned live records such as
`{"shem_chenyon":"גן זוהר","status_chenyon":"פנוי"}` with no token and no login.

### Limitations

- Status is categorical, not a count. We cannot say "214 spaces free", only
  which of four states a lot is in. We would therefore measure how long each lot
  spends in each state rather than a precise occupancy curve.
- Some lots default to "no information" and may never report usefully. Those
  stay as gaps.
- The layer covers Ahuzot HaHof lots only, not private operators, so it is a
  sample of the city rather than the whole of it.
- Geometry is returned in Israeli Transverse Mercator (EPSG:2039) rather than
  latitude/longitude. The layer also carries `lat` and `lon` fields, so we will
  request those instead of reprojecting.

### Expected record shape

```
collected_at_utc,lot_code,lot_name,address,total_spaces,status,status_updated
2026-11-05T13:05:00Z,123,Gan HaIr,71 Ibn Gabirol St,520,full,2026-11-05T13:03:00Z
```
