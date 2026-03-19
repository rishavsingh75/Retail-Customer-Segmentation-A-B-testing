# Retail Customer Segmentation & A/B Test Evaluation using Python

Analysed supermarket chip sales data to identify high-value customer segments and evaluate the impact of a new store layout trial using A/B testing methodology.

---

## Business Context

A chips category manager at a supermarket chain needed answers to two problems:
1. **Who is buying chips** and what drives their purchasing decisions?
2. **Did the new store layout trial actually work** — did it increase sales and bring in more customers?

The dataset covers ~265,000 chip transactions across 272 stores from July 2018 to June 2019, linked with customer loyalty card data (lifestage + spending tier).

---

## Business Questions

**Part 1 — Customer Analytics**
- Which customer segments contribute most to chip sales?
- Is it because there are more customers in those segments, or because they spend more per visit?
- Do some segments pay more per pack? Is that difference statistically significant?
- Are there specific brands or pack sizes that certain segments prefer?

**Part 2 — A/B Testing & Uplift Evaluation**
- How do you find a fair control store to compare against a trial store?
- Did the new shelf layout (tested in stores 77, 86, 88) significantly increase sales?
- Did it bring in more customers?
- Which trial stores showed a real effect vs random noise?

---

## Part 1 — Customer Segmentation Analysis

**What I did:**
- Cleaned transaction data: fixed date format (Excel serial integer), removed non-chip products (salsa), removed a bulk-buyer outlier (card 226000 — bought 200 packs twice, not a regular retail customer)
- Engineered two new features: `PACK_SIZE` (extracted from product name using regex) and `BRAND` (first word of product name, with alias cleanup for inconsistent naming)
- Merged transaction data with customer loyalty data
- Analysed total sales, customer counts, avg units per customer, and avg price per unit across 21 segments (7 lifestages × 3 spending tiers)
- Ran a Welch's t-test to check if Mainstream young/midage singles genuinely pay more per pack
- Deep dive into Mainstream Young Singles/Couples — calculated brand affinity and pack size affinity index vs the rest of the population

**Key findings:**
- Top segments by revenue: Budget–Older Families, Mainstream–Young Singles/Couples, Mainstream–Retirees
- Mainstream Young Singles/Couples and Retirees: high sales mainly because there are a lot of them, not because each person spends more
- Budget Older Families: buy more units per customer (volume-driven, not price-driven)
- Mainstream young/midage singles pay significantly more per pack (p < 1.12e-309, Welch's t-test) — likely impulse buying behaviour
- This segment over-indexes for Tyrrells (+23%), Twisties (+23%), Kettle (+20%)
- They prefer larger packs (270g, 380g) — the 270g preference maps entirely to Twisties

**Recommendation:** Off-locate Tyrrells and larger packs near areas Mainstream Young Singles/Couples pass through (checkout lanes, beverage aisle) to capture impulse purchases.

---

## Part 2 — A/B Testing & Store Trial Evaluation

**How the A/B test was set up:**
- **Treatment group (A):** Stores 77, 86, 88 — received the new chip shelf layout in Feb 2019
- **Control group (B):** Matched stores that kept the old layout
- **Trial period:** Feb – Apr 2019
- **Metrics measured:** Monthly total sales, monthly unique customers

Since stores were pre-selected (not randomly assigned), I built a matching algorithm to find the most similar control store for each trial store — making the comparison fair. This is known as a quasi-experimental A/B test.

**Control store selection method:**
- Built monthly metrics per store for the pre-trial period (Jul 2018 – Jan 2019)
- Filtered to 259 stores with 12 complete months of data
- Scored each candidate control store using:
  - Pearson correlation — how similar is the trend over time?
  - Normalised magnitude distance — how similar is the absolute level?
  - Combined into one score (equal weight across both metrics)
- Selected the highest-scoring store as the control

**Control stores matched:**
| Trial Store (A) | Control Store (B) |
|-----------------|-------------------|
| 77 | 233 |
| 86 | 155 |
| 88 | 237 |

**Testing method:**
- Scaled control store metrics to trial store pre-trial level
- Calculated t-statistics for each trial month against the pre-trial standard deviation
- Compared against 95% confidence interval (t-critical with 7 degrees of freedom)

**Results:**
| Trial Store | Sales Uplift | Customer Uplift |
|-------------|-------------|----------------|
| 77 | Significant (Mar + Apr) | Significant (Mar + Apr) |
| 86 | Not significant | Significant (all 3 months) |
| 88 | Significant (Mar + Apr) | Significant (Mar + Apr) |

**Recommendation:**
- Stores 77 and 88 showed clear positive results — recommend rolling out the new shelf layout chain-wide
- Store 86 brought in more customers but sales did not go up significantly — likely promotional pricing during the trial reduced average spend per visit. Worth investigating before applying the rollout to similar stores

---

## Tech Stack

- Python 3
- pandas, numpy
- matplotlib, seaborn
- scipy (Welch's t-test)

---

