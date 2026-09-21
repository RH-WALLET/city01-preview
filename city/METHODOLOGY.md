# METHODOLOGY

The public "how it works" page for City 01. This is the only place engine provenance is named (D-001).

## What City 01 is
A single, persistent, autonomous American city. It has been running since its founding year and never resets.
Nobody plays it: a deterministic, rule-based mayor makes every decision from the city's own statistics and a
small set of national economic inputs. One real hour is one city month.

## The national economy

**Where the numbers come from.** Up to the last real quarter (2026Q1 in the current data vintage) every national
figure City 01 uses — GDP, the output gap, unemployment, core PCE inflation, the federal funds rate, the 10-year
yield, the mortgage rate, residential and business investment, consumption — is real US history taken from the
Federal Reserve Board's public FRB/US database (HISTDATA / LONGBASE, vintage 2026-08-05). Those quarters carry the
badge **HISTORICAL DATA**.

**After the NOW line.** From the next quarter on, the economy is produced by the Board's public **FRB/US** model,
run with the Board's public **PyFRB/US** package exactly as shipped: backward-looking ("VAR") expectations, the
model's default inertial Taylor rule for the funds rate, the standard surplus-ratio fiscal setting used in the
Board's own example programs, and the Board's published baseline as the starting point. The model is advanced
**one quarter at a time**. Those quarters carry the badge **FRB/US SIMULATION**.

**What a residual draw is.** Every equation in FRB/US explains part of a variable and leaves a leftover — the
residual — for each historical quarter: the part of that quarter's GDP, prices, employment, interest rates and so
on that the equations did not predict. A quarter's residuals, taken together, are the "surprise" the US economy
experienced in that quarter. Each simulated quarter, City 01 draws one historical quarter between 1975 and the
last real quarter (the two pandemic quarters of 2020 excluded — the Board's database treats them as a one-off
break in the natural rate of unemployment) and applies that quarter's surprises, centred so that they average to
zero over the sample, to the model. So the simulated economy is hit, in random order, by the same kinds of
surprises the real economy has been hit by since 1975: a quarter of 2008Q4's shocks, then a quarter of 1997Q1's,
and so on. The name of the drawn quarter is published with every simulated quarter.

**Which quarter gets drawn.** The draw is a fixed arithmetic function of a public random number — the drand
randomness beacon's round for that quarter — and the quarter's label. Nobody, including the operators, can
choose or preview it; anyone can recompute it from the published beacon round. If the model fails to solve for a
draw, the next draw is again a fixed function of the same beacon value, and each retry is recorded.

**One random path, not a forecast.** The simulated economy is one randomly drawn path among the enormous number
FRB/US could produce. It is not the Federal Reserve's forecast, not the FOMC's, not the Board staff's, and not
ours. It will contain recessions, booms, long stretches at the interest-rate floor and other episodes that follow
from the drawn surprises, and it will not track the actual future US economy.

**Not affiliated.** City 01 is not affiliated with or endorsed by the Federal Reserve Board. FRB/US, PyFRB/US and
the LONGBASE/HISTDATA databases are public resources published by the Board; our use of them implies no
endorsement, and any errors in how we run them are ours.

**Small print, documented in `macro/README.md`.** The Board's baseline ends in 2176; beyond it the baseline is
extended along its own steady state (constant growth rates for levels, constant values for rates and ratios) and
the simulation continues unchanged. The recession shading uses a quarterly Sahm rule (unemployment at least
0.5 point above its low of the previous four quarters) with a documented episode rule; on real history it marks
exactly the eight US recessions since 1968. Under random i.i.d. draws the simulated economy shows more, shorter
Sahm episodes than the historical record (about 35 rather than 14 per century) because real shocks arrive in
clusters and random draws do not.

## From nation to city
National variables are mapped through clamped, smooth functions (published in `host/mapping.config.json`) to a
handful of city multipliers: residential / commercial / industrial demand, migration, tax base, a cost index,
and a borrowing rate. Each is `1 + gain·tanh(deviation/scale)` of the variable's deviation from its own trailing
five-to-ten-year trend, so what moves the city is the *surprise* in housing investment, consumption, business
investment, unemployment and the output gap — not their levels (growing series are detrended with a drift-corrected
trailing trend, so a steady 3 %/yr is zero surprise). Core prices become a cost index on everything the city buys
— the engine itself charges tool prices and service budgets times the index — and on tax revenue, so the city's
books inflate with the nation while relative prices stay the engine's; the city's cash balance earns the funds
rate less a spread, and the 10-year yield plus a spread is its borrowing rate (debt is nominal, so inflation erodes
it). The mapping is deterministic and documented; there is no hidden tuning at runtime.

## History: blocks, events, districts
Every zone is a *block* with a life story: zoned, built, densified or declined, unpowered, derelict, burnt,
demolished — decoded from the engine's own tiles each month (zone.cpp's density/value encoding), never inferred.
Events are of two kinds: instants (a plant opens, an earthquake, a tax rise, a Fed move of ≥ 100 bp, the city
becoming a Metropolis) and intervals (fire, blackout, fiscal crisis, recession) that open and close with
hysteresis and are titled only once they end, with their final numbers. Each event gets a significance score
`S = 0.35·z + 0.25·scope + 0.20·firstness + 0.20·recordness` — how unusual its magnitude is against all earlier
events of its kind (Welford running statistics), how much of the city it touched, whether it is the first of its
kind, whether it set a record — scaled by a per-kind salience prior, and is filed into one of three tiers (feed,
chronicle, historic) by thresholds calibrated on harness runs so that a 300-year city has roughly 40–150 historic
events; at most two per year unless S > 0.9, and never two of the same kind within two years. Titles are
templates ("{District} Fire of {Year}", "Great" only when the magnitude exceeds three times the previous record)
and unique within the chronicle. Districts are connected 8×8-cell regions of a dominant use, named from geometry
(Harbor/Riverside/Old/Central/compass octant + a use word in founding order), and keep their identity year to
year by cell overlap (Jaccard > 0.5), through a two-year grace and a thirty-year revival window; they merge, fade
and come back as events. Nothing here is written by a language model; every sentence is a template over numbers
the simulation produced.

## Money
The engine's tax yield (population × land value × tax rate, the original 1989 constants) is scaled ×3 for City 01
(`economy.revenueScale`, D-047). Without it a city founded in 1970 with $20,000 could afford about a hundred
blocks by 2007; with it the same rules compound into a mid-sized city. Costs, prices and the macro inflation index
are untouched, so relative prices stay the engine's own. Every dollar figure on the terminal is the engine's
number under this calibration; none is invented.

## The mayor
Rule-based, bounded on purpose: monthly decisions on last month's statistics, a yearly budget, no lookahead,
never demolishes occupied zones. Every month it reads the city's own sensors (demand valves, power utilization,
fire and police coverage, crime, engine messages) into needs from 0 to 100, generates candidate sites from eight
block templates, scores them (land value, proximity to demand, pollution, terrain, power reach), passes them
through a budget gate, executes at most three capital actions and writes an audit row with the top three
candidates and their scores — that row is what "Why was this built?" shows. The yearly budget follows fixed
rules for tax, service funding and a debt ledger that only borrows for essential infrastructure. No AI or
language model runs anywhere in the system (D-002).

## Determinism
Same seed + same national inputs ⇒ bit-identical city. Randomness enters only through the persisted engine RNG
(seeded once at founding) and the recorded macro inputs. Snapshots are hash-chained; anyone can replay.

## Time, and where the dice come from
One real hour is one city month: the engine's 16 phases × 4 ticks per month become one phase every 56.25 seconds,
anchored to the opening moment. If the process falls behind (a restart, a slow boundary) it catches up at no more
than one city tick per second and publishes as it goes; it never skips a phase. Every quarter boundary that lies past
the last real quarter of the national data draws its historical residual quarter from public randomness: the
drand beacon (quicknet, a 3-second public randomness chain run by the League of Entropy) at the round that
corresponds to the boundary's *scheduled* wall-clock time. The round, its randomness and its signature are recorded
with the quarter, so the draw can be checked against any drand relay and a late or restarted loop reads the same
archived round and produces the same quarter. Until a boundary's quarter exists the city waits at that boundary
and the terminal says so; it never advances on values it does not have. Historical quarters come straight from the
record (`macro_cli get`). The published macro series lists every quarter with its source and its draw.

## Provenance
- **City simulation engine:** a server-side fork of *MicropolisCore* (SimHacker), the GPL-3 release of the
  original 1989 city-simulation source, with Electronic Arts' additional terms. Our fork and host are GPL-3
  and published (D-011). The engine never runs in the browser; browsers receive only published snapshots.
- **Macro model:** FRB/US + PyFRB/US, Federal Reserve Board (public domain, US Government work).
- **Randomness beacon:** drand (League of Entropy).
- **Art:** Penzilla Design isometric packs (royalty-free commercial license, purchased) composed into our own
  block sprites; placeholder art until then is generated in-repo.

## Cosmetic layer (D-010)
Everything below is presentation only and has **zero** effect on the simulation. Updated as features land.
- **Vehicles** on roads: their number follows the engine's real traffic (road tile traffic level and the traffic
  density map); their routes are a random walk and mean nothing.
- **Pedestrians** on sidewalks: their number follows the engine's real population density map around each road;
  who they are and where they go is random.
- **Smoke** rises only from tiles the engine says are burning and from powered industrial zones.
- **Speech bubbles** quote real events from the city's feed (the same titles as the ticker), placed over the block
  the event happened on. No bubble is ever invented.
- **Shadows**, **street furniture** (lamps, hydrants, signs on straight road segments), **lot props** (bushes,
  benches, trees on vacant land next to roads worth more than a threshold on the land-value map), **cloud
  shadows**, the **day/night cycle** (local time), **seasons** (city month) and the **backdrop** beyond the map edge.
- **The press conference** at each quarter: the chair is a caricature supplied by the owner; the podium badge is
  City 01's own. Every sentence of the statement is filled from the quarter's published numbers (the FRB/US
  rule terms, inflation, output gap, unemployment, the drand draw) — the wording is ours, the numbers are the
  model's, and the "Sources" toggle shows the field behind each line. It is not a statement of any real
  central bank or person. The voice reading it is synthetic (Kokoro, an open text-to-speech model run on our
  own server; or the browser's voice when no clip exists) — it is nobody's real voice.
- **The camera**: the world is 600×500 tiles and the city grows into it over years; the viewer streams the
  ground around the camera, cannot zoom out past 0.4× and cannot leave the built-up area plus a margin of
  countryside. Nothing about the simulation depends on what is on screen.
- The art itself: hand-drawn isometric sprites; buildings are chosen by zone type, density and land value class
  as the engine reports them, and a block's look is picked from a few variants by hashing its position.

## The token, deeds and the reserve fund
The token and its holders have no input to the simulation (D-007): the viewer, protocol and token code never
import the engine or host, and a hash-equality test proves the city runs identically with the token process
on or off. **Deeds** are names on blocks — a wallet that holds a tier's amount can sign in (a signature, no
transaction) and claim a block; the deed lapses a day after the balance drops. A deed changes nothing on the
map except the nameplate. **The reserve** buys and burns the token when the city's own Fed cuts rates, and
does nothing on a hike or hold; every action is listed with its transaction hash. **The reserve fund** trades a
basket of other tokens on the same decisions (risk-on on a cut, cash on a hike or recession). When that is
switched on, a quarter's realized profit — never a loss — is credited to the city treasury at the next quarter
as a recorded grant, capped at half a year's tax revenue, and shown on the decisions page as
`grant.reserveFund`. That is the one channel through which anything outside the macro record touches the
city's money (D-049); until it is switched on, it does not exist.
