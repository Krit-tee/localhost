# Car rental: Rangsit → Bangsaen, 13–15 Oct 2026

Track: Read & research. Reply to the user in Thai.

## Goal
Recommend self-drive small/mid sedan rentals (2 travellers) picked up in or near
Rangsit, for 13–15 Oct 2026 (Tue–Thu), with: which provider, how to contact,
price. Every price/contact must be verified on the provider's own page (or the
user told plainly what stayed unverified).

## Context
- 13 Oct 2026 is a Tuesday and a public holiday (Navamintharamaharat Day), so expect high demand.
  Weekday checked with Python. Holiday listed in https://media.ttbbank.com/1/document/branch/Special-Holidays-for-Year-2026-TH.pdf (seen only in search results).
- Rental days: pickup 13 Oct morning → return 15 Oct evening is usually billed as 3 days (24h blocks).
  Returning before the 13 Oct pickup time is billed as 2 days.
- Previous session: the network policy blocked every provider site
  (`ckcarrental.com`, `www.ckcarrent.com`, `www.jtcarrent.com`, `www.haupcar.com`,
  `www.ecocar.co.th`, `www.chiccarrent.com`, `th.trip.com`, `www.drivehub.com`,
  `www.thairentacar.com`). The user updated permissions; that should apply in this new session.

## Findings so far (from search-result summaries only — NOT verified)
| Provider | Pickup | Car / price per day | Deposit | Contact | Source |
|---|---|---|---|---|---|
| CK Car Rent | Rangsit branch near Future Park; delivery | Yaris Ativ 1,070 THB (promo page says "from 963") | 5,000 THB | 02-126-0718, 083-918-3911, LINE @ckcarrent | https://ckcarrental.com/rangsit-car-rental/ , https://www.ckcarrent.com/car/ativ |
| HAUP | Future Park Rangsit outdoor lot (Robinson exit, near AIS shop) | from 116 THB/hour, insurance incl.; daily rate unknown | none for some models | HAUP app | https://www.haupcar.com/forum/station-location/echaarth-pthumthaanii-fiwecch-rphaarkhrangsit-future-park-rangsit-1 |
| JT Car Rent | free delivery to Rangsit / Don Mueang | Almera 1,190, Mazda2 1,190, Yaris Ativ 1,290 THB | cash, no credit card | 061-889-3662, LINE @carrent_jt | https://www.jtcarrent.com/product/20/car-rental-rangsit |
| Chic Car Rent | Don Mueang airport | Yaris 1.2 850 THB (KTC promo, may be expired); 1st-class insurance (SCDW) | ? | 02-286-6799 (Mon–Fri 07:00–19:00) | https://www.ktc.co.th/promotion/travel/car-rental-rail-pass/chic-car-rent-super-deals |
| Ecocar (Thai Rent Eco Car) | 279/57 Vibhavadi Rangsit Rd, Don Mueang, 08:00–20:00 | from 856 THB | ? | 02-002-4606, 092-284-8660, LINE @ecocar | https://www.ecocar.co.th/carrentaldonmueang |
| Lower priority, unverified | WCarrent (Future Park, "600 THB/day", 061-015-2176); JP CarRent (063-234-8904) | | | | https://wcarrent.com/County/Donmueang/Road/Future-Park-Rangsit-Carrent.html , https://www.facebook.com/jpcarrents/ |

Current recommendation (to confirm): 1) CK Car Rent (Rangsit pickup, clear price/deposit);
2) HAUP (self-service via app); 3) Chic / Ecocar if the user doesn't mind collecting the car at Don Mueang and wants a lower price.

## Steps
1. Open each provider page in the table with WebFetch → verify: price for an eco/compact sedan, deposit, documents (credit card needed?), insurance + deductible, out-of-province allowed, contact. Record the URL per fact.
2. Check for the 13–15 Oct holiday period: any holiday surcharge or minimum days, plus availability notes → verify: quoted from the page, or marked "ask provider".
3. Route cost: Rangsit → Bangsaen distance and Motorway 7 / expressway tolls (official or map source) → verify: URL. Previous estimate (unverified): 110–130 km each way, fuel ~800–1,000 THB round trip.
4. Write the final answer in Thai: comparison table with 2-day and 3-day totals, top pick and why, questions to ask when booking, a "verified vs inferred" note, and a Sources list.

## Done when
- Every price/deposit/contact in the answer is opened at its source (or flagged unverified).
- Answer covers: recommended providers, contact, price (2 & 3 days), booking checklist.

## Progress log
- 2026-10-06: Search-based findings collected (table above). All provider sites blocked by egress policy → nothing verified first-hand. Next: step 1.
