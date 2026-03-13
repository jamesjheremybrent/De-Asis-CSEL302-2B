# Lab 3 - Answer Sheet
## Simulating Student Attention and Performance with Agent-Based Modeling
**CSEL302 - Introduction to Intelligent Systems | AY 2025-2026**

**James Jheremy Brent B. De Asis**

---

## PART 1 - Pre-Lab Concept Questions

**1. What is an agent in an Agent-Based Model?**

In my perspective, an agent in an Agent-Based Model (ABM) is a self-directed unit with its own properties and decision rules. From the source code, each student is represented as one agent with separate attention, performance, mobility, and color values, and each one updates its movement and internal state independently per cycle.

The key idea I appreciate is that we do not hard-code the behavior of the whole class. We only define individual agent behavior, then collective patterns appear naturally from those local actions. That is what gives ABM its strength.

**2. What is the difference between global variables and species variables?**

| | Global Variables | Species Variables |
|---|---|---|
| **Definition** | Values shared across the entire simulation, readable by the world and all agents. | Values owned by each specific agent, with a separate copy per agent. |
| **Code examples** | nb_students, is_break, avg_attention, avg_performance | attention, performance, mobility, color |
| **How I picture it** | Like one class bulletin board that everyone sees. | Like each student keeping a private notebook. |

Simply put: editing a global variable impacts the full model, while editing a species variable affects only one agent. That distinction is essential when creating or revising GAMA models.

**3. What does `student mean_of each.attention` mean?**

I read this expression as: "Iterate through all student agents, collect their attention values, and return the average." It is a compact one-line calculation for class-wide mean attention at the current cycle.

In the model, the result is stored in avg_attention, so the GAMA monitor shows an updated live average each cycle. It is a clean way to aggregate agent values without manually writing a loop.

**4. What happens if attention continuously decreases without a break?**

If the break mechanism is removed, this is what I expect in sequence:

- **Attention eventually falls to zero.** Attention drops by 0.02 per cycle, and `max(0.0, ...)` keeps it from going below zero, so all students reach 0 and stay there.
- **Performance growth halts.** Once attention is no longer above 0.6, the growth condition never triggers, so performance remains at its last value.
- **Students become red over time.** The color rule sets red at attention <= 0.4, so the class gradually shifts to red, indicating disengagement.
- **The simulation settles into a stagnant state.** With attention at the floor, nothing meaningful changes anymore, reflecting a class where learning has stopped.

To me, the model is emphasizing that breaks are not optional; they are necessary to sustain both attention and learning.

---

## PART 3 - Data Observation Table (After 100 Cycles)

Based on the baseline setup (25 students, breaks every 30 cycles, decay rate 0.02, and recovery rate 0.05), these are the values I expect after 100 simulation cycles:

| Metric | Value |
|---|---|
| Mean Attention | 0.80 |
| Mean Performance | 1.0 |
| Students with High Attention (> 0.7) | 25 |
| Break Events Recorded | 3 breaks (toggled at cycles 0, 30, 60, 90) |

!![alt text](image.png) 
---

## PART 4 - Guided Code Analysis

### Activity 1: Break Frequency - Changing to 15 Cycles

When I adjust the break interval from 30 to 15 cycles with:

```
if (cycle mod 15 = 0)
```

**Does attention increase faster?** Yes. Because breaks occur twice as often, students recover attention more frequently. Recovery adds +0.05 during break cycles while work cycles only reduce by -0.02, so students spend less time in low-attention states. As a result, average attention stabilizes at a higher level than in the 30-cycle setup.

**Does performance grow faster?** Yes. Since attention stays above 0.6 more often, the performance gain (+0.01 per qualifying cycle) activates more frequently. Over 100 cycles, average performance should be clearly higher than in the base configuration.

**Is the system more stable?** Yes, especially for attention maintenance. The ups and downs are tighter, and attention does not dip as deeply between breaks. One caveat is that toggling `is_break` every 15 cycles creates a faster cadence, which may be less realistic in a real classroom schedule. Even so, overall stability improves.

### Activity 2: Attention Decay Rate - Changing to 0.05

When I modify the decay rule to:

```
attention <- max(0.0, attention - 0.05);
```

The issue is imbalance. Break recovery is +0.05 per cycle, but work decay is now also -0.05 per cycle. Since work cycles greatly outnumber break cycles (roughly 27 work cycles versus 3 break cycles in each 30-cycle period), attention trends downward overall. Performance can rise only during short post-break peaks, which is not enough for sustained academic growth.

### Activity 3: Performance Growth Condition - Threshold at 0.8

When I raise the threshold with:

```
if (attention > 0.8)
```

**Does performance improve slower?** Yes. Increasing the threshold from 0.6 to 0.8 reduces the number of students who qualify for performance gain in each cycle. Only students above 0.8 can improve, and many remain in the mid-range (around 0.4 to 0.7), so fewer agents accumulate progress.

**What does this represent in real classroom settings?** This corresponds to deep concentration or a flow-like state. It implies that moderate engagement (attention from 0.6 to 0.8) may not be enough for meaningful learning gains, and strong sustained focus is required. In practice, this can reflect the difference between passive listening and active cognitive processing. A threshold of 0.8 represents a strict learning environment where only highly focused students show measurable improvement.

---

## PART 5 - Experiment: Class Size Impact

| Students | Avg Attention | Avg Performance |
|---|---|---|
| 10 | 1.0 | 1.0 |
| 25 | 1.0| 1.0|
| 60 | 1.0 | 1.0 |
| 100 | 1.0 | 1.0 |

![alt text](image-1.png)
![alt text](image-2.png)
![alt text](image-3.png)
![alt text](image-4.png)

### Analysis Questions

**Does increasing class size affect average attention?** Not in a major way, based on this model. Because peer interaction is absent in the base version, each student behaves independently. What changes with larger class sizes is consistency: averages become more stable due to the Law of Large Numbers. Small classes (for example, 10 students) can vary more from run to run, while 100-student runs are usually more consistent.

**Does mobility create more randomness?** Yes, but mostly in visuals for this base model. Movement does not currently influence attention or performance, so numeric outcomes remain similar. If we introduced distance-based peer effects or teacher proximity effects, mobility would generate real behavioral randomness because location would directly matter.

**Is emergent behavior visible?** Yes. Even in this simplified setup, emergent patterns appear in color shifts. During breaks, green often spreads as many students recover together. During work phases, yellow and red become more common. No student is explicitly told to copy others, but the shared global variable `is_break` synchronizes transitions and creates a collective rhythm.

---

## PART 6 - Data Analysis: Attention vs. Performance Correlation

![alt text](image-5.png)
![alt text](image-6.png) 
![alt text](image-7.png)
From my reading of the model logic, performance is strongly tied to attention. My reasoning is:

- **Performance increases only when attention > 0.6.** That makes attention a strict gate; without passing it, performance cannot improve.
- For an **Attention vs. Cycle** plot, I expect a sawtooth pattern: rapid increases during break cycles and gradual decline during work cycles.
- For a **Performance vs. Cycle** plot, I expect a step-like curve: increases during attention peaks (when attention exceeds 0.6), then flat intervals during lower-attention periods.
- If we compute correlation coefficient `r`, it would likely be strongly positive (around 0.65 to 0.80) since both variables generally move in the same direction, even if their shapes differ.

So yes, performance in this model is highly attention-dependent. Improving performance requires keeping attention high, which means tuning break timing and reducing unnecessary attention decay.

---

## PART 7 - Critical Thinking Questions

**Q1: Why does performance only increase when attention > 0.6?**

I interpret 0.6 as the minimum engagement level needed for real learning. A student below that level may still be present physically, but likely not processing information deeply enough to improve academically. This aligns with cognitive load ideas where active mental effort is required, not just passive exposure.

In my view, setting 0.6 as a hard cutoff intentionally creates a nonlinear learning rule: students either meet the threshold and improve, or they do not. That makes the model more realistic and interesting than one where any tiny amount of attention leads to progress.

**Q2: Is this model deterministic or stochastic?**

I would describe it as stochastic overall. The update rules are deterministic once current values are given, but initial conditions are randomized. Each student starts with random attention, random position, and random mobility, and movement direction is random each cycle.

Because of that randomness, two runs can produce different outcomes. The model mixes deterministic rule execution with stochastic setup and behavior, which is common in ABMs.

**Q3: What real-world classroom factors are missing?**

Several important elements are not represented:

- **Peer effects** - classmates can either distract or motivate one another.
- **A teacher agent** - no instructor is present to redirect focus, ask questions, or adjust pacing.
- **Topic difficulty** - complex lessons might speed up attention loss, while easier topics might slow it.
- **Student differences** - learners do not all share the same decay/recovery rates or baseline focus.
- **Accumulated fatigue** - attention in reality is affected by longer-term tiredness, not just immediate breaks.
- **External distractions** - factors like phones, noise, hunger, and discomfort are absent.
- **Time-of-day effects** - morning and afternoon classes often show different attention dynamics.

**Q4: How would peer influence affect the system?**

Adding peer influence would likely make the simulation much more dynamic. High-attention students could raise nearby students, while low-attention clusters could pull others down. This would likely create spatial polarization, with attention "hot spots" and low-engagement zones.

From a systems viewpoint, this introduces feedback loops: a student helped by attentive peers may then help nearby classmates, creating cascades of engagement through local neighborhoods. That is a higher-level form of emergence than in the base model.

---

## PART 8 - Advanced Extension: Option A - Peer Influence


![alt text](image-8.png)

After adding this, I would expect students near highly attentive peers to gain a +0.01 bonus each cycle, which compounds over time. Across 100 cycles, this should produce two broad groups: a high-attention cluster and a low-attention cluster, influenced by initial local positions. That is true emergent behavior driven by local interaction, which matches the core purpose of ABM.

This addition also gives mobility a real role. Students moving toward engaged peers can gain attention more easily, while those drifting into low-attention neighborhoods may lose that advantage. Movement becomes behaviorally meaningful rather than purely cosmetic.
