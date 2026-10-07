# Institutional Intelligence for Investing

## Product thesis

The product is not a chatbot over a firm's documents.

It is a **persistent institutional intelligence system** that continuously ingests internal and external information, maintains a living model of the investment universe, and reasons through a computational representation of how the firm itself makes investment decisions.

The system has two core models:

1. A **World Model** — what is true about companies, markets, people, technologies, and evidence.
2. A **Firm Model** — how this specific investment organization interprets those facts and decides what matters.

The core idea is:

```text
WORLD MODEL  +  FIRM MODEL  →  INVESTMENT JUDGMENT
```

**Technical implementation:** Keep the World Model, Firm Model, and institutional memory outside the foundation model in structured stores and learned firm-specific scorers. Put the frontier LLM behind a model gateway so GPT, Claude, Gemini, or future models can be swapped without rebuilding the firm's proprietary intelligence.

---

# 1. The Two-Model System

The first distinction is between understanding the world and understanding **how the firm thinks about the world**.

```text
                  GENERAL INTELLIGENCE
         companies / markets / people / evidence
                           │
                           ▼
                 ┌───────────────────┐
                 │    FIRM MODEL     │
                 │                   │
                 │ Beliefs           │
                 │ Investment rules  │
                 │ Mental models     │
                 │ Risk tolerance    │
                 │ Historical calls  │
                 │ Partner views     │
                 │ Decision patterns │
                 └─────────┬─────────┘
                           │
                           ▼
                  INVESTMENT JUDGMENT
```

The AI should not merely answer:

> Is this a good company?

It should answer:

> Is this a company **we** should invest in, given how this firm believes the world works?

Two firms can look at the exact same underlying facts and come to different conclusions.

For example, one firm may strongly value:

- technological discontinuities,
- proprietary technology,
- founder-market fit,
- rapidly improving cost curves,
- enormous markets,
- structural tailwinds,

while being comfortable underwriting:

- weak early revenue,
- immature go-to-market,
- high technical risk.

Another firm may prioritize:

- proven distribution,
- predictable revenue,
- unit economics,
- capital efficiency,
- market structure.

The **World Model** should be shared and evidence-driven. The **Firm Model** should encode the organization's unique decision function.

**Technical implementation:** The World Model is built from a temporal knowledge graph plus structured entity/fact stores; the Firm Model is composed of explicit rules, retrieval, interpretable decision models, and preference models. Both are assembled into a canonical context object that is passed to a replaceable foundation LLM using constrained structured outputs.

---

# 2. The Layers of the Firm Model

The Firm Model should not be a single giant prompt. It should be composed of several explicit layers.

**Technical implementation:** Represent each layer as its own service or model with a stable interface rather than baking everything into one LLM. A model router composes constitution rules, retrieved precedents, scoring outputs, and partner context into the prompt/context pack, then validates the LLM's response against a schema.

## Layer 1: Investment Constitution

The first layer is the firm's explicit investment philosophy.

```text
FIRM INVESTMENT CONSTITUTION

Core belief:
Technological discontinuities create temporary periods where
small companies can displace incumbents.

We strongly value:
+ Proprietary technology
+ Founder-market fit
+ Rapidly improving cost curves
+ Large or expanding markets
+ Structural tailwinds

We tolerate:
~ Early revenue
~ Immature GTM
~ High technical risk

We dislike:
- Services-heavy businesses
- Growth primarily driven by paid acquisition
- Markets with structurally low gross margins
- Products with weak technical differentiation

Key questions:
1. Why now?
2. Why does this company win?
3. What has to be true for this to become enormous?
4. What evidence could invalidate the thesis?
```

The constitution should then be translated into a more structured investment ontology.

**Technical implementation:** Store the constitution as versioned structured data — e.g. typed criteria, weights, hard constraints, and question templates in Postgres/JSON — and map free-text firm philosophy into this ontology with an LLM-assisted extraction step that requires human approval before becoming active policy.

```text
Company
 │
 ├── Market
 │    ├── potential scale
 │    ├── market growth
 │    ├── structural tailwinds
 │    └── value capture
 │
 ├── Technology
 │    ├── differentiation
 │    ├── defensibility
 │    ├── cost curve
 │    └── technical risk
 │
 ├── Team
 │    ├── founder-market fit
 │    ├── technical ability
 │    ├── recruiting
 │    └── velocity
 │
 ├── Business
 │    ├── adoption
 │    ├── retention
 │    ├── margins
 │    └── distribution
 │
 └── Firm-specific thesis
      ├── Why now?
      ├── Non-consensus insight
      ├── Right to win
      └── Return asymmetry
```

## Layer 2: Historical Decision Model

What the firm **says** it believes may differ from how it actually behaves.

The system should therefore learn from historical investment decisions, passes, debates, and outcomes.

For each historical investment, reconstruct the decision state at the time.

```text
Decision: INVEST

At decision time:
Market                 9/10
Technology             9/10
Team                   8/10
Traction                4/10
Distribution            3/10

Key belief:
Technical superiority will eventually overcome weak GTM.

What happened:
Correct.

Lesson:
Firm historically generates alpha by underwriting
technical advantage before commercial validation.
```

The same process should be applied to passes.

A pass is often just as informative as an investment because it reveals which risks or characteristics are actually disqualifying.

**Technical implementation:** Reconstruct a point-in-time feature snapshot for every historical decision, then train an interpretable classifier such as regularized logistic regression or gradient-boosted trees to estimate `P(INVEST | investment state, firm)`. Preserve the raw historical examples so the learned weights can always be audited against source evidence.

## Layer 3: Preference / Regression Model

Over time, the system should estimate the firm's implicit decision weights.

Conceptually:

```text
Investment decision
       =
 f(
   market quality,
   technical differentiation,
   founder quality,
   traction,
   distribution,
   risk,
   timing,
   firm-specific beliefs,
   partner preferences
  )
```

This can begin as an interpretable scoring system and later become a learned preference model or regression/ranking model trained on actual decisions.

The point is not to blindly predict the firm's behavior. The point is to uncover the **latent decision function** beneath the firm's stated philosophy.

**Technical implementation:** Start with interpretable regression/ranking models over normalized investment features; later add a learned reward or pairwise preference model trained on `(context, preferred analysis, rejected analysis)` examples. Use these models to score and rerank LLM-generated analyses rather than relying on one generation.

## Layer 4: Dynamic Retrieval

The AI should retrieve the historical precedents, IC excerpts, previous debates, and partner commentary most relevant to the current decision.

**Technical implementation:** Use hybrid retrieval: dense embeddings + BM25/lexical search + metadata/temporal filters + knowledge-graph traversal, followed by a cross-encoder or LLM reranker. The final context pack should deliberately include supporting evidence, contradictory evidence, similar past deals, and firm/partner preferences — not simply the nearest document chunks.

```text
                    FIRM MODEL

          ┌─────────────────────────┐
          │ Structured thesis graph │
          └────────────┬────────────┘
                       │
          ┌────────────▼────────────┐
          │ Historical decisions    │
          │ + outcomes              │
          └────────────┬────────────┘
                       │
          ┌────────────▼────────────┐
          │ Relevant IC excerpts    │
          │ retrieved dynamically   │
          └────────────┬────────────┘
                       │
          ┌────────────▼────────────┐
          │ Preference / scoring    │
          │ model                   │
          └────────────┬────────────┘
                       │
                       ▼
                    LLM Agent
                       │
                       ▼
                   Firm Critic
```

## Layer 5: Firm Critic

The AI should not merely be prompted to "think like the firm." A second process should evaluate whether the analysis actually reflects the firm's worldview.

```text
Research Agent
      │
      ▼
Investor Agent
      │
      ▼
Firm Critic
      │
      ├── Did it apply our key investment principles?
      ├── Did it ignore relevant historical precedent?
      ├── Is the recommendation consistent with prior decisions?
      ├── Is it overweighting something we historically do not care about?
      └── Is it making an assumption contrary to our worldview?
      │
      ▼
Final Investment Analysis
```

A critic could produce feedback such as:

**Technical implementation:** Run a second-pass critic model against a fixed rubric and the outputs of the Firm Model. It can be the same underlying LLM with a different role initially, then later be supplemented by a smaller reward model that scores firm alignment, evidence grounding, and historical consistency.

```text
WORLDVIEW VIOLATION

The analysis heavily penalizes the company for weak near-term
revenue predictability.

This conflicts with the firm's historical willingness to tolerate
commercial uncertainty when technical differentiation is high.

Relevant precedents:
Investment #23
Investment #41
Investment #87

Re-evaluate the company weighting technology > predictability.
```

---

# 3. Disagreement Between Firm Members + the Learning Loop

A real investment firm does not have one monolithic worldview.

Partners may share a core philosophy while weighting evidence differently.

The model should therefore represent both:

- shared firm beliefs,
- partner-specific decision functions.

```text
             FIRM MODEL

              Core beliefs
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Partner A Partner B Partner C
       worldview worldview worldview
```

This enables the system to say things like:

> This company is strongly aligned with the firm's overall thesis. Sarah is likely to focus on distribution risk, while David has historically been willing to tolerate that risk when technical differentiation is unusually high.

Eventually, the system should be able to simulate:

> How is the investment committee likely to react to this deal?

Not by roleplaying personalities, but by modeling decision patterns.

**Technical implementation:** Use a hierarchical or multi-task model in which each partner inherits a shared firm-level baseline and learns only partner-specific deviations. A Bayesian hierarchical regression is attractive early because it handles sparse per-partner data via partial pooling instead of requiring a fully separate model for each person.

## The Learning Loop

The most important source of improvement is disagreement between the AI and the humans.

```text
      Investment thesis
             │
             ▼
      AI recommendation
             │
             ▼
       Human decision
             │
             ▼
      WHY did humans
      disagree with AI?
             │
             ▼
      Firm Model update
             │
             ▼
        Future deals
```

Suppose the AI recommends pursuing a company and a partner responds:

> No. We've seen this exact structure before. These marketplaces get commoditized.

That correction should not disappear into a chat transcript.

The system should extract a candidate belief:

```text
New firm belief candidate:

"Vertical marketplaces with low switching costs tend toward
commoditization despite early network effects."

Evidence:
Partner feedback
Historical deals X, Y, Z

Status:
Proposed belief

Confidence:
0.71
```

The firm's institutional knowledge should therefore **evolve explicitly** as new decisions are made.

**Technical implementation:** Log every AI recommendation, human edit, override reason, final decision, and later outcome as a labeled training event. Update state in real time, but retrain preference/decision models offline behind an evaluation gate so one anomalous correction cannot silently change production behavior.

---

# 4. The Data We Need

The core training unit is not a document.

It is a decision trajectory:

```text
situation → reasoning → partner correction → eventual decision
```

Ideally, each historical example includes:

- the facts available at the time,
- the initial investment thesis,
- the key uncertainties,
- partner reactions,
- disagreements,
- corrections to the initial analysis,
- the final decision,
- the eventual outcome.

Important sources include:

```text
IC memos
Partner meeting transcripts
Investment notes
Passed opportunities
Winning investments
Losing investments
Partner comments
Investment committee votes
Postmortems
Fund strategy documents
```

Over time this becomes a preference dataset:

```text
Investment situation → Firm decision → Partner reasoning → Outcome
```

This dataset is much more valuable than a corpus of memos because it captures **how the firm transforms information into decisions**.

**Technical implementation:** Store each trajectory as a versioned training example containing the point-in-time world state, retrieved evidence, model output, partner correction, final decision, and eventual outcome. This becomes the source for supervised decision models, pairwise preference training, calibration, and an internal eval suite.

---

# 5. Product Maturity Path

The system can mature in stages.

**Phase 1:** Explicit philosophy + RAG  
**Phase 2:** Structured historical decision model  
**Phase 3:** Learned firm preferences  
**Phase 4:** Personalized partner models  
**Phase 5:** Continuous learning from decisions and outcomes

This progression creates a compelling response to the obvious objection:

> Couldn't I just put all our memos into ChatGPT?

**No.**

Because we are not building a search interface over documents.

We are building a **computational model of how the firm makes investment decisions**, and forcing every AI workflow to reason through that model.

That model can become the deepest proprietary layer of the product.

**Technical implementation:** Keep the durable IP in structured state, decision datasets, preference models, partner-specific models, and evals — not only in fine-tuned LLM weights. Fine-tuning can be introduced later with SFT/DPO or LoRA-style adapters, but should remain an optimization layer so the core system stays portable across foundation-model providers.

---

# 6. What Happens When an Email Arrives

The system should behave like a persistent institutional process, continuously observing the organization's information environment.

Consider a founder email:

> Great speaking yesterday. As discussed, we're currently at $14M ARR and expect to finish the year around $18M. Happy to send the latest materials.

The system should not simply embed the email and add it to a vector database.

It should process it as an event that may alter the state of the firm's investment universe.

```text
NEW EMAIL
    │
    ▼
Parse + classify
    │
    ├── People: Sarah Chen
    ├── Company: Acme Robotics
    ├── Topic: fundraising / company update
    └── Access: partner-specific
    │
    ▼
Extract candidate facts
    │
    ├── ARR = $14M
    ├── Year-end ARR forecast = $18M
    └── Founder offered latest materials
    │
    ▼
Compare against existing state
    │
    ├── Previous ARR estimate = $11M
    ├── Firm currently evaluating company
    └── Revenue growth is key diligence question
    │
    ▼
Update Acme state
    │
    └── ARR: $11M → $14M
    │
    ▼
Evaluate significance
    │
    └── HIGH — directly affects active investment thesis
    │
    ▼
Trigger workflows
         ├── update company page
         ├── update financial model assumption
         ├── flag IC memo
         └── surface to deal team
```

**Technical implementation:** Consume email via provider webhooks/history APIs into an event bus, then run a staged pipeline: cheap classifier → entity extraction/resolution → structured claim extraction → contradiction/change detection → significance scoring. Only high-value events escalate to a frontier LLM; low-value events are handled by smaller models or deterministic rules.

The governing question for every incoming piece of information is:

> **What does this change about what the firm currently believes?**

That is more important than simply asking where the document should be stored.

---

# 7. How the System Stores Knowledge

The persistent object should not be the chat history.

The persistent object should be the **investment universe itself**.

The system should maintain long-lived objects such as:

**Technical implementation:** Use a temporal/event-sourced data model: relational storage for canonical entities and permissions, a graph layer for relationships, a vector index for semantic retrieval, and immutable source/event records for provenance. Store facts as typed claims with subject/predicate/value/time/source/confidence rather than only prose chunks.

```text
COMPANIES
Acme Robotics
Anthropic
SpaceX
...

PEOPLE
Founder A
Executive B
Investor C
...

MARKETS
Foundation models
Grid infrastructure
Defense autonomy
...

INVESTMENT THESES
AI inference costs collapse
Power becomes AI bottleneck
Autonomous systems reshape defense
...

DEALS
Acme Series C
XYZ Seed
...

FIRM BELIEFS
Technical superiority can compensate
for weak early GTM
...
```

Knowledge should also be stored hierarchically rather than as undifferentiated chunks.

```text
Raw Data
   ↓
Entities
   ↓
Facts
   ↓
Claims
   ↓
Evidence
   ↓
Investment Questions
   ↓
Theses / Risks
   ↓
Conviction
   ↓
Decision
```

For example, a raw observation:

> Customer said implementation took six months.

could become:

```text
Fact:
Customer A deployment took 6 months.

Supports:
"Acme has unusually high implementation complexity."

Impacts:
Gross-margin thesis
Sales-cycle thesis
Scalability thesis

Evidence:
Customer call – Sept 19
Salesforce note – March 3
G2 reviews – 14 similar references

Confidence:
High
```

## Permissions and Provenance

Every derived fact needs provenance and access controls.

```text
FACT

CEO replacement being considered

source:
email_928391

visible_to:
Partner A
Partner B

derived_from:
email_928391

confidence:
0.81
```

The system should propagate permissions from source data into derived knowledge.

A confidential email should never accidentally become a globally visible firm-level fact.

**Technical implementation:** Attach source ACLs and provenance IDs to every extracted fact and derived claim, then compute effective visibility as the intersection/union policy defined by the firm. Permission checks must happen both at retrieval time and before derived outputs are persisted or surfaced.

---

# 8. UX: The Intelligence Terminal

The application should not open to an empty chat box.

It should open to an **intelligence terminal** that answers:

> What changed that I should know about?

One possible home screen is a persistent **Pulse** view.

**Technical implementation:** Build the front end around read models derived from the event/state store rather than calling an LLM on every page load. Pulse is powered by a continuously updated ranked feed; chat invokes the same state/retrieval APIs as the rest of the product instead of maintaining a separate memory silo.

```text
┌───────────────────────────────────────────────────────────────┐
│ GOOD MORNING                                      Tue Oct 6   │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  7 meaningful changes since yesterday                        │
│                                                               │
│  🔴 ACME ROBOTICS                                             │
│  Revenue appears ~27% higher than our previous estimate.      │
│  Source: founder email                                        │
│                                                 [Open →]      │
│                                                               │
│  🟡 AI INFRASTRUCTURE                                         │
│  Two portfolio companies independently reported sharply       │
│  higher demand for inference capacity.                        │
│                                                 [Explore →]   │
│                                                               │
│  🟡 PERSON                                                    │
│  Former VP Engineering at Company X appears to have joined    │
│  one of the companies we're currently diligencing.            │
│                                                 [Open →]      │
│                                                               │
├───────────────────────────────────────────────────────────────┤
│ WORKING IN BACKGROUND                                         │
│                                                               │
│ ● Monitoring 34 active companies                              │
│ ● Updating AI infrastructure landscape                        │
│ ● Preparing Thursday IC                                       │
│ ● Tracking 9 unanswered diligence questions                   │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

The product should feel like a persistent employee or institutional process that is always working in the background.

## Living Company Page

Clicking a company should reveal a live object rather than generate a temporary report.

```text
ACME ROBOTICS

STATUS
Series C diligence

OUR CURRENT VIEW
Promising technical architecture with unusually strong
enterprise pull. Main unresolved risk remains deployment cost.

                   CONVICTION
                  7.2 → 7.7 ↑

WHAT CHANGED
────────────────────────────────────────────

Today
ARR confirmed at $14M
+ strengthens growth thesis

Yesterday
Customer reference cited 6-month deployment
- increases implementation-risk concern

Oct 2
VP Engineering left competitor
potential recruiting opportunity


KEY BELIEFS
────────────────────────────────────────────

✓ Market could support $10B+ outcome
  confidence: HIGH

✓ Technical architecture is differentiated
  confidence: MEDIUM

? Gross margins can exceed 60%
  confidence: LOW

✕ Deployment can scale without services
  confidence: LOW


OPEN QUESTIONS

→ Why are deployments taking 4–6 months?
→ What % of revenue requires implementation support?
→ What is true software gross margin?
```

Chat should exist, but as one interface into persistent state rather than the state itself.

**Technical implementation:** Materialize company pages from canonical entity state plus cached belief/conviction summaries, and update them incrementally when new events land. Long-form summaries can be regenerated asynchronously by the LLM when the underlying state changes beyond a defined threshold.

## Agent Activity View

Because the AI is always running, users need to see what it is doing.

```text
AI ACTIVITY

4:31 PM
Processed 143 new information events

4:28 PM
Updated Acme Robotics revenue estimate
$11M → $14M
[source]

4:22 PM
Associated meeting transcript with:
Acme Robotics
Sarah Chen
Series C diligence

4:05 PM
Started monitoring newly identified competitor:
Vector Dynamics

3:42 PM
Found contradictory information regarding
Acme's estimated customer count.

Investigating...
```

This makes the system feel transparent and persistent rather than magical and opaque.

**Technical implementation:** Maintain an append-only agent/workflow ledger containing each action, source inputs, model/version used, confidence, and resulting state mutation. Expose that ledger in the UI so users can trace every proactive update back to its triggering event.

---

# 9. Ingestion Should Be Continuous, Not Batch

Every data source should be treated as an event stream.

**Technical implementation:** Normalize provider-specific webhooks, polling deltas, and external feeds into a common event schema on Kafka/Pub/Sub/SQS-like infrastructure. Make processors idempotent and checkpointed so the system can replay history, recover from failures, and rebuild state deterministically.

```text
                    EVENT BUS

Email ───────────────┐
Slack ───────────────┤
Calendar ────────────┤
Meeting transcripts ─┤
CRM ─────────────────┤
Drive ───────────────┤
Portfolio metrics ───┤
Pitch decks ─────────┤
News ────────────────┤
SEC filings ─────────┤
Job postings ────────┤
Websites ────────────┤
Market data ─────────┘
                     │
                     ▼
              INFORMATION ENGINE
                     │
              "What is this?"
                     │
                     ▼
                ENTITY GRAPH
                     │
              "What changed?"
                     │
                     ▼
                 FIRM STATE
                     │
           "Does anyone care?"
                     │
           ┌─────────┴─────────┐
           ▼                   ▼
       SILENT UPDATE       TAKE ACTION
                               │
                        ┌──────┼──────┐
                        ▼      ▼      ▼
                       alert  agent  workflow
```

The important part is the final question:

> **Does anyone care?**

A persistent intelligence system may process thousands of events per day. Most of them should not generate notifications.

The system therefore needs an **attention model**.

Conceptually:

```text
Importance =
    relevance to active investments
  × magnitude of change
  × confidence
  × novelty
  × urgency
```

For example:

```text
TechCrunch posts another generic AI article

Relevance       .2
Novelty         .1
Importance      → IGNORE
```

versus:

```text
Founder of a company you're diligencing tells a partner
revenue is 30% above your model

Relevance       1.0
Novelty          .9
Magnitude        .9
Confidence       .9
Importance      → SURFACE IMMEDIATELY
```

This attention layer is what allows the system to be proactive without becoming noisy.

**Technical implementation:** Begin with a rules + gradient-boosted-tree ranking model using features such as entity relevance, novelty, contradiction score, magnitude, source reliability, deal stage, and urgency. Over time, train it on implicit labels such as opened/dismissed/shared/acted-on to personalize attention ranking per investor.

---

# 10. The Architectural Shift

The difference between a traditional enterprise AI system and this product is the difference between **retrieval** and **persistent institutional state**.

```text
Traditional enterprise AI

Documents
   ↓
Vector database
   ↓
Retrieve
   ↓
LLM
   ↓
Answer
```

versus:

```text
Your system

Events
   ↓
Interpret
   ↓
Entity Resolution
   ↓
Institutional State
   ↓
Belief Updates
   ↓
Active Workflows
   ↓
Investor Attention
```

The right-hand system is effectively building a **digital twin of the investment organization**.

**Technical implementation:** Separate the system into replaceable layers: ingestion/event processing, persistent institutional state, retrieval, firm-specific ML, and a model gateway for foundation LLMs. Define strict typed interfaces between layers so the underlying LLM can be benchmarked and swapped without migrating the firm's memory or decision models.

---

# 11. Bottom Line

The product can be summarized as:

```text
persistent world state
        +
firm-specific judgment
        +
continuous ingestion
        +
learning from decisions
        =
institutional intelligence
```

The durable advantage is not the chat interface, the retrieval layer, or even the connectors.

It is the **Firm Model**:

> A computational representation of how the partnership interprets evidence, weighs tradeoffs, disagrees, learns, and ultimately makes investment decisions.

As the system observes more decisions, corrections, outcomes, and partner disagreements, that model improves.

The product therefore does not merely retrieve institutional knowledge.

It **learns the institution itself**.

**Technical implementation:** The production loop is `event → state update → retrieval → candidate reasoning → firm scoring/critic → human action`; the training loop is separate and offline: `corrections + outcomes → training set → retrain → eval suite → deploy if better`. Fine-tuning is optional and model-family-specific; the enduring institutional model remains external, auditable, and portable.
