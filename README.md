# Seguro: AI-Powered Parametric Income Insurance for Gig Delivery Workers

Guidewire DEVTrails 2026 | Protecting delivery partners from income loss caused by uncontrollable external disruptions.

**Video Link**  : https://youtu.be/wdHjv9sngxA
**Website Link**: https://elseguro.netlify.app/ 
---

## Table of Contents

1. [Inspiration](#inspiration)
2. [What It Does](#what-it-does)
   - [Weekly Cycle](#weekly-cycle)
   - [Fraud Review and Payout Pipeline](#fraud-review-and-payout-pipeline)
   - [Persona-Based Scenarios](#persona-based-scenarios)
3. [AI and ML Integration](#ai-and-ml-integration)
4. [How We Plan to Build It](#how-we-plan-to-build-it)
5. [Adversarial Defense & Anti-Spoofing Strategy](#adversarial-defense--anti-spoofing-strategy)
6. [Challenges We Ran Into](#challenges-we-ran-into)
7. [Accomplishments That We're Proud Of](#accomplishments-that-were-proud-of)
8. [What We Learned](#what-we-learned)
9. [What's Next for Seguro](#whats-next-for-seguro)
---

## Inspiration

India's food delivery workers on platforms like Zomato, Swiggy, Zepto, and Amazon earn income entirely contingent on being able to go outside and work. A monsoon evening, a local curfew, or dangerous air quality wipes out their earnings for that period with no recourse. There is no paid leave, no employer coverage, and no insurance product built for their reality.

The data to fix this already exists. Every platform logs every order, every zone, every rupee earned. Seguro connects that data with external disruption signals to determine, when a worker applies for compensation, whether their loss was real and caused by something outside their control.

---

## What It Does

Seguro is a web-based parametric income insurance platform operating on a weekly cycle, aligned to how gig platforms pay their workers.

### Weekly Cycle

**Week 1: Onboarding and Baseline Collection**

A worker registers and links their delivery platform accounts. No premium is collected and no compensation is available. The system collects the following data daily to establish a personal income baseline:

| Data Point | Description |
|---|---|
| Orders per zone | Deliveries completed in each area the worker operates in |
| Total orders across platforms | Combined count across all linked platforms |
| Delivery amount per order | Earning value of each delivery |
| Delivery history | Area, earnings, and time of every delivery, logged daily |

**Week 2: Account Approved, Coverage Begins**

The worker's account is approved and a weekly premium is calculated from their Week 1 risk profile. The worker selects a coverage tier.

| Tier | Weekly Premium | Compensation % | Weekly Payout Cap |
|---|---|---|---|
| Basic | Rs. 29 | 40% of estimated loss | Rs. 800 |
| Standard | Rs. 49 | 60% of estimated loss | Rs. 1,400 |
| Premium | Rs. 79 | 80% of estimated loss | Rs. 2,200 |

The base tier price is adjusted by a zone risk multiplier from the ML risk scoring model, kept within a narrow band to preserve affordability.

**Payout Calculation Example**

A worker with a rolling average of Rs. 650 per day, two disrupted days, on the Standard tier:
```
Estimated Loss = Rs. 650 x 2 = Rs. 1,300
Payout         = Rs. 1,300 x 60% = Rs. 780
```
Payouts are capped at the weekly tier maximum. The cap keeps the product financially sustainable and ensures workers always earn more by working than by relying on Seguro alone.

**Week N Onwards: Worker-Initiated Compensation Application**

Compensation does not trigger automatically. The worker logs in and submits an application for the affected week. Two conditions must both be satisfied.

**Condition 1: Parametric Order Volume Drop**
```
Orders(Week N-1) < k * Orders(Week N-2)
```
`k` is a sensitivity threshold between 0 and 1. The worker's order count in the claimed week must fall below `k` times their order count the previous week. This filters out normal quiet weeks and only flags meaningful drops.

`k` is calibrated per zone using the standard deviation of historical week-over-week order change ratios. A low-variance zone gets a `k` closer to 1 since even a small drop is statistically unusual. A high-variance zone gets a lower `k` since a larger drop is needed before it stands out from natural fluctuation. Exact values will be finalised in Phase 2 once zone-level order data is available.

**Condition 2: External Disruption Confirmed in the Worker's Zone**

The system cross-references multiple APIs for the claimed week and zone. No single source is treated as definitive. A genuine disruption leaves a consistent trace across independent data sources; a false or minor event is unlikely to.

| Data Source | What is Checked |
|---|---|
| OpenWeatherMap | Rainfall crossing a defined severity threshold in the worker's zone |
| WAQI | AQI reaching hazardous levels for a sustained period |
| GNews API | Curfew, bandh, or strike reports in the worker's city |
| Google Maps Traffic API | Anomalous congestion drop in the worker's operating zones |

For example, a rain event requires both OpenWeatherMap to confirm rainfall severity and Google Maps to show a corresponding traffic drop in the same zone and time window. A GNews curfew report without a matching traffic anomaly does not satisfy Condition 2 on its own.

**Decision Matrix**

| Condition 1 | Condition 2 | Outcome |
|---|---|---|
| Order drop confirmed | Disruption confirmed | Proceeds to fraud review |
| Order drop confirmed | No disruption detected | Rejected. Slow week, not a covered event. |
| No order drop | Disruption confirmed | Rejected. Worker maintained activity. |
| Neither | Neither | Rejected. |

### Fraud Review and Payout Pipeline

![alt text](<pipeline.png>)

---

## Persona-Based Scenarios

**Scenario 1: Ravi, Swiggy partner in Koramangala, Bengaluru (successful claim)**

Ravi operates in Koramangala and HSR Layout, averaging 38 to 42 orders per week. His Week 1 baseline produces a daily average of Rs. 620. He is on the Standard tier. In Week 4, heavy rain hits South Bengaluru for two days. He completes only 21 orders.

Condition 1: 21 orders is below k times his Week 3 count of 39. Satisfied.
Condition 2: OpenWeatherMap confirms sustained heavy rainfall in Koramangala. Google Maps shows a sharp congestion drop on the same days. Both signals agree. Satisfied.
Zone check: Koramangala is in his Week 3 delivery history.
Cross-check: His logs show morning deliveries on both days but none in the afternoon. Loss is calculated against afternoon hours.

Estimated Loss: Rs. 620 x 1.5 effective days = Rs. 930. Payout at 60% = Rs. 558. Paid to his UPI the same day.

**Scenario 2: Meena, Zomato partner in Anna Nagar, Chennai (rejected claim)**

Meena averages 33 orders per week. In Week 6 she submits a claim citing a bandh in Chennai. Her order count was 28, down from 35 the prior week.

Condition 1: 28 is below k times 35. Satisfied.
Condition 2: GNews returns one article about a partial bandh in North Chennai. Google Maps shows no significant traffic anomaly in Anna Nagar or Kilpauk. The two signals do not agree. Condition 2 fails.

Application rejected. Meena is shown that the disruption could not be confirmed in her zone from multiple data sources.

**Scenario 3: Arjun changes operating zone**

Arjun has worked in Indiranagar for three weeks. In Week 4 he starts taking orders in Whitefield. The system detects Whitefield appearing in his delivery logs for the first time. Coverage for Whitefield is paused for Week 4. He is notified that one full week of delivery history is needed there before claims are eligible. His Indiranagar coverage is unaffected. From Week 5, Whitefield is added to his eligible zones and his premium is recalculated to reflect the new zone mix.

---

## AI and ML Integration

**Risk Scoring for Premium Calculation**

A weekly premium is not flat across workers on the same tier. An XGBoost model takes the following inputs and produces a zone risk multiplier that adjusts the tier base price:

- Historical disruption frequency in the worker's registered zones
- Season and time of year (monsoon months carry higher base risk)
- Zone-level order variance
- Number of distinct zones the worker operates in

The model is trained on historical zone-level disruption data and can be retrained on actual payout outcomes as the platform accumulates its own claims history.

**Fraud Detection Using Isolation Forest**

The cohort anomaly check in Step 5 uses an Isolation Forest model. When workers in the same zone submit claims for the same event, their losses form a distribution. The model scores each claim against that distribution and flags outliers for manual review rather than auto-approval. Because it is unsupervised, it does not require labelled fraud data to train on, which matters in the early stages when confirmed fraud cases are scarce.

**Rolling Baseline**

The income baseline transitions from Week 1 data to a rolling average after sufficient history accumulates. This ensures a worker who has grown their order volume is not permanently anchored to their first week, and a worker who had an unusually strong first week does not start with an inflated baseline. The rolling window length will be tuned during Phase 2.

---

## How We Plan to Build It

### Architecture

![alt text](<architecture_diagram.png>)

### Tech Stack

**Frontend:** React.js with Tailwind CSS, built as a Progressive Web App.

**Why PWA over native mobile:** Delivery workers typically use mid-range Android phones with limited storage. A Play Store app creates download and update friction before a worker has even seen the product. A PWA is accessible via a URL that can be shared over WhatsApp, which is how most delivery worker networks communicate. It also means one codebase for a six-week build instead of maintaining separate Android and iOS versions. The main interaction pattern, checking coverage status or submitting a weekly application, does not require deep native integration, so the PWA trade-offs are acceptable.

**Backend:** Node.js with Express for the main API. Python with FastAPI for the ML microservice. Daily data ingestion and weekly compensation checks run as scheduled cron jobs.

**Database:** PostgreSQL for all persistent data. Redis for caching API responses and trigger state per zone.

**ML and AI:** XGBoost for risk scoring, Isolation Forest for cohort anomaly detection, rolling average for baseline calculation. See the AI and ML Integration section for detail.

**External APIs**

| API | Purpose | Access |
|---|---|---|
| OpenWeatherMap | Weather data and forecasts | Free tier |
| WAQI | Air quality index by location | Free token |
| GNews API | Curfew and strike detection | Free tier |
| Google Maps JS API | Zone mapping and traffic anomaly | Free tier |
| Razorpay Test Mode | Simulated UPI payout | Free sandbox |
| Internal mock server | Simulated platform order data | Self-hosted |

**Hosting:** Frontend on Vercel, backend on Render or Railway, version control on GitHub.

---

## Adversarial Defense & Anti-Spoofing Strategy

### Why GPS Spoofing Doesn't Work

Seguro never uses GPS as evidence. Both payout conditions rely on data that lives outside the worker's device: 
- platform order logs (Condition 1) and 
- independently queried external APIs like OpenWeatherMap, WAQI, and Google Maps Traffic (Condition 2). 

Spoofing a location doesn't alter Swiggy's servers or fabricate rainfall data.

The delivery log is the hardest check to beat: if a worker earned anything during the claimed window, the platform log shows it and the payout shrinks accordingly. Zone eligibility is derived from prior delivery history, so spoofing into a new zone doesn't retroactively add orders there either.

### Detecting Coordinated Rings

- **Cohort anomaly:** Isolation Forest flags claims that are statistical outliers against other workers claiming the same event in the same zone.
- **Claim timing:** A burst of submissions for the same zone and week in a short window is treated as a coordination signal.
- **Cross-platform consistency:** A drop on one app but normal volume on another in the same zone is flagged.

### Handling Flagged Claims Fairly

Flagged claims show a "verification required" message with a 48-hour window: same phrasing for honest workers and bad actors alike. Appeals are accepted, and a single flag doesn't permanently mark an account.

| Attack | Defense |
|---|---|
| GPS spoof to fake zone | GPS not used in evaluation |
| Claim loss without order drop | Condition 1 reads platform logs |
| Fabricate disruption | Condition 2 requires multi-source agreement |
| Coordinated ring | Isolation Forest cohort scoring |
| Claim outside delivery history | Zone eligibility from historical logs |
| Drop on one platform only | Cross-platform consistency check |
---
## Challenges We Ran Into

**Single-week baseline.** Week 1 may not represent a worker's true earning capacity if it was an unusual week. The rolling average mitigates this over time but early claims rely on limited data.

**Causality vs coincidence.** An order drop and a disruption can occur in the same week without being related. We address this by requiring the drop to overlap with the specific hours and zones where the disruption was confirmed, not just the same calendar week.

**Normalising across platforms.** Workers on multiple platforms have order counts with different commission structures and peak patterns. We aggregate into a single platform-agnostic delivery count, which requires avoiding double-counting when the same order appears across platforms.

**Moral hazard.** Partial replacement only (40-80%) means working through a disruption always pays more than not working.

---

## Accomplishments That We're Proud Of

**Zone-calibrated k.** The sensitivity threshold adjusts per zone based on historical order variance, making Condition 1 meaningful across different types of operating areas.

**Delivery log as the primary fraud signal.** The most robust check in the pipeline is Step 3. If the worker earned income during the disruption window, the log shows it, and the claim is reduced accordingly.

**Multi-platform aggregation.** Workers are assessed on their combined activity across all linked platforms, matching how they actually think about their income.

---

## What We Learned

The hard part of parametric insurance is threshold design, not payout mechanics. Every number in the system is a policy decision with real consequences. Too tight and valid claims get rejected. Too loose and the product becomes unviable.

Keeping the application worker-initiated was the right call. Workers who were not meaningfully affected are unlikely to apply, which concentrates payouts on genuine cases without needing to filter them out algorithmically.

The best fraud signal is the worker's own delivery data. If they worked during a disruption, the log says so. The system is largely self-auditing.

---

## What's Next for Seguro

Moving from the mock platform API to real platform data integration, even in a sandbox capacity, would significantly improve baseline and fraud check accuracy.

Regional language support in Hindi, Kannada, Tamil, and Telugu for broader accessibility.

Exploring the IRDAI sandbox framework as the regulatory path toward a formally licensed product.


---
