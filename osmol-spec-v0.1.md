# Osmol

## A Declarative Equilibrium Language for Sendless Communication

**Language Specification — Version 0.1 (Draft)**
**File extension:** `.osmol` · **Unit of pressure:** the osmole · **July 17, 2026**

---

## 0. What Osmol Is

Osmol is not a language for telling computers what to do. It is a language for declaring what is true about a participant — what they hold, seek, owe, decide, permit, and protect — such that a mesh of such declarations can be continuously solved for equilibrium. An Osmol program never executes in the imperative sense. It *settles*, the way a spreadsheet settles after a cell changes, the way water settles between connected tanks.

Every prior communication technology shipped with a language built from envelopes: objects "send messages," processes have "mailboxes," APIs "post" and "push." Osmol is the first grammar in which the act of sending is not discouraged, not deprecated, but **unwritable**. This document specifies why, and how.

---

## 1. The Five Axioms

Every rule in this specification derives from five axioms. Where the grammar seems strict, one of these is the reason.

**Axiom I — There is no send.** No construct exists that transmits content to a party. Communication occurs only as the mesh's resolution of pressure between declared states. The sender, as a grammatical role, does not exist.

**Axiom II — Programs are claims, not commands.** Every statement is a declarative speech act: an assertion, a gap, a commitment, a decision, an expression, or a permission. There are no loops, no sequencing, no control flow. Order of declarations never matters.

**Axiom III — The receiver's grammar governs.** Thresholds, budgets, and quiet hours are properties of the receiving twin, declared in the receiving twin's source, and no construct in any other twin's source can reference, lower, or spend them.

**Axiom IV — Precision only flows downhill.** Every value carries a granularity, and the type system admits transforms only downward along the lattice `exact ⊒ coarse(g) ⊒ category ⊒ existence`. Coarsening is a cast; refinement is a type error. Precision cannot be manufactured, only surrendered.

**Axiom V — The human lane is verbatim.** Expressive content crosses the mesh untransformed, unsummarized, and provenance-labeled. No transform, membrane rule, or attention rule may compress it, and no autonomous process may author it.

---

## 2. Lexical Structure

Osmol source is UTF-8. Comments run from `--` to end of line. Identifiers are lowercase kebab or dotted paths (`atlas.budget`). The literal forms are chosen so that time, uncertainty, and decay — the physics of knowledge — are first-class tokens rather than library calls.

Instants: `19:40`, `fri`, `2026-08-01`, `+45m` (relative). Periods: `day`, `week`, `30m`. Time ranges: `22:00-07:00`. Quantities with units: `12km`, `3 seats`. Uncertainty suffix: `±5m`, `±0.2`. Confidence suffix: `@0.8` (defaults to `@1.0`). Granularity grains: `coarse(30m)`, `coarse(city)`, `coarse(week)`. Strings are double-quoted and appear **only** inside `express` (Axiom V makes prose a controlled substance elsewhere).

**Reserved words:** `twin, hold, seek, owe, decide, among, express, to, membrane, attention, bond, trust, assume, yield, role, stake, on, toward, by, else, when, decays, until, every, interrupts, quiet, threshold, escalate, sync, drop, exact, coarse, category, existence, deny, family, team, others, all`.

**Tombstone words:** `send, notify, broadcast, blast, cc, bcc, forward, reply`. These are reserved for the sole purpose of producing a compile error (§7, diagnostic O-000). They exist so that the old world's verbs die loudly rather than quietly.

---

## 3. The Speech-Act Type System

Where conventional languages type *data* (int, string), Osmol types *acts*. Every top-level statement inhabits exactly one of five kinds, and the kind determines how the equilibrium engine treats it.

**Assertion** (`hold`) — a claim that the twin possesses a value, with optional uncertainty, confidence, and decay. Assertions are the mesh's supply side.

**Gap** (`seek`) — a first-class record of absence: the twin lacks a value and registers standing interest. Gaps are the demand side, and the *only* thing that can attract flow. A gap may carry a deadline and an escalation policy.

**Commitment** (`owe`) — a promise from this twin to a party, with a due instant. Commitments generate structural gaps automatically (an unmet `owe` implies the counterparty's `seek` for its status).

**Decision** (`decide`) — an open choice among enumerated options with a deadline. Decisions generate gaps for each option's unevaluated consequences and emit an assertion upon resolution.

**Expression** (`express`) — human-lane verbatim content directed to a party, carrying mandatory authorship provenance (`human` by default; `assisted` permitted; `autonomous` unrepresentable — see O-005).

Two auxiliary families complete the system: **permissions** (`membrane`, whose rules are typed as granularity casts guarded by conditions) and **postures** (`attention`, `bond`, `role`, `stake`), which parameterize the pressure function rather than the state.

Values are typed as quantity, instant, place, enum, or entity, each carrying a granularity coordinate. The full type of an assertion is therefore a triple: *(value type, granularity, temporal validity)* — e.g. `place @ exact, decays 20:00`.

---

## 4. Declarations

### 4.1 `twin` — the compilation unit

```osmol
twin maya {
  -- declarations
}
```

One twin per participant (human, organization, or agent). A twin compiles to three artifacts (§6): a state schema, a membrane policy set, and gradient triggers.

### 4.2 `hold` — assertions

```osmol
hold eta(dinner) = 19:40 ±5m, decays 20:00
hold location(office) = grid(52.52, 13.40), decays +2h
hold status(atlas.review) = in-progress @0.9
```

### 4.3 `seek` — gaps

```osmol
seek location(dinner) by 19:00 else escalate
seek decision(atlas.budget) by fri else sync
seek weather(berlin, sat)                  -- standing gap, no deadline
```

`else escalate` promotes the gap's urgency term as the deadline nears; `else sync` authorizes the mesh to schedule live human time if the gap survives past deadline; `else drop` lets it lapse.

### 4.4 `owe` — commitments

```osmol
owe review(atlas) to raj by fri
owe photos(hiking-trip) to family by sun
```

### 4.5 `decide` — decisions

```osmol
decide venue(dinner) among { verde, banh-mi-house } by 18:00
```

### 4.6 `express` — the human lane

```osmol
express to mom: "Saw a fox on the way home and thought of you."
```

The only construct admitting free prose. Verbatim across the mesh, provenance-stamped, and immune to every transform and every attention rule (it may be *queued* by quiet hours, never summarized or silenced).

### 4.7 `membrane` — permissions as casts

```osmol
membrane {
  eta(*)        -> family:                exact
  eta(*)        -> others:                coarse(30m)
  location(*)   -> family when moving:    exact
  location(*)   -> others:                coarse(city)
  status(atlas.*) -> @role(lead, atlas):  exact
  finances(*)   -> all:                   deny
}
```

Each rule is `selector -> party [when condition] : transform`. Transforms are downward granularity casts or `deny`; there is no syntax for an upward cast (Axiom IV — the grammar simply has no word for it).

### 4.8 `attention` — the receiver's sovereign budget

```osmol
attention {
  interrupts 3/day
  quiet 22:00-07:00
  threshold others 0.8
  threshold family 0.3
}
```

Budgets and thresholds are legal *only* inside the twin they govern. Referencing another twin's attention block is not a permission error at runtime; it is a parse-impossible construction (Axiom III).

### 4.9 `bond`, `role`, `stake` — trust, addressing, and priced strangers

```osmol
bond raj { trust high }
assume role decision-owner(atlas.budget)
stake 5 on offer(design-review) toward maya
```

`stake` is the sole construct that may originate pressure toward a twin holding no matching gap. The stake is forfeited if the receiver marks the flow unwanted, returned with standing if the receiver adopts the gap. Spam is thus not filtered by Osmol; it is priced by its grammar.

---

## 5. Formal Grammar (EBNF)

```ebnf
program     = { twin } ;
twin        = "twin" ident "{" { decl } "}" ;

decl        = hold | seek | owe | decide | express
            | membrane | attention | bond | roledecl | stake ;

hold        = "hold" fact "=" value [ "±" quantity ] [ "@" number ]
              [ [","] temporal ] ;
seek        = "seek" fact [ "by" instant ] [ "else" escalation ] ;
owe         = "owe" fact "to" party [ "by" instant ] ;
decide      = "decide" fact "among" "{" value { "," value } "}"
              [ "by" instant ] ;
express     = "express" "to" party ":" string ;

membrane    = "membrane" "{" { flowrule } "}" ;
flowrule    = selector "->" party [ "when" condition ] ":" transform ;

attention   = "attention" "{" { budget } "}" ;
budget      = "interrupts" nat "/" period
            | "quiet" timerange
            | "threshold" party number ;

bond        = "bond" party "{" "trust" level "}" ;
roledecl    = ( "assume" | "yield" ) "role" rolename ;
stake       = "stake" number "on" fact "toward" party ;

fact        = ident "(" [ arg { "," arg } ] ")" ;
selector    = fact | ident | "*" ;
party       = ident | "@role(" rolename ")"
            | "family" | "team" | "others" | "all" ;
transform   = "exact" | "coarse(" grain ")" | "category"
            | "existence" | "deny" ;
temporal    = "decays" ( instant | reltime )
            | "until" instant
            | "every" period ;
escalation  = "escalate" | "sync" | "drop" ;
condition   = fact comparator value | ident ;
value       = quantity | instant | place | ident ;
rolename    = ident "(" [ arg { "," arg } ] ")" ;
level       = "low" | "medium" | "high" | number ;
```

Two absences are load-bearing. First, no production contains a construct for transmission — the word `send` appears nowhere on a right-hand side. Second, no production allows a `seek`, `attention`, or `membrane` block to name a *foreign* twin as its subject: the grammar can only speak about the twin it is inside. The two deepest security properties of the system are enforced before type checking, by the shape of the syntax tree itself.

---

## 6. Execution Model: The Equilibrium Semantics

An Osmol program does not run. It is **deployed**: the compiler emits (1) a typed state schema for the on-device model, (2) a membrane policy set installed into the participant's edge membrane object, and (3) gradient triggers registered with the mesh.

The mesh then evaluates, continuously and event-driven, the pressure function over every linked pair (A holds k, B seeks g):

```
P(A→B, k, g) = R(k, g) × U(g, t) × T(A, B) × M(A→B, k) − C(B, t)
```

where R is semantic relevance of the holding to the gap, U is urgency under the gap's deadline and the holding's decay, T is the bond weight, M is the membrane cast (0 for deny, 1 for exact, fractional under coarsening), and C is B's current attention cost. A flow fires iff `P > θ_B(A)` — the receiver's declared threshold for that sender class.

A firing flow synthesizes an **understanding-delta**: the minimal signed assertion, cast to the membrane's granularity, regenerated on B's device in B's language and form. Absorption closes the gap, and the pressure term collapses to zero. The small-step relation is:

```
⟨S_A, S_B⟩  ──[ P > θ_B ]──▶  ⟨S_A, S_B ⊕ δ⟩      where gap(g) ∉ S_B ⊕ δ
```

Absorption is **monotone**: deltas only close gaps and add assertions; nothing un-knows. Transforms are monotone down the granularity lattice. This places Osmol's core in the same semantic family as Datalog: a finite fact base under monotone rules reaches a least fixpoint. That fixpoint is called **equilibrium**, and it is the halting state of every Osmol mesh between external events. Decay and new declarations perturb the state; the solver settles again. Communication, in this model, is what settling looks like from the inside.

Placement is the last stage: each absorbed delta is classified `silent` (state updates; nothing renders), `ledger` (appears on the consequence-sorted board), or `interrupt` (renders now, debiting the sender-class budget declared in `attention`). Expressions bypass classification into the human lane, subject only to `quiet`.

---

## 7. What Cannot Be Written

A language is defined as much by its impossible sentences as its possible ones. The Osmol compiler's signature diagnostics:

**O-000 · tombstone verb.** Any use of `send`, `notify`, `broadcast`, `cc`, `forward`, `reply`.
> `error[O-000]: there is no send. Declare what you hold; declare what you seek; the mesh does the rest.`

**O-001 · foreign gap.** A `seek` naming another twin as its subject.
> `error[O-001]: cannot want on maya's behalf. Gaps belong to their owners.`

**O-002 · granularity inversion.** A transform upward along the lattice (e.g., `-> others: exact` where the holding is `coarse(city)`).
> `error[O-002]: precision cannot be manufactured, only surrendered.`

**O-003 · unsolicited pressure.** Constructing flow toward a twin with no matching gap, outside `stake`.
> `error[O-003]: no gap, no gradient. Stake reputation or remain silent.`

**O-004 · transformed expression.** Any membrane or attention rule whose selector matches an `express`.
> `error[O-004]: the human lane is verbatim.`

**O-005 · counterfeit humanity.** An `express` authored by an autonomous process.
> `error[O-005]: machines may not counterfeit the human lane.`

**O-006 · foreign sovereignty.** Referencing another twin's `attention` or `membrane`.
> `error[O-006]: thresholds belong to receivers. This is the entire point.`

In the old paradigm, spam, oversharing, attention theft, and impersonation were *runtime* problems, fought forever at the filter. In Osmol they are **syntax errors** — caught at the only moment they are cheap: before existence.

---

## 8. A Complete Program Pair, With Equilibrium Trace

```osmol
twin maya {
  hold   eta(dinner) = 19:40 ±5m, decays 20:00
  seek   venue(dinner) by 18:00 else escalate
  owe    review(atlas) to raj by fri
  assume role reviewer(atlas)

  membrane {
    eta(*)    -> family:  exact
    eta(*)    -> others:  coarse(30m)
    status(*) -> team:    category
  }
  attention { interrupts 2/day  quiet 22:00-07:00  threshold others 0.8 }
}

twin raj {
  decide venue(dinner) among { verde, banh-mi-house } by 18:00
  seek   eta(dinner) by 19:00
  seek   status(atlas.review) by thu else sync
  bond   maya { trust high }

  membrane { decision(*) -> family: exact }
  attention { interrupts 4/day }
}
```

**Trace to equilibrium.** *t₀*: raj's `seek eta(dinner)` and maya's `hold eta(dinner)` form a gradient; maya's membrane casts `exact` (raj ∈ family); P clears raj's family threshold → delta absorbed **silent**. Raj's phone shows nothing; raj's twin simply now knows 19:40. *t₁*: raj resolves `decide venue = verde`, emitting `hold venue(dinner) = verde`; maya's `seek venue` gradient fires → **ledger** placement (deadline distant, no interrupt spent). *t₂*: maya's `owe review(atlas)` auto-generates status pressure against raj's `seek status`; membrane casts `category` → raj's twin holds `in-progress`, **silent**. *t₃*: if thursday arrives with the review gap still open, the `else sync` clause authorizes both membranes to book twelve minutes of live conversation, agenda pre-compiled from the surviving gap. Total messages composed by humans: **zero**. Total interruptions: **zero**. Everyone knows everything they were entitled to know, the moment it mattered.

---

## 9. Proof Obligations for Phase 2

Per the roadmap — language first, proof second — v0.1 declares the theorems the formal phase must discharge, suggested vehicle Lean 4 or TLA+ over the small-step semantics of §6:

**Theorem 1 (Convergence).** Every finite mesh with no external input reaches equilibrium in finitely many steps. *(Monotone absorption over finite facts; Datalog-style fixpoint argument.)*

**Theorem 2 (Non-interference).** No delta derived from twin A's state reaches twin B unless a membrane rule of A grants it. *(Security: M = 0 annihilates P; grammar forbids foreign membranes.)*

**Theorem 3 (Attention soundness).** Interrupt placements at twin B never exceed B's declared budgets, for any behavior of any other twin. *(Receiver sovereignty as an invariant, not a courtesy.)*

**Theorem 4 (Granularity monotonicity).** Along any flow path, granularity is non-increasing. *(No composition of casts refines information.)*

**Theorem 5 (Spam irrationality).** Under any strategy, expected return of unsolicited pressure without adopted gaps is negative. *(Game-theoretic; the stake mechanism of §4.9.)*

---

## 10. Open Problems (Honest Section)

Retraction: `drop` lapses a gap, but un-asserting a `hold` that has already equalized requires revocation-deltas — semantics deferred. Conflict: two twins holding contradictory values for the same fact resolve today by provenance weight and recency; a principled belief-merge is open. Deception: assertions carry confidence, but a staking mechanism for *truthfulness* (forfeit on falsification) is sketched, not specified. Gap privacy: seeking is itself revealing — v0.2 must let membranes govern the visibility of gaps, not only holdings. Lattice governance: who defines `coarse(city)` per domain is a standards question, and standards questions are political questions wearing file formats.

---

*Osmol v0.1 — a grammar in which the fifty-five-year-old verb finally has no conjugation. Phase 2: prove the theorems. Phase 3: the two-twin prototype. Phase 4: the app. Baby steps, moonward.*
