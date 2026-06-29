# Modified Weakest-Link/Minimum Effort Model Code
ODD:
  # ODD Protocol: Weakest-Link Coordination Game ABM Suite

### Nine NetLogo Models (NetLogo 6.4.0)

 

---

 

## OVERVIEW

 

---

 

### 1. Purpose and Patterns

 

**Purpose.** The suite models how cooperative equilibrium selection emerges in a weakest-link coordination game across three *information conditions* (Personal Experience, Word of Mouth / Communication, Common Knowledge / Global) and three *initial trust structures* (Non-polarized / Basic, Polarizing Figure, Polarizing Group / In-Group). The nine resulting models form a 3×3 factorial design. The central question is whether and how fast populations converge on universal cooperation versus universal defection, and which information and trust conditions facilitate or impede that convergence.

 

**Real-world patterns the model targets:**

- How information propagation mechanisms (direct experience vs. social gossip vs. broadcast) affect collective action outcomes.

- How pre-existing trust polarization around a central figure or an in-group cohort shapes equilibrium selection.

- Why coordination failures persist even among well-intentioned actors in the weakest-link game (where the worst cooperator determines collective success).

 

---

 

### 2. Entities, State Variables, and Scales

 

**Entities:**

 

| Entity | Type | Description |

|---|---|---|

| `humans` | Turtle (agent) | The decision-making agents |

| `links` (directed) | `directed-link-breed [meetings meeting]` | Bilateral, directed trust relationships between every pair of humans |

| `patches` | Grid cells | Used as spatial markers for group co-location and (in WoM) group memory storage |

 

**Agent state variables (`turtles-own`):**

 

| Variable | Active Use | Description |

|---|---|---|

| `cooperate?` | Yes | Boolean decision for current round |

| `cooperations` / `defections` | Yes | Cumulative individual counts |

| `coop-points` | Yes | Payoff accumulator (FinalScore) |

| `partner` | Yes | Assigned groupmate for current iteration |

| `counted` | Yes | Flag preventing double-counting of group events |

| `need-partner` | Yes | Flag; reset each phase 1 |

| `reputation` | Initialized, never updated | Set to 100 at setup; not used in any decision rule |

| `coop-points` (alias `points`) | Yes | Payoff |

| `previous-group-ids` | Yes (WoM only) | IDs of last round's groupmates, used for gossip |

| `decision-made?` | Yes | Bookkeeping flag |

| `talking-with` | Initialized only | Set to `nobody`; not used in logic |

| `target`, `role`, `betrayer`, `mutual-betrayals`, `my-trustability`, `other-trustability`, `coop-rate`, `suckered`, `cheated`, `desired-partner`, `consequences?`, `patch-green-count`, `patch-all-green?` | **Declared but never set or read in any model** | Vestigial variables from earlier development |

 

**Link state variables:**

 

| Variable | Description |

|---|---|

| `thickness` | Trust level, range approximately [0.01, 0.1] (signed conceptual range −0.1 to 0.1, where red links are treated as negative in decision logic) |

| `color` | `green` = positive trust (thickness ≥ 0.05); `red` = negative trust (thickness < 0.05) |

 

**Patch state variables:**

 

| Variable | Model(s) | Description |

|---|---|---|

| `human-count` | All | Count of agents on patch |

| `compiled-previous-group` | WoM only | Aggregated list of all IDs of agents grouped here last round |

 

**Scales:**

- **Spatial:** 33×33 patch grid (−16 to 16 on each axis). Space is used only for co-location logic; no spatial distance effects on decisions.

- **Temporal:** Discrete ticks. One tick = one complete round (phases 1–4). Default run length: 2425 rounds (`number-of-meetings = 2425`).

- **Population:** Default 50 agents (`number-of-humans = 50`).

 

---

 

### 3. Process Overview and Scheduling

 

Each call to `start` executes exactly one phase per call; the procedure loops via repeated button-press or BehaviorSpace scheduling. The phase sequence within each iteration is:

 

**Phase 1 — Partner Selection (executes at start of each new iteration):**

- Increment `iteration`.

- All agents teleport to random, non-overlapping positions.

- Each agent identifies its outgoing links; preferentially selects the highest-thickness *green* link as partner. Fallback: highest-thickness link of any color (prevents isolation).

- `partner` variable is set; `need-partner` reset to false.

 

**Phase 2 — Move to Partner:**

- A `while` loop runs until every agent is co-located (distance 0) with its assigned partner.

- Patch `human-count` updated.

 

**Phase 3 — Group Decision:**

- Each agent examines its outgoing links to all co-located groupmates.

- Computes a signed mean trust: green link thickness contributes positively, red link thickness contributes negatively.

- If mean > 0: `cooperate? = true`; increments own `cooperations` and global `total-cooperations`.

- If mean ≤ 0: `cooperate? = false`; increments own `defections` and global `total-defections`.

- *Variation by information condition:* which links are examined differs (see §18).

 

**Phase 4 — Consequences (trust update + payoff):**

- Payoffs assigned:

  - All groupmates cooperate → cooperator earns `2 − (1/group-size)`.

  - Mixed group → cooperator earns −1; defector earns +1.

  - Defector always earns +1.

  - The range of possible payoff scores per agent across a full run is −25 to 49.8 points.

- Trust links updated multiplicatively (see §18).

- Global convergence flags recorded: `all-cooperate-iteration` and `all-defect-iteration` set on first unanimous round.

- Link thickness capped at 0.1.

- `tick` called; phase resets to 1.

 

**Phase 5 — Final Grouping (after `iteration = number-of-meetings + 1`):**

- Agents move to their last partner's patch for visual display.

- `print-human-table` outputs tabular results to the NetLogo output window.

 

**Phase 6 — Halt:** `stop`.

 

**Scheduling:** All `ask humans` blocks use NetLogo's default random activation order. `foreach sort` constructs are used where deterministic ordering is required (e.g., trust updates, table output).

 

**Time model:** Discrete steps. One tick per iteration.

 

---

 

## DESIGN CONCEPTS

 

---

 

### 4. Basic Principles

 

The model is built on three theoretical foundations applied at the system level:

 

1. **Weakest-link coordination game:** Group cooperation succeeds only when *all* groupmates cooperate. A single defector causes every cooperator in that group to incur a cost (−1) while the defector gains (+1). This is implemented directly in the payoff logic of Phase 4.

 

2. **Trust as an endogenous, experience-weighted heuristic:** Rather than computing expected payoffs, agents use the valence and thickness of their bilateral trust links as a proxy for the probability that a partner will cooperate. Trust evolves via a multiplicative reinforcement rule (×1.1 for confirmed expectation, ×0.9 for disconfirmation). This reflects bounded rationality / heuristic decision-making rather than full Bayesian updating.

 

3. **Information condition as a structural treatment:** The three conditions vary *whose behavior* informs trust updates — only direct experience (Exp), experience of previous groupmates as reported by current groupmates (Communication/WoM), or the behavior of every agent in the population (Global Knowledge). These are system-level structural differences, not individual adaptive traits.

 

The model is a means of *implementing* these principles in order to generate comparative outcome data, rather than a vehicle for testing whether the principles themselves hold.

 

---

 

### 5. Emergence

 

**Expected to emerge non-trivially:**

- Equilibrium type (universal cooperation, universal defection, or fragmentation) as a function of information condition and initial trust structure.

- Speed of convergence (`all-cooperate-iteration` / `all-defect-iteration`).

- Final group composition (who ends up grouped with whom after trust evolution).

- Population fragmentation into persistent cooperator and defector subgroups.

 

**More tightly imposed by assumptions:**

- The weakest-link property itself: a single defector always produces group failure for cooperators. This is structurally hard-coded, not emergent.

- The trust cap at 0.1 and floor dynamics at 0.05 (color-flip threshold) bound the state space.

- In the Global Knowledge condition, trust updates cover *all* agents regardless of group membership — this assumption strongly biases the model toward rapid convergence.

 

---

 

### 6. Adaptation

 

Agents have one adaptive trait: **the `cooperate?` decision**, which is determined each round by the signed average of trust links to current groupmates. The rule is a direct threshold function: cooperate if and only if the mean signed trust is strictly positive. This is a simple, direct objective-seeking rule — not a complex evolutionary or learning algorithm.

 

---

 

### 7. Objectives

 

Agents do not explicitly maximize any objective. The decision rule uses trust as a cooperation signal, implicitly approximating "cooperate with those I trust, defect against those I distrust." Payoffs (`coop-points`) accumulate but do not feed back into decisions. There is no fitness selection, reproduction, or payoff-based strategy updating.

 

---

 

### 8. Learning

 

Agents do not learn in the cognitive sense. Trust links are updated deterministically based on observed behavior each round, which creates a form of reinforcement:

- Cooperating partner → confirmed green trust → thickness increases; confirmed red trust → thickness decreases (toward green flip).

- Defecting partner → disconfirmed green trust → thickness decreases; confirmed red trust → thickness increases.

 

This is best characterized as **experience-weighted reinforcement of priors**, not learning in the sense of hypothesis revision or strategy switching.

 

---

 

### 9. Prediction

 

Agents use no forward-looking prediction. The cooperation decision is based entirely on the *current* signed trust toward present groupmates — a backward-looking heuristic. Agents implicitly assume "past behavior predicts present behavior," but this assumption is not modeled explicitly or with any error term. The model of the future is therefore: **none** (myopic, backward-looking only).

 

---

 

### 10. Sensing

 

**What agents sense:**

- Their own outgoing trust links (thickness and color) to all other agents.

- The identities of agents currently co-located on their patch (groupmates this round).

- In the WoM condition: the previous round's decisions of agents who shared their last patch (via `compiled-previous-group` and `round-decisions`).

- In the Global Knowledge condition: the current-round decisions of *all* agents in the population (trust update covers `other turtles`, not just groupmates).

 

**Sensing range:**

- Exp: purely local — only groupmates currently on the same patch.

- WoM: one step removed — previous groupmates' decisions as remembered by current groupmates.

- Global: global — all 49 other agents' decisions are used in trust updates regardless of proximity.

 

**Basis:** Sensing is an explicit structural assumption representing the information condition treatment. It is not based on empirical data about human information processing.

 

---

 

### 11. Interaction

 

Interactions are **direct and bilateral**, mediated by the trust link network. All agents are connected to all other agents via directed links (complete directed graph), so the network structure is always fully connected. The relevant interaction structure changes each round via the partner-selection mechanism: agents preferentially group with their highest-trust green link partner, creating emergent clustering without an explicit network-formation model.

 

Group interactions are simultaneous within each round (all phase 3 decisions are made before any phase 4 consequences are applied).

 

---

 

### 12. Stochasticity

 

Three stochastic processes:

 

| Process | Mechanism | Justification |

|---|---|---|

| Initial agent placement | `setxy random-xcor random-ycor` with rejection sampling for non-overlap | Arbitrary starting positions; no spatial structure intended |

| Initial trust link values | `random-float 0.1`, then colored green/red by threshold | Represents unknown prior history between agents |

| NetLogo `ask` activation order | Random by default | Reflects absence of priority ordering among agents |

 

No random seed is set in any model. All other processes (decisions, trust updates, payoffs) are deterministic given the current state.

 

---

 

### 13. Collectives

 

Groups form each round through the partner-selection mechanism. Each group is the set of agents who all chose the same highest-trust partner (since partner selection is mutual only by coincidence — agent A's best partner may not reciprocate). The resulting groups are **emergent**: they are not pre-specified and vary round by round as trust evolves.

 

There is no persistent collective entity. Groups dissolve and reform each round. The model tracks group-level outcomes (`all-collaborated`, `failed-collaborations`, and `GroupSizes` at end) but groups themselves are not agents.

 

In the Polarizing Figure and In-Group models, the initial trust structure creates implicit social categories (Trustful/Mistrustful or In-Group/Mistrustful/Indifferent) — these are **assumed** at initialization, not emergent.

 

---

 

### 14. Observation

 

**Data collected at end of run** (via `print-human-table` in Phase 5):

 

| Column | Level | Description |

|---|---|---|

| `AgentID` | Individual | NetLogo `who` number |

| `Cooperations` | Individual | Cumulative cooperation count |

| `Defections` | Individual | Cumulative defection count |

| `FinalScore` | Individual | Accumulated `coop-points` |

| `TotalCooperations` | Global | Sum across all agents and rounds |

| `TotalDefections` | Global | Sum across all agents and rounds |

| `FirstUnanimousCooperation` | Global | Iteration of first all-cooperate round (or "Never") |

| `FirstUnanimousDefection` | Global | Iteration of first all-defect round (or "Never") |

| `SuccessfulGroupCooperations` | Global | Count of groups where all cooperated |

| `FailedGroupCooperations` | Global | Count of groups with mixed outcomes |

| `GroupSizes` | Population | List of group sizes in final round |

| `AgentCategory` | Individual | Trustful/Mistrustful/Polarizing Figure or In-Group/Mistrustful/Indifferent (PF and IG models only) |

 

Data are collected **once per run**, at the end of Phase 5. No within-run time-series data are automatically saved (though the NetLogo GUI displays tick-by-tick agent colors and link thicknesses during execution).

 

---

 

## DETAILS

 

---

 

### 15. Implementation Details

 

- **Platform:** NetLogo 6.4.0.

- **Files:** Nine `.nlogo` files, one per cell of the 3 (information) × 3 (trust structure) design.

- **Code availability:** Provided as uploaded source files (see filenames in submission).

- **Interface parameters:** `number-of-humans` (default 50) and `number-of-meetings` (default 2425), set via input boxes. `show-trust?` toggle controls link visibility.

- **Output:** Tabular data printed to the NetLogo output window; intended to be captured via BehaviorSpace for batch runs.

 

---

 

### 16. Initialization

 

**Common to all models:**

- All globals reset to 0 or −1 sentinel values.

- `number-of-humans` agents created, placed at random non-overlapping positions.

- Each agent initialized: `cooperations = 0`, `defections = 0`, `coop-points = 0`, `reputation = 100`, `cooperate? = false`, `need-partner = true`, `talking-with = nobody`.

- A complete directed link graph is created: every agent creates outgoing links to every other agent.

- **Default trust initialization (Basic / Non-polarized):** Each link receives `thickness = random-float 0.1`. If thickness ≥ 0.05 → `color = green`; if < 0.05 → `color = red`, thickness set to `thickness + (0.05 − thickness)` (i.e., floored at 0.05). This produces an approximately even random split of green and red links.

 

**Polarizing Figure override (applied after default link creation):**

- Agent 0 is designated the Polarizing Figure.

- Agents 1 through `floor((N−1)/2)` are the Trustful group.

- Remaining agents are the Mistrustful group.

- Links between Agent 0 and the Trustful group (both directions): `thickness = 0.1`, `color = green`.

- Links between Agent 0 and the Mistrustful group (both directions): `thickness = 0.1`, `color = red`.

- Links not involving Agent 0 retain random initialization.

 

**Polarizing Group / In-Group override (applied after default link creation):**

- Agents sorted by `who`. Bottom 20% = In-Group; next 40% = Mistrustful group; top 40% = Indifferent.

- Within-In-Group links (both directions): `thickness = 0.1`, `color = green`.

- Mistrustful group → In-Group links: `thickness = 0.1`, `color = red`.

- All other links retain random initialization.

 

**Initialization is not always the same:** Initial link colors and thicknesses (for non-overridden links) are randomized each run. No fixed random seed is set. Structural overrides (PF, IG) are deterministic given agent IDs.

 

Initial values are **not based on empirical data**; they are arbitrary design choices to create the intended trust conditions.

 

---

 

### 17. Input Data

 

The models use **no external input data**. All dynamics are driven by internal agent interactions. There are no time-series inputs, no external files read at runtime.


 

### 18. Submodels

 

#### A. Partner Selection (all models, Phase 1)

 

1. Agent scans all outgoing links; filters to those with `color = green`.

2. Selects `max-one-of green-links [thickness]` as partner.

3. Fallback: if no green links exist, selects `max-one-of my-out-links [thickness]` regardless of color.

4. `partner` set to the agent at `end2` of the selected link.

 

*Parameters:* None beyond link state. No random tie-breaking — `max-one-of` selects one if tied by NetLogo's internal ordering.

 

#### B. Decision Rule (Phase 3)

 

All models: agent computes signed mean trust over co-present groupmates:

 

```

signed-trust(L) = +thickness(L)  if color(L) = green

                = −thickness(L)  if color(L) = red

avg = mean over all out-links to current groupmates

cooperate? = (avg > 0)

```

 

**Variation by information condition:**

- **Experience (Exp):** Only links to agents on the same patch this round are read (strictly local).

- **Word of Mouth (Communication):** Same local links read for the decision, but trust updates in Phase 4 use a one-step-removed gossip mechanism (see below).

- **Global Knowledge:** Links to all agents in the population read (via `foreach sort other turtles-here` in decision, but trust updates cover `other turtles` globally).

 

*Note: In the Exp models, the decision block explicitly reads `other turtles-here`; in Global Knowledge, the Phase 4 trust update covers `other turtles` (entire population), not just groupmates.*

 

#### C. Payoff Rule (Phase 4)

 

```

If cooperate? = true AND all groupmates cooperate? = true:

    coop-points += 2 − (1 / group-size)

    all-collaborated += 1  [once per group]

 

If cooperate? = true AND any groupmate defects:

    coop-points −= 1

    failed-collaborations += 1  [once per group]

 

If cooperate? = false (defector):

    coop-points += 1

```

 

#### D. Trust Update — Experience (Exp models, Phase 4)

 

Each cooperating agent updates **incoming** links from co-present groupmates only:

 

```

For each groupmate t on same patch:

  L = in-link-from t

  If self cooperated:

    If L is green: thickness *= 1.1

    If L is red:   thickness *= 0.9

      → if thickness < 0.05 after reduction: flip to green, floor at 0.05

  If self defected:

    If L is green: thickness *= 0.9

      → if thickness < 0.05: flip to red, floor at 0.05

    If L is red:   thickness *= 1.1

```

 

Link thickness capped at 0.1 globally after updates.

 

*Logic:* A cooperating self interprets partners' actions as confirming or disconfirming the link color. A defecting self strengthens distrust (red) and weakens trust (green).

 

*Note:* The update is on incoming links from groupmates, meaning the observed behavior of groupmate t updates the link from t to self — i.e., "how much do I trust t" (read from `t`'s outgoing link, but updated via t's incoming link to self, which is the outgoing link from self's perspective since links are directed). This is slightly asymmetric: the cooperate/defect branch tracks *self's* action, not the groupmate's.

 

#### E. Trust Update — Word of Mouth (Communication models, Phase 4)

 

Trust updates are deferred by one round. In Phase 4:

1. The current round's decisions are stored in global `round-decisions` (list of [who, cooperate?] pairs).

2. Each agent stores `previous-group-ids` (IDs of current groupmates).

3. Patches compile `compiled-previous-group` from all agents on the patch.

4. **On iteration ≥ 2:** Each agent on each patch loops over `compiled-previous-group` (the previous round's groupmates of anyone on this patch). For each remembered agent, the agent looks up that agent's decision in `round-decisions` and updates its outgoing link accordingly, using the same ×1.1/×0.9 multiplicative rule as Exp.

5. A `updated-links-this-tick` list prevents any single link from being updated more than once per tick.

 

*Key difference from Exp:* Agents update trust based on the *prior round's* behavior of *prior groupmates*, as recalled by *current* groupmates — a one-hop word-of-mouth propagation. This means an agent can update trust toward someone it did not directly interact with this round.

 

#### F. Trust Update — Global Knowledge (Global models, Phase 4)

 

Each agent iterates over **all other agents in the population** (`foreach sort other turtles`), updates the incoming link from each based on whether that agent cooperated this round. The ×1.1/×0.9 rule is identical, but the scope is the full population regardless of group membership.

 

#### G. Convergence Tracking

 

```

If all agents have cooperate? = true AND all-cooperate-iteration = −1:

    all-cooperate-iteration = iteration

 

If all agents have cooperate? = false AND all-defect-iteration = −1:

    all-defect-iteration = iteration

```

 

These are set once (first occurrence) and never reset.

 

#### H. Link Constraints

 

- Thickness ceiling: `if thickness > 0.1 [set thickness 0.1]` applied globally after each update.

- Color flip threshold: 0.05. When thickness crosses below 0.05 during an update, color flips and thickness is set to `thickness + (0.05 − thickness)` (i.e., = 0.05). In the WoM model, a floor of 0.01 is also applied: `if thickness < 0.01 [set thickness 0.01]`.

 
