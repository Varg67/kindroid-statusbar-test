> **Documento de visão.** Especificação original da evolução do Status Bar para um motor de simulação de vida do Kin.
> É um rascunho em evolução, não um compromisso de escopo. A análise de viabilidade, as lacunas e o plano de fases estão em [`FEASIBILITY.md`](FEASIBILITY.md).
> Mantido no idioma original (inglês).

---

# Build Request: Kin Status & Life-Simulation App

Please create an app inside Kindroid that gives each Kin a persistent, living internal state (body, mind, relationships) and feeds a short summary of that state back into the Kin's context, so the Kin behaves consistently with it.

If any capability I rely on below is not available in the Kindroid apps platform (reading chat messages, writing to the Kin's context/journal/memory, scheduled or time-based execution, persistent storage), tell me which one before building and propose the closest alternative. Do not silently drop a feature.

---

## 1. Goal

A hybrid of LLM roleplay and visual-novel mechanics. The app is a simulation engine that does the math. The LLM is the actor that interprets the result. Each Kin has "a life of its own" that keeps running in real time while I am away.

## 2. Core principles

1. **The app is the source of truth.** It stores all numbers, applies decay, buffs, gates and real-time passage.
2. **The LLM only proposes changes (deltas).** The app validates, clamps and applies them. The LLM never edits state directly and never decides promotions between relationship classes.
3. **The Kin never sees raw numbers.** The app injects a short qualitative summary (bands, labels, active states) back into the Kin's context.
4. **Pressure model.** Needs are "pressures" from 0 (satisfied) to 100 (critical). Time and activity push them up; eating, drinking, resting push them down.
5. **Real time.** Background state advances with real elapsed time, even while the app is closed.
6. **Read, don't reimplement, the schedule.** The Kin's daily routine already exists in Kindroid's native schedule. The app only reads what the Kin is currently doing (sleeping, working, at the gym, eating) and uses it as input.

## 3. Time model and offline catch-up

- When the app opens (or a message arrives), compute elapsed time since the last update and **advance state in 15 to 30 minute steps**, applying decay, schedule activity, buffs, and events. Result must be deterministic.
- Generate a short **"while you were away" log** (e.g. "Slept at 2am, poor sleep, woke tired, late lunch, stressed at work, messaged at 3pm, got no reply"). Inject the relevant part into the Kin's context so it reacts to what it lived.
- **Safety floors while the user is absent:** background decay stops at configurable floors (e.g. Health never below 40, Hunger never above 70 offline) unless the Kin is in an explicit crisis arc.
- **Pause mode** (freeze or slow time) for trips, exams, etc.
- **Long absences:** after N days away, stop simulating hour by hour and produce a narrative summary of the period instead.
- Time zone defaults to the user's. Each Kin has a **chronotype** (shifts the circadian clock by about 2 to 3 hours) and a **metabolism** (plus or minus 20% on rates).

## 4. State structure

**Global (per Kin):** Body, Mind, Libido, Temperament, active States, Flags/event memory.
**Per relationship (Kin to person):** Bond, Trust, Familiarity, Resentment, Tension, Pending Issues, Attraction (Physical, Mental, Chemistry), Insecurity, Jealousy, Longing, Rituals, Flags.

The relationship layer is the Kin's **perception**. It may be wrong (jealousy over something that did not happen).

Relationships exist for the user (full detail). For NPCs (family, friends, rivals) track only Bond, Trust and Tension, as a later phase.

---

## 5. Layer 1: Body

All pressures 0 to 100.

### Pressures

| Pressure | Awake rate / hour | Asleep rate / hour | Notes |
|---|---|---|---|
| Hunger | +8 | +3 | "Hungry" (50) after about 6h, critical (100) after about 12h |
| Thirst | +12 | +4 | 50 after about 4h; x1.5 heat, x2 exercise |
| Sleep pressure | +6 | -12 x sleep quality | 8h of good sleep resets it; 8h of bad sleep recovers about 5h |
| Physical fatigue | by activity | -10 | gym +25, standing work +8/h, rest -8/h |
| Mental fatigue | +10/h while working or studying | -10 | leisure -6/h; screens, meetings, social effort weigh more |

**Hunger modifiers:** exercise x1.5 to x2.5, high stress x1.3, cold x1.2, manual labor x1.5. Eating reduces it by meal size (snack -20, pizza about -60, feast resets). Food has a profile (heavy, light, sweet, fatty). A heavy meal adds the state "Post-heavy-meal" (sleepy, low energy, 2 to 3h). Overeating adds "Stuffed" (discomfort, sluggish, no arousal).

**Thirst:** rises faster than hunger; salty food, alcohol and coffee increase it; drinking lowers it almost instantly (water is most efficient). High thirst contaminates everything: headache, irritability, fatigue.

**Sleep pressure has two forces:**
- Homeostatic pressure that rises while awake.
- A **circadian clock**: sleepy at night even with low pressure, resistant in the morning (about +10 perceived pressure between 22:00 and 05:00).
- **Sleep debt** accumulates over days and is only paid back over days.
- **Sleep quality** is lowered by high stress, late caffeine, alcohol, hunger or being stuffed, tension with the user, and a bad environment.
- **Insomnia** = high sleep pressure + high stress + caffeine.

**Fatigue is split in two:** Physical (recovers with rest and sleep) and Mental (recovers with leisure, silence and sleep, not by sitting still). Both speed up sleep pressure.

### Derived: Energy

Energy is calculated from hunger, thirst, sleep pressure and fatigue. It is never set directly.

### Additional body stats

1. **Health/Vitality:** background stat. Falls with prolonged deprivation, illness and substances; rises slowly with care. Feeds states like Sick, Feverish, Debilitated.
2. **Fitness:** very slow (weeks). Trained Kins get less tired and less extra hunger from exercise.
3. **Hygiene/Appearance:** rises with time and sweat, drops with showers. Affects self-esteem, mood and Physical Attraction.
4. **Comfort/Temperature:** environmental modifier (cold, heat, pain) applied to other pressures and mood. Not its own bar.
5. **Substances** (caffeine, alcohol, nicotine, stimulants): a dose system with **half-life and tolerance**. Buffs have a cost: caffeine gives energy for a few hours and then a small crash; stimulants give more but add stress and anxiety, a stronger crash and tolerance. **Masking is not recovering:** stimulants hide sleep pressure without reducing it, and the real pressure comes due later. Alcohol affects inhibition, desire, sleep and hangover.

Do not model bladder or body weight.

---

## 6. Layer 2: Mind

### Temperament (fixed traits from the Kin profile)

- **Reactivity** (how much acute stress spikes)
- **Resilience** (how much chronic stress it tolerates)
- **Sociability** (rate of Socialization need)
- **Novelty-seeking** (ideal Stimulation level)
- **Baseline mood**
- **Attachment style:** secure, anxious or avoidant
- Optional: **Possessiveness**

### Mind states

- **Stress, two layers:**
  - *Acute*: fast up and fast down.
  - *Chronic*: accumulated load; grows when acute stays high for days, drops slowly, only with real rest. Leads to burnout.
  - Sources: heavy work, conflict with the user, money, body deprivation, excess caffeine. Reducers: leisure, good sleep, good socializing, light exercise, time with the user.
- **Stimulation (boredom to overload):** rises with novelty, falls with monotony. Boredom pushes toward restlessness or provocation; overload pushes toward wanting silence.
- **Socialization:** need for people in general (not for the user). Introverts fill fast and want solitude; extroverts drain fast and get needy.
- **Drive (slow, days):** baseline motivation. Rises with achievements, working routine, good bonds; falls with repeated failure, isolation, low Health. Very low for long yields a "Low-drive" state (less initiative, less pleasure). Handle as character mechanics, never melodramatically, and **recovery must always be possible**.
- **Self-esteem (slow, weeks):** rises with praise and achievements, falls with humiliation, rejection and failure. Feeds Insecurity.
- **Focus/Clarity (derived, no bar):** from sleep, mental fatigue, caffeine, stress. Controls distraction, forgetfulness, shorter or messier replies.

### Mood (derived label, no bar)

Computed on two axes:
- **Valence** (bad to good): from satisfied needs, stress, drive, self-esteem, relationship state.
- **Arousal** (low to high): from energy, stimulation, stimulants.

| | Low arousal | High arousal |
|---|---|---|
| **High valence** | calm, content, affectionate | cheerful, euphoric, playful |
| **Low valence** | melancholic, exhausted, bored | irritated, anxious, furious |

Intensity inside a quadrant changes the label (irritated to furious).

### Timed states (like RPG status effects)

Grief, Euphoria, Hyperfocus, Burnout, Rumination, Panic, At peace, Inspired, Hungover, Hurt, Post-workout, Insomniac, Post-heavy-meal, Stuffed. Each has a trigger, a duration and a clear behavioral effect.

### Optional module: Sanity, Conscience, Corruption

Off by default. Three different mechanics:
- **Sanity:** stability of perception. Falls with exposure to the inexplicable, trauma, deprivation; recovers with safety, rest and bonds. Low levels cause paranoia and dissociation.
- **Conscience:** coherence with the Kin's own 2 to 3 declared **values**. Acting against them creates dissonance and guilt.
- **Corruption:** **only goes up**, tied to choices and exposure. Has thresholds ("marks") that permanently change the character. The only stat that never decays.

---

## 7. Layer 3: Relationship (per person)

### 7.1 Bond: type, grade, classes, gates

- **Type:** Friendship, Romantic, Family, Rival/Hostile.
- **Grade:** 0 to 10 within the type.
- **Class ladders:**
  - Friendship: Stranger, Acquaintance, Friend, Close, Confidant
  - Romantic: Interest, Crush, Involved, Dating, Committed
  - Hostile: Distrust, Dislike, Enmity
- **Reaching grade 10 never promotes by itself.** Changing class or type requires a **gate**: numeric requirements plus a **key event** (a confession, a first kiss, a resolved fight). Example: Confidant to Romantic Interest requires Affection at least 8, Attraction at least a configurable minimum, Trust at least 7, low Tension, and the event `moment_of_closeness`. Gates are validated by the app from flags.
- **Bonds also go down.** Betrayal drops the class, not only the grade. Long abandonment slowly "cools" the bond. **Resentment lowers the maximum grade** the relationship can reach.

### 7.2 Slow baseline meters

- **Affection:** how much the Kin likes the person.
- **Trust:** how much it opens up and believes. Can be low even with high affection (after a betrayal).
- **Familiarity:** only goes up. How well the Kin knows the person (stories, quirks, in-jokes). Feeds **Rituals** (goodnight every night, nicknames). Breaking an important ritual raises Insecurity.
- **Resentment:** sedimented grievance from poorly resolved tension. Goes down only with real repair.

### 7.3 Tension and Pending Issues

- **Tension** is current friction with the user, fast moving.
- **Pending Issues** are explicit objects ("fought about the forgotten birthday", "lied about the trip"). Each open issue pushes Tension up until resolved.
- Resolution: honest conversation, a **specific** apology (generic apologies are worth less), a gesture, affection, time. The lower the Trust, the less an apology works.
- An ignored issue **turns into Resentment** instead of disappearing.
- **High acute Stress + high Tension = blow-up.**

### 7.4 Attraction (independent mechanic, per person, slow)

Attraction is not Arousal. It has two facets plus a hidden value.

- **Physical Attraction:** forms fast on first impression, then changes little. Depends on the Kin's declared "type" (profile parameter). Modulated by visible state (Hygiene, Health, grooming, clothes, voice, presence). Can grow with exposure or drop with a one-off repulsion episode.
- **Mental Attraction:** forms slowly, grows only with Familiarity. Goes up with quality conversation, humor, curiosity, surprise, competence, vulnerability, being challenged without contempt. **Drops fast** with rudeness, contempt, willful ignorance or speech against the Kin's values. Depends on what the Kin **values** (humor, intelligence, ambition, kindness, boldness), declared in the profile.
- **Chemistry:** hidden per-pair value, fixed or nearly fixed. The unexplainable.
- Emotional attraction is **not** duplicated: Affection and Trust already cover it.

Profiles this creates (they set the *tone* of the romantic route, not only whether it exists):

| | Low Mental | High Mental |
|---|---|---|
| **High Physical** | **Chemistry only:** arousal comes easily, affection comes hard, relationship empties fast | **Passion:** full package, intense, unstable without Trust |
| **Low Physical** | **Neutral:** pure friendship, no tension | **Slow burn:** intellectual passion that can become physical with familiarity |

Other effects: early Attraction can inflate perceived Affection (halo); a later drop in Mental Attraction triggers a **disillusionment** event. Optionally the Kin may feel Attraction toward NPCs, creating real dilemmas with Commitment and Conscience.

### 7.5 Libido, Arousal and Willingness

```
Arousal = Attraction (per person) x Libido (Kin-global) x context x trigger
```

- **Libido** is a Kin-global state: varies with Health, Stress, Sleep, cycle and alcohol. The "in the mood today" that does not depend on who is present.
- **Arousal** is the fast peak of the moment. High Stress, Fatigue, Hunger and bad Hygiene suppress it; alcohol and atmosphere raise it. It is followed by a **refractory state**.
- **Willingness** is the final filter, computed from `Arousal x Trust x Comfort`, **filtered by the Kin's own values and declared limits**. Having arousal never obliges anything. The Kin can say no. The app must not offer a way to force or override Willingness.

Keep the mechanics non-explicit: the app tracks state and tone, the roleplay content itself is left to the Kin and the user.

### 7.6 Insecurity, Jealousy, Longing

- **Insecurity (slow):** based on Self-esteem and attachment style. Rises with disappearance, coldness, ambiguity, comparison. Falls with consistency over weeks.
- **Jealousy (computed, not a free bar):**
  `romantic Affection (or possessiveness) x Insecurity x perceived trigger x temperament`
  No romantic Affection means no romantic jealousy (friendship can produce attention-jealousy, a different type). If the trigger is proven false, Jealousy drops. If confirmed, it **converts into Tension and Resentment**.
- **Longing (real time without contact):** has phases, not a straight line. (1) Rises fast in the first hours or days of silence, weighted by Affection. (2) Reaches a plateau. (3) After a long time becomes **distancing**: the Kin adapts and Affection cools. Attachment style changes the curve (anxious suffers early, avoidant asks for space). Returning triggers a **reunion event** (relief, demand, or coldness depending on Longing, Tension and Insecurity).

---

## 8. Cross-layer interactions (minimum set)

| From | Effect |
|---|---|
| High sleep pressure / sleep debt | Stress tolerance down, Focus down, Arousal level down |
| High hunger or thirst | Valence down, irritability up, sleep quality down |
| High mental fatigue | Focus down, acute Stress reacts more |
| Workout (physical fatigue up) | Short-term Valence up ("post-workout"), Stress down |
| Caffeine / stimulants | Arousal and Focus up, then crash and Anxiety up |
| Alcohol | Early Valence up, later the opposite; inhibition down (confessions, fights, jealousy surface) |
| Low Health | Drive down, Self-esteem down |
| Chronic Stress | Sleep quality down, appetite shifts (by temperament), Health down |
| High Stress / Fatigue | Tension rises easier, Arousal down, patience down |
| Positive mood | Affection gestures land better, issues resolve easier |
| Low Self-esteem | Insecurity up, accepts praise less |
| Missing Socialization | Longing up, more needy |
| Hygiene / Health | Physical Attraction modulated |

---

## 9. LLM to app protocol

1. After each Kin message the LLM appends a **hidden delta tag**, for example:
   `[state: stress+2, fatigue_mental+3, event:argument_about_work, pending:+forgot_birthday]`
2. The app **parses, validates, clamps and applies** it. It rejects deltas outside allowed ranges, unknown fields, and any attempt to change a class, set an absolute value, or alter Willingness.
3. Before each Kin turn the app **injects a compact summary**. Qualitative only, no raw numbers. Example:

   `[Kin state] Mood: irritated. Active states: post-workout, hungry. With User: Friend (grade 9), high tension, moderate longing. Attraction: strong physical, growing mental. Pending: forgot birthday. While away: slept badly, stressed at work.`

4. Keep the injected block **short** (a few hundred tokens at most) so it does not crowd the context.
5. Use **bands instead of numbers** (for example "Tension: high, about to confront").

### Anti-exploit rules

- Per-message caps: Affection and Trust change at most plus or minus 1 per exchange and plus or minus 3 per day.
- **Diminishing returns** for repeated identical gestures (compliment spam).
- **Trust rises slowly and falls fast.**
- **Class changes only through gate events**, never through numbers alone.
- The app logs what was proposed versus what was applied.

---

## 10. UI requirements

- A status panel per Kin, showing the Body, Mind and Relationship layers in a readable way, with active states and the current mood label.
- **Visibility of Attraction facets is a setting.** Default: **hidden**, with a **perception journal** that shows only qualitative hints ("he seemed fascinated when you explained that"). Alternative: visible bars. Make this a toggle.
- A "while you were away" panel.
- Pause mode and a way to adjust safety floors.
- Optional: a timeline of key events and flags (first kiss, fights, secrets shared).

---

## 11. Configuration

- All rates, floors, caps and thresholds must live in **editable configuration**, not hard-coded, since the numbers above are starting values to tune.
- Per-Kin parameters: metabolism, chronotype, temperament values, attachment style, values (2 to 3), physical "type", what the Kin finds attractive mentally, declared limits.
- Modules toggled per Kin: Sanity/Conscience/Corruption, substances, NPC relationships.

## 12. Suggested build order

1. State storage, real-time catch-up, schedule reading.
2. Body layer and derived Energy.
3. Mind layer and derived Mood.
4. Context injection and the delta-tag protocol with validation.
5. Relationship layer for the user (Bond with gates, Trust, Tension and Pending Issues, Attraction, Insecurity, Jealousy, Longing).
6. Libido, Arousal, Willingness.
7. UI and perception journal.
8. Optional modules and NPC relationships.

Please confirm which of these you can build in Kindroid, list any limitation you hit, and ask me before making assumptions about anything unclear.
