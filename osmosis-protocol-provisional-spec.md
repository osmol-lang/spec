# PROVISIONAL SPECIFICATION (Concept Draft)

## SYSTEM AND METHOD FOR SENDLESS COMMUNICATION VIA GRADIENT-DRIVEN EQUALIZATION OF UNDERSTANDING BETWEEN PERMISSIONED COGNITIVE STATE MODELS

**Working name:** The Osmosis Protocol
**Applicant / Inventor:** [You]
**Drafted:** July 17, 2026
**Status:** Concept specification in the style of a provisional patent application. This is a design document, not a legal filing, and not legal advice.

---

## ABSTRACT

A communication system in which no messages are sent. Each user is represented by a cognitive state model ("twin") hosted primarily on the user's mobile device, comprising machine-readable knowledge assertions, unresolved informational gaps, commitments, and pending decisions, enclosed within a cryptographically enforced permission membrane. A distributed mesh continuously computes knowledge-pressure gradients between permissioned twins — conditions in which one model holds information that another model needs and is entitled to receive. When a gradient exceeds a receiver-defined threshold, the system synthesizes an understanding-delta: a minimal semantic payload, regenerated at the receiving side in the receiver's own language, framing, and device context. The receiving model absorbs the delta, the gradient collapses, and equilibrium is restored. Addressing is semantic (by role or by live state) rather than by endpoint. Human attention is treated as a metered resource: most deltas equalize silently, and interruption requires clearing a receiver-set budget. Legacy electronic mail is ingested through an adapter and processed as inbound pressure, rendering the composition and transmission of messages obsolete for any party inside the mesh.

---

## FIELD OF THE INVENTION

The invention relates to digital communication systems, and more particularly to methods of transferring understanding between parties without the composition, addressing, transmission, or receipt of user-composed messages.

---

## BACKGROUND OF THE INVENTION

Electronic mail, standardized in the 1970s and formalized in SMTP (RFC 821 and successors), established a paradigm that all subsequent communication systems have inherited: a sender composes an unstructured payload, addresses it to a static endpoint identifier, and pushes it into a receiver-side queue ordered by arrival time. This paradigm exhibits at least four deficiencies that no successor has cured.

First, inverted economics: the cost of initiating communication is borne almost entirely by the receiver, whose attention is consumed parsing payloads to discover their intent. Second, semantic opacity: payloads are opaque to machines, so no component of the system can act on, deduplicate, or resolve them without human labor. Third, endpoint addressing: messages are routed to mailbox identifiers rather than to the party who actually holds a given role or decision, producing forwarding chains, misdirected requests, and staleness. Fourth, transmission as the atomic act: the systems measure success by delivery, whereas the value of communication lies in a change of state — a decision made, a commitment recorded, a gap in one party's knowledge closed.

Prior art has addressed fragments of the problem without curing it. Instant messaging and channel-based systems (e.g., XMPP-family and proprietary workplace platforms) increase frequency while preserving all four deficiencies. Publish-subscribe architectures allow interest registration but transport raw payloads to topic endpoints and carry no model of the subscriber's understanding. File synchronization and CRDT-based replication equalize replicas of an identical artifact between devices of the same logical owner; they do not equalize understanding between heterogeneous, private, differently-permissioned models belonging to different parties. Speech-act messaging systems of the 1980s (e.g., The Coordinator, after Winograd and Flores) typed messages by intent but required humans to perform the structuring, and failed for that reason. Predictive assistant systems surface information to their own user but provide no inter-party protocol, no permission calculus, and no economics of attention.

There exists, therefore, a need for a communication system in which understanding flows between parties as a function of need, entitlement, and priority — without any party composing or sending a message.

---

## SUMMARY OF THE INVENTION

The invention replaces the message with the gradient. Each participant maintains a cognitive state model comprising, at minimum: (a) knowledge assertions the participant holds; (b) informational gaps — values the participant lacks and has standing or predicted interest in; (c) commitments given and received; and (d) decisions pending or resolved. The model is wrapped in a permission membrane: a policy layer, authored and owned by the participant, that determines what may flow outward, to whom, at what granularity, and under what transformation.

A mesh layer instantiates one always-on, globally addressable, stateful membrane object per participant on distributed edge compute. Between any pair of linked membranes, a gradient engine continuously evaluates a knowledge-pressure function over holdings and gaps. Where pressure exceeds the receiving party's threshold and the membrane grants passage, a synthesis engine generates an understanding-delta — not the source text, but the minimal semantic content sufficient to close the receiving model's gap — signed, provenance-labeled, and regenerated at render time in the receiver's language, reading depth, and device context. An attention classifier then assigns the delta to one of three placements: silent state update, ledger entry, or budgeted interruption. Absorption of the delta collapses the gradient.

Addressing is performed semantically: flows are directed to roles ("the owner of decision D"), to live states ("the current truth of project P"), or to registered gaps, and the mesh resolves the current binding at flow time. Unsolicited pressure — flow initiated toward a party that has registered no corresponding gap — is not computed unless the initiator stakes transferable reputation, making bulk unsolicited contact economically irrational rather than merely filtered. Human-expressive content is exempt from regeneration, transferred verbatim under a protected flag, and routed to a reserved human lane. A legacy adapter ingests SMTP mail, decomposes it into candidate assertions and gaps, and submits it to the same pipeline, permitting single-party adoption with immediate utility.

The result is a network in which, for parties inside the mesh, composing an email is equivalent to hand-carrying water between two tanks already joined by a pipe.

---

## BRIEF DESCRIPTION OF THE DRAWINGS

**FIG. 1** depicts two cognitive state models joined by permission membranes, with a knowledge-pressure gradient forming between a holding in the first model and a registered gap in the second, and an understanding-delta crossing the membranes.

**FIG. 2** depicts the preferred hardware embodiment: a mobile device hosting the private model, on-device inference, and secure-enclave keys; and an edge mesh hosting one stateful membrane object per participant, a semantic role registry, a vector index, and delta queues.

**FIG. 3** is a flowchart of the osmosis cycle: state ingestion; gap registration; gradient computation; membrane evaluation; delta synthesis; placement classification; absorption and gradient collapse.

**FIG. 4** depicts semantic address resolution, in which a flow directed to a role identifier is bound at flow time to the membrane of the participant currently attested to hold that role.

**FIG. 5** depicts the legacy adapter, in which inbound SMTP mail is parsed into assertions, requests, and gaps, and enters the pipeline of FIG. 3 as external pressure.

---

## DETAILED DESCRIPTION OF PREFERRED EMBODIMENTS

### 1. Definitions

"Twin" or "cognitive state model" means a continuously updated, typed, machine-readable representation of a participant's communicative state, comprising at least assertions, gaps, commitments, and decisions, each carrying timestamps, confidence values, and provenance. "Membrane" means the participant-owned policy layer governing outbound and inbound flow. "Gap" means a first-class record that the model lacks a value and holds standing, explicit, or predicted interest in obtaining it. "Understanding-delta" or "delta" means the minimal signed semantic payload sufficient to close a specified gap. "Pressure" means the scalar output of the gradient function defined in Section 3. "Mesh" means the distributed layer hosting membrane objects and registries.

### 2. State Ingestion and Gap Registration

The twin ingests from sources the participant authorizes — calendar, documents, task systems, transcripts, sensors — and normalizes them into the typed state. Gaps arise three ways. Explicit: the participant asks a question, which is recorded as a gap rather than transmitted as a message. Structural: an open decision or commitment implies missing inputs (a decision with three options implies a gap for each unevaluated consequence). Predictive: an on-device model infers, from the participant's trajectory, that a value will be needed before a time T (a traveler will need the changed meeting location before departure), and pre-registers the gap. Predictive gaps are marked as such and weighted conservatively.

### 3. The Gradient Engine

For each linked pair of participants A and B, and for each holding k in A's model, the engine evaluates a pressure function of the general form:

P(A→B, k) = R(k, G_B) × U(k, t) × T(A, B) × M(A→B, k) − C(B, t)

wherein R is the semantic relevance of holding k to B's registered gaps G_B, computed by embedding similarity over the vector index; U is urgency, a function of deadlines attached to the gap and the decay rate of k's value; T is the trust weight of the A–B link; M is the membrane passage score, equal to zero where policy denies, one where policy grants, and fractional where policy grants under transformation; and C is the current attention cost at B, a function of B's interruption budget, focus state, and local time. Flow is triggered if and only if P exceeds a threshold θ_B set by the receiving participant. The threshold is the receiver's property; no sender may lower it. In one embodiment the engine runs event-driven upon any state change at either endpoint, with a low-frequency periodic sweep for predictive gaps.

### 4. The Membrane

The membrane evaluates each candidate flow against participant-authored rules and learned preferences, returning allow, deny, or transform. Transformations include granularity reduction (precise location becomes city-level for non-family links), temporal delay (financial figures flow only after an embargo), aggregation (individual datapoints flow only as statistics), and redaction. All membrane decisions are logged in an append-only record visible to the membrane's owner. The membrane is portable: its policy set and logs export in a standard format, such that the participant may rehost it on any conforming mesh provider without loss.

### 5. Delta Synthesis and Receiver-Side Regeneration

Upon a granted flow, the synthesis engine produces the delta as a signed semantic assertion — subject, predicate, value, validity window, provenance chain — rather than as source prose. The source text, where any exists, never crosses the membrane. At the receiving side, a renderer regenerates the delta into whatever form the receiver's context demands: eight words on a watch, a sentence in the receiver's native language on a phone, a paragraph of briefing at a desk, or synthesized audio in a vehicle. Regeneration is performed by on-device inference where the device is capable, and by mesh-side inference under the receiver's keys where it is not. The same delta therefore has no canonical rendered form; form is the receiver's property.

### 6. Placement and the Attention Budget

An attention classifier assigns each absorbed delta to one of three placements. Silent: the state updates and no human rendering occurs (the majority case; the participant simply possesses current knowledge on next reference). Ledger: the delta surfaces in the participant's consequence-sorted board of commitments, decisions, and open questions, without interruption. Interruption: the delta is pushed to the participant's attention immediately, which debits the sender's standing against the receiver's interruption budget. Budgets are denominated per link and per period; chronic overdraft raises the effective θ_B for that sender. Interruption placement additionally requires the delta to carry a declared consequence-if-ignored field, which the classifier verifies for plausibility.

### 7. Semantic Addressing

The mesh maintains a role registry binding attested roles ("decision-owner: Atlas budget", "on-call: payments", "current truth: Project P schedule") to membrane objects, with bindings signed by the parties entitled to confer them. A flow directed to a role resolves at flow time to the currently bound membrane, so that reorganizations, handoffs, and delegation require no notification to any counterparty: the pipe simply rebinds. Queries to live states resolve to a read against the owning twin's exported state slice, subject to the membrane, rather than to any stored document.

### 8. Anti-Spam Economics

A flow initiated toward a participant who has registered no corresponding gap is termed unsolicited pressure. The gradient engine does not evaluate unsolicited pressure unless the initiator attaches a reputation stake, which is forfeited to the receiver if the receiver's membrane or the receiver marks the flow unwanted, and returned with interest to the initiator if the receiver adopts the resulting gap. Because bulk unsolicited contact under this regime carries linear cost and superlinear reputational risk, the spam strategy is not filtered but priced out of rationality. This inverts the economics of the 1971 paradigm at the protocol layer rather than at the client layer.

### 9. The Human Lane

Content flagged by its author as human-expressive — affection, condolence, humor, art, conflict, anything whose value lies in its exact form — is exempt from synthesis and regeneration, crosses the membrane verbatim, carries a mandatory authorship label (human-authored, machine-assisted, or machine-autonomous), and is routed to a reserved lane that the attention classifier may not silence. In the preferred embodiment the human lane is deliberately inefficient: no summarization, no batching, full fidelity. The system's design objective is that traffic in every other lane falls while traffic in this lane rises in warmth per unit.

### 10. Escalation to Synchrony

Where a gradient persists unresolved beyond a latency budget — the parties' models detect that accumulated asynchronous cost exceeds the projected cost of live conversation — the membranes jointly schedule a bounded synchronous session, with the agenda compiled from the open gaps that caused escalation. Synchronous human interaction thereby becomes the system's exception handler rather than its default ritual.

### 11. Legacy Adapter

An SMTP gateway receives conventional electronic mail addressed to the participant, parses each message into candidate assertions, requests, commitments, and gaps, assigns provisional provenance (unverified-legacy), and submits the result to the pipeline of FIG. 3. Outbound, the adapter can serialize a delta into a well-formed email for counterparties outside the mesh. The adapter yields single-participant utility on day one — a participant's existing inbox is equalized into their twin — and functions as the adoption vector by which the mesh absorbs, and eventually retires, the legacy protocol.

### 12. Preferred Hardware Embodiment

In the preferred embodiment the twin's private model and inference run on the participant's mobile device, using the device's neural processing unit for quantized on-device models, the device's secure enclave for the participant's signing keys, and encrypted local storage for the state. The membrane object is instantiated as an isolate-based, globally addressable, stateful compute object on a distributed edge platform, one per participant, holding only the membrane policy, the outbound-permitted state slice, queued deltas for offline devices, and the append-only logs. The semantic registry is implemented over a distributed vector index. All mesh-side computation on plaintext state occurs under the participant's keys; in a further embodiment, membrane evaluation is performed within attested trusted execution environments, such that the mesh operator is technically incapable of reading state slices. The device being the twin's primary body, loss of connectivity degrades the system to queued equalization rather than failure: deltas accumulate at the membrane and absorb on reconnection.

### 13. Security, Provenance, and Governance

Every delta is signed over its content, provenance chain, and authorship class. Unverifiable inbound content is unrenderable by default. Membranes, models, and logs are the participant's exportable property; the protocol conformance suite requires portability as a condition of interoperation, in order that no mesh operator may convert the filter layer into a captive position.

---

## CLAIMS

What is claimed is:

**1.** A computer-implemented method of communicating between a first participant and a second participant without transmission of a user-composed message, the method comprising: maintaining, for each participant, a cognitive state model comprising knowledge assertions, informational gaps, commitments, and decisions; establishing a permissioned link between the participants' models via participant-owned policy membranes; continuously computing a knowledge-pressure value between a holding of the first model and a gap of the second model as a function of at least semantic relevance, urgency, link trust, membrane passage, and receiver attention cost; responsive to the pressure value exceeding a threshold set by the second participant, synthesizing a signed understanding-delta comprising a minimal semantic assertion sufficient to close said gap; regenerating the delta at the second participant's device in a form determined by the second participant's language, context, and device; and absorbing the delta into the second model such that the pressure value collapses.

**2.** The method of claim 1, wherein the cognitive state model of each participant is hosted on a mobile device of that participant, and wherein synthesis or regeneration is performed by on-device machine-learning inference.

**3.** The method of claim 1, wherein at least one gap is registered predictively by an on-device model in advance of any explicit request by the second participant.

**4.** The method of claim 1, wherein the membrane returns a transformation directive, and the delta is synthesized at reduced granularity, delayed validity, or aggregated form in accordance with said directive.

**5.** The method of claim 1, wherein each delta carries a cryptographic signature and an authorship class selected from human-authored, machine-assisted, and machine-autonomous, and wherein content lacking a verifiable signature is not rendered by default.

**6.** The method of claim 1, wherein the delta is directed to a semantic role identifier, and the mesh resolves said identifier to the membrane of the participant currently attested to hold said role at flow time.

**7.** The method of claim 1, further comprising classifying each absorbed delta into one of a silent state update, a ledger entry, and an interruption, wherein interruptions are debited against a receiver-defined budget per link and per period.

**8.** The method of claim 1, wherein a flow initiated toward a participant having no corresponding registered gap is evaluated only upon attachment of a forfeitable reputation stake by the initiator.

**9.** The method of claim 1, wherein content flagged human-expressive is transferred verbatim, exempt from synthesis and regeneration, and routed to a lane that the placement classifier is prohibited from silencing.

**10.** The method of claim 1, further comprising ingesting a legacy electronic mail message via an SMTP adapter, decomposing said message into assertions, requests, and gaps, and processing the result as inbound pressure under claim 1.

**11.** The method of claim 1, further comprising detecting that a gradient has persisted beyond a latency budget, and automatically scheduling a bounded synchronous session between the participants with an agenda compiled from the unresolved gaps.

**12.** A sendless communication system comprising: a plurality of edge devices, each hosting a participant's cognitive state model, on-device inference, and signing keys held in secure hardware; a distributed mesh instantiating one globally addressable stateful membrane object per participant; a gradient engine configured to compute knowledge-pressure values between linked models; a synthesis engine configured to generate signed understanding-deltas; a semantic registry binding attested roles to membrane objects; and delta queues for offline devices.

**13.** The system of claim 12, wherein each membrane object is an isolate-based stateful compute object executing at a network edge location proximate to its participant.

**14.** The system of claim 12, wherein the semantic registry comprises a distributed vector index over gap and holding embeddings, and role bindings signed by parties entitled to confer said roles.

**15.** The system of claim 12, wherein membrane evaluation executes within an attested trusted execution environment such that a mesh operator cannot read participant state.

**16.** The system of claim 12, wherein membrane policies, state models, and decision logs are exportable in a standard format and rehostable on a conforming alternative mesh without loss.

**17.** The method of claim 7, wherein an interruption-class delta is required to carry a declared consequence-if-ignored field verified for plausibility by the placement classifier.

**18.** A non-transitory computer-readable medium storing instructions that, when executed by one or more processors, cause the one or more processors to perform the method of claim 1.

---

## INDUSTRIAL APPLICABILITY

The invention applies to personal, enterprise, governmental, and machine-to-machine communication, and in particular to the replacement of electronic mail, workplace messaging, notification systems, and status reporting with permissioned equalization of state. For parties operating inside the mesh, the composition and transmission of a message becomes a legacy gesture — the hand-carried bucket beside the pipe.

---

*Note: this document adopts the structure and language of a provisional patent specification for design purposes. An actual filing requires a qualified patent attorney, a professional prior-art search, and jurisdiction-specific drafting.*
