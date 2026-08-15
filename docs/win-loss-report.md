# The Report That Names Your Competitors

**A case for adding Win/Loss analysis to the Filtec Sales Dashboard**

Filtec Reporting · 15 August 2026
Data as of 15 August 2026 · ERPNext (Filtec), both companies

---

## Summary

The dashboard has eight data tabs. Every one of them measures **money Filtec
won**. None of them measures **money Filtec lost**.

That is the gap, and it is the whole of the competitive question. Competitive
position is not defined by the revenue you booked — it is defined by the
revenue that went to somebody else. Every quotation that does not convert is
an order a competitor took.

The single highest-value report Filtec does not have is a **Win/Loss (quote →
order conversion) report**. This document sets out what the current data
supports, what it does not, and what has to change in ERPNext before the
report can name a competitor.

---

## 1. What the dashboard measures today

| Tab | What it measures | Direction |
|---|---|---|
| Brand | Revenue by item brand | Won revenue |
| Key Accounts | Revenue concentration by customer | Won revenue |
| GP % | Margin on invoiced lines | Won revenue |
| Retention | Repeat purchasing by existing customers | Won revenue |
| Segmentation | Revenue by customer segment | Won revenue |
| Expenses | Cost base | Internal |
| Net Profit | Revenue less cost | Internal |
| Staff | Attribution of orders to salespeople | Won revenue |

Eight tabs, one direction. This is a well-built rear-view mirror. It answers
"how did we do?" with real precision and cannot answer "why did we not win
that one?" at all.

---

## 2. What the quotation data shows

Every quotation ever raised in ERPNext, across PT Filtec Indotama Teknindo
and PT Filtec Indo Teknologi:

| Quotation status | Count | Share |
|---|---:|---:|
| Draft — never issued or never advanced | 274 | 60.8% |
| Ordered | 85 | 18.8% |
| Expired | 46 | 10.2% |
| Cancelled | 31 | 6.9% |
| Partially Ordered | 12 | 2.7% |
| **Lost** | **3** | **0.7%** |
| **Total** | **451** | **100%** |

Two conversion readings are possible, and they disagree:

- **85 of 451 (18.8%)** of all quotation records became orders.
- Of the 146 quotations that reached a *decided* outcome — Ordered, Partially
  Ordered, Expired or Lost — **97 (66%) were won at least in part**.

Neither figure is a true win rate, and that is the finding. With 274
quotations sitting in Draft, the denominator is not trustworthy in either
direction. Filtec does not currently know its own win rate.

---

## 3. Three findings

### 3.1 Losses are not being recorded

Three quotations in the entire history carry the status `Lost`. All three are
from the legacy `SAL-QTN-2025` series. Not one quotation raised under the
current `QTN-FITT` or `QTN-PFIT` numbering has ever been marked lost.

This is not a 99% win rate. It is an absent habit. The 46 `Expired`
quotations are the losses nobody wrote down — deals that aged out of validity
with no disposition, no reason, and no competitor named. On present practice,
a lost bid leaves no trace in the system at all.

The consequence is concrete: **Filtec cannot answer who beat it, on what, or
by how much.** Not because the answer is hard to compute, but because nobody
is writing the answer down.

### 3.2 Nearly one order in five is lost *after* it is won

| Sales Order status | Count |
|---|---:|
| Total Sales Orders | 230 |
| Cancelled | 43 (18.7%) |

Forty-three cancelled Sales Orders is a larger leak than most companies'
entire win-rate problem, and it is invisible on the dashboard today. These are
deals already won — quoted, negotiated, accepted — and then lost anyway. Some
will be administrative re-issues rather than genuine losses; the point is that
nobody currently knows the split, because the reason is not captured.

Won-then-lost revenue is the cheapest revenue in the business to recover. It
needs no new customer, no new quote and no new price.

### 3.3 The bid channel is opaque

On 11 June 2026, eight quotations were raised on the same day, each for
exactly **Rp 285,936,000**, to eight different customers:

```
QTN-FITT-2026-00096   PT Kilang Pertamina Internasional RU II Dumai
QTN-FITT-2026-00097   PT. Dua Lion Internasional
QTN-FITT-2026-00098   PT PETRO INDONESIA
QTN-FITT-2026-00099   PT LIENETIC JAYA
QTN-FITT-2026-00100   CV ANGELINE ENGINEERING
QTN-FITT-2026-00101   PT. LAMBANG MAS SEJATI
QTN-FITT-2026-00102   CV. ENERGI KARUNIA ABADI
QTN-FITT-2026-00103   PT. CENTRA GLOBAL INDO
```

Identical value, identical date, eight counterparties. The natural reading is
one end-user requirement bid through eight trading channels at once. **This is
an inference, not a recorded fact** — no field in ERPNext links these eight
quotations to a common tender, which is precisely the problem.

All eight remain in `Draft`. There is no report that says which channel
converts, so Filtec is quoting the same requirement eight ways without
learning which route wins. That is margin spent to buy information the system
then discards.

---

## 4. What the report should contain

The report is best built in two stages, because half of it is available today
and half of it requires a change to how quotations are dispositioned.

### Stage 1 — buildable now, from existing data

Nothing here needs a new ERPNext field. All of it is computable from
Quotation, Sales Order and Sales Invoice records as they stand:

1. **Conversion rate over time** — quotations raised vs converted, by month,
   showing the trend rather than a single lifetime average.
2. **Conversion by brand** — which product lines Filtec wins with and which it
   merely quotes. The Brand tab shows the revenue; this shows the hit rate
   behind it.
3. **Conversion by deal size** — bucketed by quotation value. A hit rate that
   collapses above a threshold is a pricing signal; one that collapses below a
   threshold is a service-cost signal.
4. **Conversion by salesperson** — with the same confidence-band caveat the
   Staff tab already applies, since order-level attribution is derived rather
   than recorded.
5. **Silent losses** — total value sitting in `Expired`, aged, and ranked. This
   is the backlog of deals that were never formally lost, only abandoned.
6. **The draft backlog** — 274 quotations, by age and value. Some are live
   work; some are dead. Nobody currently knows which.
7. **The post-win leak** — cancelled Sales Orders by value and, where
   recoverable, by cause.
8. **Quote turnaround time** — days from quotation to order for won deals.
   Slow quoting loses deals that price alone would have won.

Charts and tables in this report should start at **Chart 12** and **Table
1324**, continuing the dashboard's global numbering.

### Stage 2 — requires a data-capture change

The competitive half of the report cannot be built from what ERPNext holds
today. It needs three things, in order:

1. **A lost reason field on Quotation.** ERPNext ships with
   `order_lost_reason`; it is not being used. A short controlled list works
   better than free text — *price*, *lead time*, *technical specification*,
   *payment terms*, *incumbent supplier*, *no budget / project cancelled*.
2. **A competitor name field.** Free text is acceptable at first; the list of
   real names that matter will be short, and can be tightened into a Link
   field once it is known.
3. **The discipline of dispositioning quotations.** Every quotation gets
   marked `Ordered` or `Lost`. Letting one expire stops being an acceptable
   outcome. This is a process change, not a software change, and it is the
   part that actually determines whether Stage 2 ever produces anything.

Given that discipline, within roughly one quarter the same tab begins
answering the question that was originally asked — *what makes us ahead of
competitors?* — in the only form that is actionable:

> We lose to Competitor X on lead time, in the power-generation segment, on
> orders above Rp 300 million.

That sentence is worth more than every chart currently on the dashboard,
because it is the only one that can be acted on directly.

---

## 5. Two runners-up

**Delivery reliability (OTIF).** Across the 200 most recent Sales Orders,
promised lead time runs from same-day to 156 days, with a median of 4 days and
a 90th percentile of 32. Promise-versus-actual is not tracked anywhere. In
industrial filtration, delivery reliability is frequently the thing a supplier
is actually beaten on, and it is measurable from existing Delivery Note dates
without any new fields.

**Price realization.** Quoted price versus invoiced price, per item. This
exposes where discount is being given away to hold business, and complements
the GP % tab by showing *why* margin moves rather than only that it did.

---

## 6. How to read this document

- Every count in sections 2 and 3 is a direct record count from ERPNext as of
  15 August 2026, not an estimate.
- The eight-way tender in section 3.3 is an **inference** from identical value
  and date. It is not recorded as a tender anywhere in the system.
- Lead-time figures in section 5 are drawn from the 200 most recent Sales
  Orders, not all 230.
- Section 4's Stage 2 describes work that has **not** been done. No ERPNext
  fields have been created or modified. Changing the Quotation doctype affects
  the live system and is a decision for the business, not a reporting change.

---

*Filtec Reporting · companion to the Filtec Sales Dashboard*
*Source: ERPNext (Filtec) — Quotation, Sales Order and Sales Invoice records,
both companies*
