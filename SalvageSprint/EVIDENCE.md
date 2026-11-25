# Week 1 evidence

Keep this short. Bullet points, a small table and one diagram or code trace are enough. This is your checkpoint record, not an extra report. Replace the prompts as you work.

| | |
|---|---|
| **Name** | Alexander L F Johnsson |
| **Exact Unity version** | 6000.3.26f1 |
| **Checkpoint commit / repository** | https://github.com/alexlfj/fg-ai-salvage-sprint (checkpoint commit: `db15c5d` "week1: working baseline") |
| **Current stage and anything unfinished** | Stage 0 done (scoring, win result and restart work). Stage 1 done. DecisionPolicy is still the starter logic; only the Label and Collect reason changed. Fixed a supplied bug in `Runtime/Navigator.cs` that stopped the rival from moving (see Collaboration). |

---

## Initial prediction

**Three observations the rival needs, including one it might not have:**

1. If the scrap is visible or not
2. Remember a list of all the scraps it has seen
3. If it's currently carrying scrap or not

What it cannot do:

4. Know if it can reach a scrap or not

**Seeing a pickup across a gap proves / does not prove reachability because:**

Does not prove. Seeing does not mean being able to get to the place.

**One question about NPC behaviour:**

How does the Nemesis System work in the Shadow of Mordor/War games?

---

## One complete trace

Scene 01_Normal, starter policy, two pauses with the F1 debug panel (25 Nov).

### Pause A: heading for scrap

- **Observation** (`PickupSensor.Capture`, called in `RivalBrain.Update` l.66): Cargo: False, Visible: 4, Memory: 1, Blocked: 0
- **Facts** (`DecisionPolicy.Infer`, l.67): NeedsDelivery: False, UsableTarget: True (target 11)
- **Task + reason** (`DecisionPolicy.Choose`, l.68): Collect 11, "Nearest candidate; reselected each tick"
- **Movement request** (`RivalBrain.Update` l.82-87 → `Navigator.Request`): Destination = pickup 11 position
- **Result** (`Navigator.Poll` → `NavigationFeedback.Classify`): Travel: Moving, Last failure: None
- **Counters:** Decisions 23, Requests 3

### Pause B: carrying scrap

- **Observation** (`PickupSensor.Capture`, called in `RivalBrain.Update` l.66): Cargo: True, Visible: 2, Memory: 0, Blocked: 0
- **Facts** (`DecisionPolicy.Infer`, l.67): NeedsDelivery: True, UsableTarget: False
- **Task + reason** (`DecisionPolicy.Choose`, l.68): Deliver, "Cargo held: return to depot"
- **Movement request** (`RivalBrain.Update` l.77-80 → `Navigator.Request`): Destination = depot (`Request(-2, DepotPosition)`)
- **Result** (`Navigator.Poll` → `NavigationFeedback.Classify`): Travel: Moving, Last failure: None
- **Counters:** Decisions 33, Requests 4

### Feedback into a later decision

The rival reached target 11. `MatchController.Update` checked that the item was available and within distance, then called `MatchState.TryCollect`, which set `cargo[team] = true`, meaning the rival is carrying the scrap. On the next tick, `PickupSensor.Capture` passed that value into the Observation as `Carrying`. `DecisionPolicy.Infer` turned it into the fact `NeedsDelivery = true` (target -1, since there's nothing to collect while carrying). In `DecisionPolicy.Choose`, the first `if` runs and returns the Deliver task, so the task switched from Collect to Deliver.

Observed: the rival picked up 11, switched to Deliver and delivered it to the depot.

### Stage 1 navigation notes

**Travel status predictions** (compare `CoreTests.PendingPathIsNotMistakenForFailureOrArrival`):

- Pending computation, unknown distance: Pending
- Partial route, even near its endpoint: Failed
- Complete route, four units away: Moving
- Complete route, within the stopping threshold: Arrived

**Decisions vs route requests** (33 decisions, 4 requests in Pause B): It would have to re-process absolutely everything instead of just continuing. You only request when you need to.

**Does Arrived prove collection?** I mean, it will always arrive if it reaches a specific destination. Whether there is something to pick up or drop off is irrelevant.

**Moving a shelf and rebaking:** Prediction: the blue walkable area will adapt to the new position, making the new available floor blue and making the space the shelf is now occupying unavailable for traversal. Observed: matched the prediction. Shelf restored afterwards.

**Policy label and reason:** changed `Label` to "Part 1 starter: nearest candidate within range" and the Collect reason to "Nearest candidate within range; reselected each tick".

---

## Scenario record

| Scenario | Predicted result | Observed result and short evidence / remaining issue |
|---|---|---|
| Normal loop and restart | | |
| Carrying cargo | | |
| Hidden / remembered pickup | | |
| Target taken | | |
| Unreachable target | | |
| No usable pickup | | |

**Supplied test result, with any added tests identified:**

Starter: 11 pass, 6 fail (Unity Test Runner, EditMode), 25 Nov. All 7 CoreTests green. The failing ones are the Stage 2-4 policy tests. No added tests.

**Live walkthrough checked by / recording link if used:**

---

## One correction and one independent check

**The original weakness and why it happened:**

**What I changed and why it addresses the cause:**

**My counterexample or experiment: prediction, observed result, and one limitation:**

Stopping-distance experiment (Stage 1 step 6): Rival stopping distance changed from 0.6 to 1.6, collection radius left at 1.1.

- **Prediction:** It will stop and not pick up the scrap.
- **Observed:** I predicted that it would stop and not pick up, but it still did. That is because it was just the label that the Navigator reports. It was saying that the rival "Arrived" before it actually did.
- **Limitation:** I added 6.6, and it completely stopped. At 2.6 it moved but didn't collect the scrap. So the point where it stops collecting is somewhere between 1.6 and 2.6, and I didn't pin it down.
- Restored stopping distance to 0.6.

---

## Collaboration and help

**People I discussed or playtested with:**

**Code, sources or substantive tools/AI assistance I used, and how I verified them:**

- **Infrastructure fix:** after the teacher's 24 Nov "Improve readability" commit, the rival never moved. Line 67 of `Runtime/Navigator.cs` had a misplaced parenthesis. I put the `)` back after `Pending`, checked that the rival moves, collects and delivers in 01_Normal, and told the teacher. CoreTests still pass.
- **AI assistance (Claude):** helped me get the tests running in Unity's Test Runner, asked guiding questions and pointed me to where to look in the code, and assembled this EVIDENCE.md from my answers. The predictions and observations are my own.

**If unfinished: what works, exact reproduction steps, and the help I need:**
