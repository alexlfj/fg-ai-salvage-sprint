# Week 1 evidence

Keep this short. Bullet points, a small table and one diagram or code trace are enough. This is your checkpoint record, not an extra report. Replace the prompts as you work.

| | |
|---|---|
| **Name** | Alexander L F Johnsson |
| **Exact Unity version** | 6000.3.26f1 |
| **Checkpoint commit / repository** | https://github.com/alexlfj/fg-ai-salvage-sprint (checkpoint commit: `db15c5d` "week1: working baseline") |
| **Current stage and anything unfinished** | Stage 0 done (scoring, win result and restart work). Stage 1 in progress. DecisionPolicy is still the starter ("nearest candidate"). Fixed a supplied bug in `Runtime/Navigator.cs` that stopped the rival from moving (see Collaboration). |

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

`MatchController` confirms the pickup and gives the rival cargo. On the next decision tick `Capture` reports Cargo = True, `Infer` returns NeedsDelivery = True, and `Choose` takes the delivery branch first, so the task switches **Collect → Deliver**.

Observed: the rival picked up 11, switched to Deliver and delivered it to the depot.

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

---

## Collaboration and help

**People I discussed or playtested with:**

**Code, sources or substantive tools/AI assistance I used, and how I verified them:**

- **Infrastructure fix:** after the teacher's 24 Nov "Improve readability" commit, the rival never moved. Line 67 of `Runtime/Navigator.cs` had a misplaced parenthesis. I put the `)` back after `Pending`, checked that the rival moves, collects and delivers in 01_Normal, and told the teacher. CoreTests still pass.
- **AI assistance (Claude):** helped me get the tests running in Unity's Test Runner and helped structure and format this file from my own answers and screenshots. Predictions and observations are my own.

**If unfinished: what works, exact reproduction steps, and the help I need:**
