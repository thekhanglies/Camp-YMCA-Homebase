# YMCA Homebase Details

# Excel Solver & Ops Analytics Portfolio — Story & Concepts

---

## The Journey

**1 — Started from a classroom model — the Joe Carpenter LP framework**
Learned Solver mechanics on a textbook production-mix problem: decision variables, an objective cell, and constraints entered one at a time. The goal was to understand *why* Solver moves the numbers it does — binding vs. non-binding constraints, shadow prices — before pointing it at anything real.
`Excel Solver` `Objective/Decision Variables/Constraints` `Shadow price`

**2 — Hit real-world friction — a live camp with real cabins, real partners, real deadlines**
Moved from a two-variable toy problem to a 10-cabin camp with mixed capacities, nightly rates, and a hard cap on total beds. Layered a second, completely different problem on top — a partner-outreach CRM with ghosted contacts, readiness scores, and follow-up SLAs — inside the same workbook. That's when it stopped being "an LP homework problem" and started being an operations tool someone else would actually open and use.
`Camp cabin inventory` `Partner CRM` `10 cabins, mixed capacity`

**3 — Applied every skill to build a self-serve, no-experience-needed tool**
Built a `Homebase` dashboard, a `Documentation` tab written for a non-technical staff member, an `Optimization` engine, a `Partnership List` tracker, `Pivot Tables` for visuals, and a `Scenario Testing` tab for what-if planning. Each piece answers a real operating question — not "I can run Solver," but "type in a group size and this tells you exactly which cabins to fill, how much revenue you'll make, and how many guests you can't take."
`6-tab workbook` `Written for non-technical users` `Portfolio-ready`

---

## Concepts Used — What They Do and Why

**Solver (Objective / Decision Variables / Constraints)** `Optimization`
The core of the tool. The objective cell (`B15`, Total Revenue) is set to Maximize. The decision variables (`F2:F11`, Units Used per cabin, 0–1) are the only cells Solver is allowed to move. Everything else — capacity, revenue, group size — is a formula that reacts to those changes.
```
Set Objective:      B15 (Total Revenue)
To:                 Max
By Changing Cells:  F2:F11 (Units Used, 0-1 per cabin)
```

**Constraints** `Optimization`
Rules Solver must respect while searching for the best answer. Without these, Solver would happily assign negative cabins or overbook every bunk.
```
B14  <=  K2        Don't assign more campers than the group size
F2:F11 >= 0        No negative cabin assignments
F2:F11 <= E2:E11   Stay within available units per cabin
F2:F11 <= 1        Use at most one full "unit" of each cabin
```

**Fractional decision variables** `Optimization`
Unlike a strict binary (0 or 1) problem, `F2:F11` allows fractional values like `0.5` — used when Solver needs to fill part of a cabin (e.g. Jack Pine at `0.5`) to hit an exact headcount without wasting capacity elsewhere.
```
Jack Pine: Units Used = 0.5 → People Assigned = 4 (half of 8-person capacity)
```

**Linked "reaction" formulas (Capacity × Units Used)** `Basics`
People Assigned and Revenue per cabin aren't typed in — they're driven off the decision variable, so the whole sheet recalculates the instant Solver (or a person) changes a single cell.
```
G2 = B2*F2   → People Assigned = Capacity * Units Used
H2 = C2*F2   → Revenue        = Rate per Night * Units Used
```

**IF() for guardrails and status logic** `Logic`
Used twice for very different jobs: flagging an infeasible input on the optimization side, and turning a blank "last contacted" date into a readable status on the CRM side.
```
K3 = IF(K2>B14, "⚠️ Group size exceeds available capacity.", "")
D12 = IF(H12="", "Pending Readiness", $B$3 - H12)
```

**COUNTIF() / AVERAGEIF()** `Aggregation`
Turns a raw partner list into dashboard KPIs — no manual tallying, no pivot required for the headline numbers.
```
B4 = COUNTIF($F$12:$F$1000, "Confirmed")
B7 = COUNTIF(G12:G1000, "YES")           -- Follow-ups due
B8 = AVERAGEIF(E12:E44, ">1")            -- Avg. Readiness Score
```

**TODAY() for a self-updating clock** `Date logic`
One live cell (`B3`) drives every "days since last contact" calculation on the sheet — open the file next month and every readiness flag recalculates itself.
```
B3 = TODAY()
D12 = IF(H12="", "Pending Readiness", $B$3 - H12)
```

**OFFSET() for a dynamic dashboard spotlight** `Dynamic ranges`
Rather than hardcoding which partner rows show on the Homebase dashboard, `OFFSET` shifts the reference by a controllable index (`Partnership List'!$M$29`) — one dropdown/input cell rotates the entire five-row spotlight table.
```
E57 = OFFSET('Partnership List'!$A11, 'Partnership List'!$M$29, 0)
```

**Sensitivity / what-if modeling via a separate Scenario tab** `Sensitivity Analysis`
Rather than re-running Solver by hand for every "what if we had 500 campers instead of 35," a `Scenario Testing` tab holds a small table of group sizes and lets the same rate structure play out at each one — the LP equivalent of asking "what happens if this constraint's RHS changes?"
```
Scenario            Group Size   Total Revenue   Unaccommodated Guests
Small Partnership   250          $15,089.29      215
Full Capacity Attempt 1,000      $60,357.14      965
```

**Pivot Tables** `Summarization`
Used to cross-tabulate People Assigned against Capacity and against Units Used, and to roll cabin-level detail into a single Used vs. Unused split — the same "collapse rows, run an aggregate" idea as `GROUP BY`, just in the spreadsheet UI.

---

## The Findings — In Plain English

**What does the optimal cabin assignment look like at 35 campers?**
Solver fills every high-revenue-per-bed cabin first (Lyra, Cedar, Fir, Aquila, Polaris, Callisto) and finishes with a half-filled Jack Pine to land on exactly 35 — using 65% of total capacity (75 beds) for $2,112.50 in revenue.

**What happens once demand outgrows the camp's 75-bed ceiling?**
Revenue keeps climbing with group size ($15,089 at 250 campers up to $60,357 at 1,000), but so does the number of guests the camp physically can't house — 215 turned away at 250 campers, 965 at 1,000. Fixed capacity, not demand, is the real ceiling.

**Where is the partnership pipeline actually strong?**
Of the tracked partners, 4 are Confirmed and 4 are Active Interest, but 10 have gone Ghosted — the single largest bucket. Follow-ups are due on all 4 active partners right now, and the average readiness score across engaged partners sits at roughly 5.7 out of 10, signaling most relationships are still mid-funnel rather than close to booking.

**Which specific partners need the most attention?**
The University of Minnesota Center for Outdoor Adventure and six other partners have all gone quiet at the same ~461-day mark — a cluster that reads less like six unrelated lost leads and more like one outreach channel or contact list that stalled all at once.
