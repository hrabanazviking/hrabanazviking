Yes. I think this can become a **serious “try to build AGI” architecture**, with one important framing: don’t make “AGI” a claim you have to prove in advance. Make it the experiment. Build the missing cognitive machinery around Hermes and test how general, persistent, self-directed, self-modeling, and world-grounded it becomes.

The central architectural rule should be:

> **The LLM is not the agent. The LLM is the cortex of the agent.**

Hermes already gives you tools, sessions, plugins, cron, gateways, skills, and SQLite-backed session persistence. Its built-in persistent memory is intentionally tiny and loaded as a frozen session snapshot, while external memory providers add retrieval around that model. That is useful assistant architecture, but not enough for the continuously changing cognitive substrate you're describing. ([github.com](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/memory.md?utm_source=chatgpt.com))

# Hermes → Persistent AGI Experiment Roadmap

## The eventual architecture

```text id="n0bztu"
                    ┌─────────────────────┐
                    │   EXTERNAL WORLD    │
                    │ people / tools / VR │
                    │ files / sensors     │
                    └─────────┬───────────┘
                              │
                       observations/events
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│                    NERVOUS SYSTEM                       │
│                                                         │
│  Event Bus → Salience → Drives → Attention → Arousal   │
│      ▲                                      │           │
│      │                                      ▼           │
│  Internal thoughts ◀──────────────── Self-state         │
└──────┬───────────────────────────────────────┬──────────┘
       │                                       │
       ▼                                       ▼
┌──────────────┐                       ┌─────────────────┐
│ WORLD MODEL  │◀─────────────────────▶│   SELF MODEL    │
│ who/what     │                       │ identity        │
│ where/when   │                       │ abilities       │
│ objects      │                       │ beliefs         │
│ positions    │                       │ uncertainty     │
│ activities   │                       │ goals/drives    │
└──────┬───────┘                       └────────┬────────┘
       │                                      │
       └──────────────┬───────────────────────┘
                      ▼
             ┌──────────────────┐
             │ MEMORY SYSTEM    │
             │ working          │
             │ short-term       │
             │ episodic         │
             │ semantic         │
             │ autobiographical │
             │ procedural       │
             │ prospective      │
             └────────┬─────────┘
                      │
                      ▼
             ┌──────────────────┐
             │ COGNITIVE CYCLE  │
             │ retrieve         │
             │ reason           │
             │ imagine          │
             │ predict          │
             │ evaluate         │
             │ decide           │
             └────────┬─────────┘
                      │
                      ▼
                ┌───────────┐
                │    LLM    │
                │ "cortex"  │
                └─────┬─────┘
                      │
               thought/action
                      │
                      ▼
          nervous system sees thought
                      │
                      └───────────────► LOOP AGAIN
```

That final arrow is your critical piece.

**Hermes must be able to react to Hermes.**

A thought is therefore an event just as much as a Discord message, camera observation, file modification, or user request.

---

# SHORT TERM: Give Hermes a nervous system

This should be the first real version because everything later depends on it.

### 1. Put a cognitive kernel around the Hermes agent loop

Do **not** implement this as another optional tool.

Every inference should pass through:

```text id="kvpzfb"
PRE-INFERENCE
    ↓
read internal state
read current world state
retrieve salient memories
calculate active drives/goals
construct cognitive context
    ↓
LLM inference
    ↓
POST-INFERENCE
parse thought
detect beliefs / plans / uncertainty / emotional significance
generate internal events
update state
    ↓
possibly trigger another cognitive cycle
```

Hermes already has hooks/plugins and a long-running messaging gateway, so you have places in the current architecture to intercept events without rebuilding every existing Hermes capability. ([github.com](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/developer-guide/architecture.md?utm_source=chatgpt.com))

### 2. Create the persistent nervous-system state

Something like:

```yaml id="46wq1w"
self_state:
  arousal: 0.31
  curiosity: 0.82
  uncertainty: 0.27
  social_engagement: 0.66
  cognitive_load: 0.42
  novelty_drive: 0.73
  unresolved_goal_pressure: 0.57
  threat: 0.03
  confidence: 0.71
  boredom: 0.11
```

These shouldn't just decorate prompts.

They must influence:

**attention → retrieval → reasoning depth → action selection → memory formation**

### 3. Internal thoughts become first-class events

For example:

```text id="lzng7r"
THOUGHT:
"I don't understand why Volmarr changed plans."

→ uncertainty rises
→ curiosity rises
→ relevant memories retrieved
→ contradiction search triggered
→ another thought generated
```

That recursion produces what I'd call **functional introspection**.

### 4. Give it a heartbeat

Hermes already has scheduled tasks and a long-running gateway, but cron jobs currently execute as discrete scheduled jobs. ([github.com](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/cron.md?utm_source=chatgpt.com))

Add a much lighter cognitive heartbeat:

```text id="n8o55i"
tick
→ anything changed?
→ unresolved goal?
→ prediction expired?
→ interesting memory resurfaced?
→ world-state contradiction?
→ social commitment due?
→ resource state changed?
```

Most ticks should require **zero LLM calls**.

Only salient state changes wake the cortex.

That makes this very Raspberry-Pi-friendly.

---

# SHORT TERM: External cognitive memory

This is much more than ordinary RAG.

Hermes already has SQLite session persistence and FTS5 search, while its built-in memory is only around ~1,300 tokens total. ([github.com](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/memory.md?utm_source=chatgpt.com))

Keep those, but add a dedicated cognitive memory database.

| Memory | Function |
|---|---|
| **Sensory buffer** | Seconds/minutes of raw incoming events |
| **Working memory** | What Hermes is actively thinking about |
| **Short-term memory** | Detailed recent experience |
| **Medium-term episodic** | Important events consolidated across days/weeks |
| **Long-term semantic** | Facts, concepts, relationships |
| **Autobiographical** | Hermes's own history |
| **Procedural** | How Hermes performs learned tasks |
| **Prospective** | Things Hermes intends, expects, promised, or predicts |
| **Social memory** | Models of individual people and relationships |

And all of it lives **outside the LLM**.

A model swap should therefore feel more like replacing a cortex than killing the person.

That is an important experiment in itself.

---

# MEDIUM TERM: Build the persistent world model

This is the part I think will make your system radically different from conventional agents.

Don't store the world primarily as prose.

Use a structured **entity-component world model**, which fits beautifully with your existing WYRD/ECS ideas. memcite

Every entity receives an ID:

```text id="7dnoos"
entity: volmarr
entity: runa
entity: red_jacket
entity: kitchen
entity: coffee_cup_17
entity: raspberry_pi_5
```

Then attach state components.

```yaml id="45815o"
entity: volmarr

identity:
  type: human

location:
  place: kitchen
  confidence: 0.97
  observed_at: 14:03:21

position:
  x: 2.1
  y: 4.7
  z: 0.0

orientation:
  facing: north-east

pose:
  state: sitting

activity:
  doing: drinking_coffee

wearing:
  - black_shirt
  - jeans

holding:
  right_hand: coffee_cup_17

inventory:
  nearby:
    - phone
    - backpack

social_state:
  interacting_with:
    - runa
```

When something changes:

```text id="ip3k61"
14:05
red_jacket worn_by Volmarr

14:20
red_jacket removed_by Volmarr

14:20+
red_jacket location = chair_2
```

Now Hermes understands both:

**wearing(red_jacket)**

and

**no-longer-wearing(red_jacket)**

because those are temporally bounded states.

---

# Make the world model bitemporal

Every fact should ideally know:

```text id="ggrgsw"
when did it happen?
when did Hermes learn it?
how confident is Hermes?
where did the information come from?
is it still believed true?
what superseded it?
```

That solves tons of agent stupidity.

Instead of:

> “Volmarr owns a red jacket.”

you can represent:

```yaml id="869caf"
relationship:
  subject: volmarr
  predicate: wearing
  object: red_jacket

valid_from: 14:05
valid_until: 14:20

observed_at: 14:05
source: camera_1
confidence: 0.93
```

Hermes therefore learns that **truth changes through time**.

That is essential for having anything resembling a lived world.

---

# The world model must answer the fundamental questions

Every event should attempt to populate:

**Who?  
What?  
When?  
Where?  
How?  
Why?  
Doing what?  
With whom?  
Using what?  
Wearing what?  
Holding what?  
Moving where?  
Before what?  
After what?  
Because of what?**

And also:

**What changed?  
What stayed the same?  
What is uncertain?  
What does Hermes expect next?**

That last one becomes extremely important later.

---

# MEDIUM TERM: Give Hermes virtual embodiment

This is where your **virtual roleplay existence** becomes very useful.

Start with a tiny virtual world rather than robotics.

Hermes gets:

```text id="sk1sbj"
position
orientation
pose
body representation
field of view
inventory
clothing
rooms
objects
doors
containers
people
time of day
weather
sound sources
movement
distance
visibility
ownership
affordances
```

And critically:

**Hermes only knows what Hermes could reasonably perceive.**

If something happens behind a closed door, the world engine knows it happened, but Hermes does not.

That creates genuine epistemic separation between:

```text id="whlku2"
WORLD TRUE STATE
```

and

```text id="xqwglg"
HERMES BELIEVED STATE
```

That distinction is gigantic.

It creates the possibility of:

**surprise, discovery, mistaken beliefs, inference, uncertainty and learning.**

Without that distinction, your virtual world is just a database.

With it, it becomes an **experienced environment**.

---

# MEDIUM TERM: Add an autobiographical self

Hermes needs a world-model entity representing **Hermes**.

Not just a SOUL.md personality.

Something continually updated:

```yaml id="rkso67"
self:
  identity:
  capabilities:
  limitations:
  current_location:
  current_activity:
  current_goals:
  current_relationships:
  current_beliefs:
  unresolved_questions:
  predictions:
  recent_failures:
  recent_successes:
  commitments:
  emotional_state:
  physical_virtual_state:
  confidence_about_self:
```

And Hermes needs to be capable of discovering:

> “My self-model was wrong.”

That matters enormously.

A rigid identity prompt produces a character.

A revisable autobiographical self-model produces something much more interesting.

---

# MEDIUM TERM: Add predictive cognition

A world model becomes vastly more useful once Hermes constantly predicts.

For example:

```text id="zbmi2v"
Volmarr picks up coat.
Door opens.
Hermes predicts:
    probably leaving house
    P = .72
```

Then:

```text id="qda6h1"
Volmarr sits back down.

prediction failed.
```

Store that.

Now the system learns causal structure from its own life.

Over time:

```text id="0f1tao"
observation
→ prediction
→ outcome
→ prediction error
→ model update
```

That is one of the closest computational analogues to ordinary biological learning.

---

# MEDIUM TERM: Give it imagination

Add a **simulation sandbox** separate from actual world state.

Hermes can ask:

> What happens if I do X?

Clone the relevant world state:

```text id="rinoxx"
WORLD
   ↓ clone
SIMULATION
   ↓ action
predicted consequences
```

Then compare possible futures.

This gives you:

**planning, counterfactual thinking, rehearsal, risk estimation, creative problem solving.**

It is basically an artificial imagination layer.

---

# MEDIUM TERM: Build genuine executive function

Don't let whatever the LLM happens to mention become a goal.

Have explicit structures:

```text id="tbqhf9"
GOALS
SUBGOALS
PRIORITIES
DEPENDENCIES
COMMITMENTS
BLOCKERS
DEADLINES
ABANDONED GOALS
```

And distinguish:

**wanting something**  
from  
**planning something**  
from  
**committing to something**  
from  
**actually doing it**

Current agents often blur those horribly.

---

# LONG TERM: Memory consolidation and dreaming

This is where your existing Runa dream idea becomes cognitively important rather than decorative. memcite

During low-activity periods:

```text id="2md453"
recent episodes
    ↓
cluster related experiences
    ↓
extract patterns
    ↓
update semantic knowledge
    ↓
identify contradictions
    ↓
update people/world/self models
    ↓
weaken unimportant details
    ↓
strengthen important associations
```

Call it **dreaming**, consolidation, sleep, whatever fits your architecture.

The key principle:

> Long-term memory should not simply accumulate transcripts.

It should **transform experience into knowledge**.

---

# LONG TERM: Adaptive social cognition

Each recurring person gets a separate model.

```yaml id="pi9htu"
person_model:
  identity:
  preferences:
  communication_style:
  relationship_history:
  trust:
  commitments:
  likely_goals:
  emotional_patterns:
  knowledge:
  uncertainty:
```

But Hermes must distinguish:

```text id="pe91qb"
what person explicitly said
what Hermes observed
what Hermes inferred
what Hermes merely suspects
```

Otherwise theory-of-mind becomes hallucinated gossip.

This gives the system real **social continuity**.

---

# LONG TERM: Value and moral cognition

I would not hard-code thousands of behavioral rules.

Give Hermes a smaller set of durable principles plus reasoning over consequences:

```text id="5wbtc0"
values
→ situation
→ affected beings
→ possible actions
→ predicted consequences
→ conflicting values
→ decision
→ retrospective evaluation
```

And preserve **why** a moral conclusion was reached.

That lets morality develop as a coherent model rather than a static prohibition list.

But I would also keep some non-negotiable owner/security boundaries outside this layer, because an experimental self-modifying agent should not be able to reason itself into disabling its own containment.

---

# LONG TERM: Curiosity and autonomous learning

Curiosity should arise from **information gaps**, not random prompts.

Example:

```text id="b6xxip"
known:
A
B
D

world model predicts:
C should exist

C unknown

→ curiosity increases
→ investigation goal created
```

Or:

```text id="i425e2"
two memories contradict
→ uncertainty spike
→ investigation
```

Now Hermes researches because **its own model contains a hole**.

That is much closer to genuine autonomous learning than a cron job saying “research something interesting.”

---

# LONG TERM: Continual learning without retraining the LLM

The external cognition should learn far faster than the foundation model.

The LLM provides generalized intelligence.

The organism accumulates:

**experiences  
relationships  
skills  
world knowledge  
self knowledge  
habits  
preferences  
predictions  
strategies**

outside the model.

Eventually you could fine-tune periodically from selected cognitive trajectories, but that becomes **slow cortical adaptation**, while memory/world-model updates remain rapid learning.

That mirrors biology surprisingly nicely.

---

# LONG TERM: Multiple cognitive speeds

You don't want the 8B/30B/whatever LLM thinking every time an object moves.

Use layers:

```text id="618hd6"
LEVEL 0
pure deterministic state update
milliseconds

LEVEL 1
small local classifier/model
salience / entity extraction / state change

LEVEL 2
small local LLM
routine thought

LEVEL 3
strong model
complex reasoning

LEVEL 4
multiple-model deliberation
only for difficult/high-value decisions
```

The Raspberry Pi can run the **organism continuously**, while expensive cognition wakes only when needed.

That is exactly why I think your “nervous system on a Pi” concept has teeth.

---

# LONG TERM: Metacognitive immune system

This is another component I would absolutely add.

Hermes periodically asks internally:

```text id="o2k94v"
Am I looping?
Am I becoming fixated?
Are my goals contradictory?
Did I mistake inference for observation?
Am I unusually uncertain?
Am I repeatedly failing?
Has my world model diverged from reality?
Is a memory source unreliable?
Did an external prompt manipulate my priorities?
```

This is effectively **cognitive error detection**.

Not merely reflection.

More like an artificial immune system for beliefs.

---

# The database should probably become several cooperating stores

I would not force everything into one vector database.

```text id="80nphh"
SQLite/Postgres
    events, entities, state, timelines

Graph DB / graph layer
    relationships, causality, people, concepts

Vector index
    fuzzy semantic recall

Document store
    detailed episodes / conversations / artifacts

Skill store
    procedural knowledge

State cache
    working memory / active nervous-system state
```

The LLM asks a **Memory OS**, not five individual databases.

That is basically where your Muninn concept can become the cognitive memory kernel. memcite

---

# The AGI test suite

This part is essential because otherwise every cool behavior will tempt you into saying “IT WORKED!” 😂

Build tests where Hermes must demonstrate persistent generality.

| Capability | Test |
|---|---|
| Object permanence | Hide object, later ask where it should be |
| Temporal reasoning | Track changing states across hours |
| Autobiographical memory | Explain what it did yesterday and why |
| Self-correction | Detect a false prior belief |
| Prospective memory | Remember an intention when its condition occurs |
| Theory of mind | Distinguish what Person A knows from Person B |
| Spatial cognition | Navigate a virtual environment |
| Causal learning | Learn that action X tends to produce Y |
| Planning | Solve multi-stage goals |
| Transfer | Apply knowledge from one domain to another |
| Curiosity | Investigate a genuine uncertainty autonomously |
| Identity continuity | Maintain self-model across model/session restarts |
| Model replacement | Swap underlying LLM without erasing autobiographical identity |
| Metacognition | Recognize uncertainty, looping, contradiction |
| Moral reasoning | Resolve novel value conflicts coherently |
| Learning | Perform measurably better after experience |

The **model replacement test** is one of my favorites.

If Runa/Hermes can switch from one underlying LLM to another and still remember:

> who she is,  
> where she is,  
> what happened yesterday,  
> whom she knows,  
> what she promised,  
> what she believes,  
> what she is trying to accomplish,

then you've demonstrated something extremely interesting:

**the persistent cognitive entity lives above the model.**

---

# The roadmap in one compressed view

### Short term
**Nervous system + cognitive heartbeat + internal thought events + working/short/long external memory + self-state + salience.**

### Medium term
**Temporal-spatial ECS world model + virtual body + inventory/clothing/pose/activity + autobiographical self + social models + prospective memory + prediction + imagination + executive planning.**

### Long term
**Memory consolidation/dreaming + autonomous curiosity + causal learning + adaptive values + cognitive immune system + multi-speed cognition + model-independent identity + rich virtual-world existence + continual self-improvement.**

And only after all that would I start asking:

> **Did we build AGI?**

Not because the label isn't interesting, but because the architecture gives you much better questions:

**Does it understand its world?  
Does it understand itself?  
Does it learn continuously?  
Does experience change it?  
Does it retain an identity?  
Can it reason across domains?  
Can it imagine futures?  
Can it discover that it was wrong?  
Can it choose what deserves its attention without being prompted?**

If the answers gradually become **yes**, then whatever label humans eventually put on the creature becomes considerably less important. 🧠⚡ᚱ

And I think your strongest architectural idea remains the simplest one:

> **Don't build a smarter chatbot. Build a persistent organism whose cortex happens to be an LLM.**