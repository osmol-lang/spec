# Everything the Language Needs

## The Osmol Engineering Dissertation — v0.1

*From grammar to governance: the complete inventory of what must exist for Osmol to live on any machine, any edge, and any phone. July 2026.*

---

### Abstract

A language is not a grammar. A language is a grammar plus a compiler, a runtime, a wire format, a cryptographic identity layer, an installer, an editor experience, a package presence, a conformance suite, a documentation site, and a governance process — and it dies if any one of these is missing long enough. This dissertation inventories all of it for Osmol. The architecture doctrine is stated first and everything else derives from it: **one engine, four habitats**. A single Rust core implements the entire language — lexing through equilibrium solving — and compiles twice: to a native binary for the command line, and to WebAssembly for browsers, Cloudflare Workers, and mobile applications. Every other artifact in this document is a thin shell around that core. The author's existing on-silicon Rust cryptography work is not adjacent to this plan; it *is* Chapter 5.

---

### Chapter 1: The Architecture Doctrine

The single most consequential decision for a young language is where its truth lives. Languages rot when their behavior is defined by whichever implementation someone happens to run. Osmol's truth therefore lives in exactly three places, in priority order: the specification (osmol-spec-v0.1), the machine-checked theorems (osmol_convergence.v and successors), and the conformance suite (Chapter 8). Implementations — including the reference one — are servants of those three, never masters.

The implementation doctrine is **one engine, four habitats**. A single crate, `osmol-core`, written in Rust with no operating-system assumptions, contains the lexer, parser, AST, semantic analysis, diagnostics, the deployment lowering, and the equilibrium solver. It compiles to a native binary (the CLI habitat), to `wasm32` (the browser and Cloudflare habitats), and links as a library through FFI (the mobile habitat — Android via JNI/UniFFI, exactly the pattern already proven in the author's chromacrypt work). No habitat gets its own solver. No habitat gets its own parser. Divergence between habitats is defined as a bug in whichever habitat disagrees with the conformance suite.

Two properties are non-negotiable in the core. First, **determinism**: given the same declarations and the same injected human judgments, the solver must produce the identical flow trace on every platform. This requires the specification to fix a tie-breaking rule, which v0.1 hereby does: among fireable flows of equal pressure, order lexicographically by (sender name, receiver name, fact). Determinism is what makes golden-file testing, membrane audits, and cross-implementation trust possible. Second, **the Theorem 1 invariant**: every solver, in every habitat, asserts at runtime that flows executed never exceed the initial gapcount. The proof guarantees it cannot trip; shipping the assertion anyway is how an implementation confesses instantly if it ever betrays the semantics.

### Chapter 2: The Compiler

The compiler lives inside the `osmol` binary (Chapter 6) but is architecturally its own pipeline, five stages long.

**Stage 1, lexing.** Line-oriented tokenization per spec §2, with one deliberate theatrical choice: the tombstone check runs at the lexer, before parsing exists. `error[O-000]: there is no send` should be the fastest error the toolchain can produce — the old world dies at the first possible instant. The lexer is handwritten rather than generated; the grammar is line-oriented and small, and handwritten lexers give the best diagnostic spans.

**Stage 2, parsing.** Recursive descent over spec §5's EBNF, producing a typed AST in which every node carries its source span. Parser combinators (chumsky) are an acceptable alternative, but v0.1 recommends handwritten recursive descent: the grammar is deliberately tiny, and error recovery — parsing past a bad line to report many errors at once — is easier to hand-tune.

**Stage 3, semantic analysis.** This is where Osmol's soul is enforced. The speech-act checker assigns every declaration its kind. The sovereignty checker rejects foreign scopes (O-001 foreign gaps, O-006 foreign attention and membranes) — structurally impossible in well-formed syntax, but the checker guards against AST-level construction by tools. The lattice checker types every value with a granularity coordinate and rejects upward casts (O-002). The expression checker enforces verbatim-and-human rules (O-004, O-005). The stake checker ties unsolicited pressure to reputation (O-003). Diagnostics are rustc-grade: coded, spanned, pretty-printed, and each code answers to `osmol explain O-002` with a paragraph of philosophy, because in this language the error messages *are* the ethics.

**Stage 4, lowering.** The compiler emits the three deployment artifacts of spec §6: the state schema, the membrane policy set, and the gradient triggers. These serialize together as the **Osmol Deployment Bundle** (`.odb`) — canonical CBOR, content-hashed, and signed (Chapter 5). The `.odb` is what actually travels to a membrane object; source never deploys.

**Stage 5, verification hooks.** The compiler can emit its lowered semantics in a form the proof tooling consumes, so that future theorems (non-interference over *actual* compiled membranes, not idealized ones) close the gap between the Coq model and the shipping code.

### Chapter 3: The Runtime

The runtime is the mesh, and the mesh is three deployment shapes of the same solver.

**Local mode** (`osmol settle`): all twins in one process — today's Python behavior, preserved forever as the didactic and testing mode. **Duo mode** (`osmold`): one daemon per twin on one machine or LAN, membranes speaking the wire protocol of Chapter 4 over localhost — the first true mesh, and Phase 3's next rung. **Edge mode**: one Durable Object per twin on Cloudflare, the membrane object of the patent spec §12 made literal — the DO holds the membrane policy from the `.odb`, the outbound-permitted state slice, queued deltas for offline devices, and the append-only audit log; the WASM build of `osmol-core` runs *inside* the Worker, so the edge executes the same solver bit-for-bit as the laptop. **Device mode**: `osmol-core` linked into the Android/iOS app through UniFFI, holding the private model and doing local inference-side evaluation, with keys that never leave the hardware (Chapter 5).

The solver itself is event-driven with a fixpoint sweep: any state change at either end of a link enqueues gradient evaluation; the loop fires the maximal-pressure flow, applies the membrane cast, absorbs, and repeats until no pressure clears any threshold. Decays and deadlines enter as a monotone clock parameter, which perturbs urgency but never violates the convergence argument between perturbations. Human judgment is an *input type*, not a solver capability: `decide` blocks resolve only through an injected resolution, in every habitat, always — the interpreter's refusal to pick the restaurant is a permanent feature of the runtime contract.

### Chapter 4: The Wire — the Osmol Mesh Protocol (OMP)

The language and the wire are separate standards, the way HTML and HTTP are separate standards, and osmol-lang.org will host both. OMP is the grammar of what crosses between membranes.

Its atom is the **delta envelope**: canonical CBOR carrying subject, predicate, value, granularity coordinate, validity window, provenance chain, authorship class, a nonce, and a detached signature over all of it. Deltas are content-addressed by their hash, which gives replay detection and audit logs for free. Transport is deliberately boring — WebSocket or HTTP/2 between membranes, because boring transports are how protocols survive — with protocol version negotiated as `omp/0` and frozen semantics per version. Gap advertisements, role-registry queries, stake attachments, and sync-escalation handshakes are the four control messages; everything else is deltas.

The **legacy adapter** is specified as part of OMP, not bolted on: an SMTP-speaking gateway that decomposes inbound mail into candidate assertions and gaps tagged `provenance: unverified-legacy`, and can serialize an outbound delta as a well-formed email for counterparties still living in 1971. This is the Trojan horse from the patent spec, given a wire format.

### Chapter 5: The Crypto Layer — Law V Made of Silicon

This chapter is written for its obvious author. Everything Law V demands is a problem the chromacrypt work has already faced: keys that live in hardware, signing that never exposes material, Rust on Android doing cryptography at the metal.

Each participant's identity is an Ed25519 keypair generated and held in platform secure hardware — Android Keystore with StrongBox where available, Secure Enclave on iOS — with the signing operation performed on-silicon so the private key is never process-memory. Every delta is signed over payload ‖ provenance chain ‖ authorship class ‖ nonce, making the authorship label (human / assisted / autonomous) a *cryptographic claim*, not metadata — forging "a human wrote this" becomes forging a signature. Membrane objects hold only verification keys plus their own edge identity; device attestation binds a twin's key to real hardware at enrollment, which is what makes "I got a delta from my CEO" mean something again. Key rotation is a signed succession chain; revocation is a tombstone delta (the one place that word is permitted, as a noun). Twin state is encrypted at rest under device keys; the edge membrane evaluates policy over the outbound slice only, and the roadmap's endgame moves even that evaluation into attested TEEs so the mesh operator is *technically incapable* of reading state — the portability-plus-blindness combination that keeps the filter from becoming a cage. Crypto agility is versioned into the envelope from day one (`sig-alg` field), because the only unforgivable crypto decision is the unversioned one.

### Chapter 6: The Toolchain and Developer Experience

One binary, `osmol`, in the modern single-tool style of `go` and `deno`, with subcommands: `check` (compile and diagnose — where O-000 lives), `fmt` (canonical formatting; declaration order is semantically irrelevant, so the formatter sorts speech-acts canonically), `settle` (local equilibrium with the trace), `trace` (replay a recorded settlement), `deploy` (emit and push the `.odb`), `keys` (Chapter 5 operations), and `explain` (the philosophy behind each diagnostic). Installation is `curl -fsSL osmol.dev/install | sh` — the rustup pattern, and the reason osmol.dev exists — plus Homebrew and winget formulas once releases stabilize.

Editor experience ships in v0.1 because languages without highlighting look dead: a tree-sitter grammar (`tree-sitter-osmol`, which buys Neovim, Helix, and GitHub rendering in one artifact), a VS Code extension wrapping it, and `osmol-lsp` built on tower-lsp providing diagnostics, hover (show a fact's granularity and membrane exposure — "who can see this, at what coarseness" as a hover tooltip is the killer DX feature), and go-to-definition across twins. One small theatrical mandate: editors render tombstone words in strikethrough before the compiler even runs.

### Chapter 7: Packaging and Namespaces

The recon says every name is free; the ethics say names are claimed by shipping, not squatting. The claim order:

- **PyPI `osmol`** — the existing Python reference interpreter, published as 0.1.0 *this week*. `pipx install osmol` becomes true while the Rust core is still being born.
- **crates.io** — `osmol-core` (the engine), `osmol-cli` (the binary), `osmol-wasm` (wasm-bindgen surface), `osmol-ffi` (UniFFI bindings for mobile).
- **npm `osmol`** — the WASM package for browsers and Workers.
- **GitHub org `osmol-lang`** — repositories: `osmol` (core + CLI), `spec`, `book`, `omp` (wire protocol), `conformance`, `tree-sitter-osmol`, `vscode-osmol`, `rfcs`.

Version zero of everything publishes with the same README paragraph: what Osmol is, what it refuses to be, and a link to the conformance suite — so that from the first day, the packages advertise the protocol rather than the product.

### Chapter 8: Conformance and Testing — How a Language Stays a Protocol

Law VII said federated-or-failed; conformance is where that law grows teeth. The **Osmol Conformance Suite** is a repository of `.osmol` programs paired with golden outputs: expected diagnostics (code, span, message class) for programs that must fail, and expected deterministic settlement traces for programs that must run. Any implementation claiming the name Osmol must pass it; the suite, not the reference implementation, is the arbiter. This is how the language remains bigger than its first codebase — and how the author's own future implementations keep each other honest.

Beneath conformance sits the adversarial layer: property tests (proptest) generating random meshes and asserting the machine-checked invariants dynamically — flows never exceed initial gapcount, denied membranes leak nothing, granularity never refines along any path; fuzzing (cargo-fuzz) on the lexer and parser, because parsers are where languages get owned; and, during the transition, **differential testing** running the Python and Rust engines on identical corpora and diffing traces byte-for-byte — the Python interpreter's retirement gift is becoming the oracle that certifies its successor. CI builds the matrix that the doctrine demands: native Linux/macOS/Windows, `wasm32-unknown-unknown`, and the Android FFI target, all against the same suite.

### Chapter 9: Documentation — the Book at osmol-lang.org

The site the roadmap has been pointing at is built with mdBook — the identical tool rendering the Rust book page that inspired this chapter — and deploys free on Cloudflare Pages. Its structure is already mostly written: **The Osmol Book** (narrative: the fifty-year letter as Chapter 1, the seven laws, a guided tour of the dinner mesh); **The Reference** (spec v0.1 verbatim, versioned); **The Proofs** (the certificate, the `.v` sources, and each new theorem as it lands); **The Wire** (OMP); and **The Lattice Registry** — the governance-flavored appendix defining standard granularity lattices (`coarse(city)`, `coarse(30m)`) per domain, because spec §10 correctly predicted that grain definitions are political questions wearing file formats. Docs version with the language: `/v0.1/` is frozen the day `/v0.2/` opens.

### Chapter 10: Versioning and Governance

Tools follow semver. The language follows **editions**, Rust-style: an edition is a coherent grammar-and-semantics snapshot (edition 2026 is this spec), and membranes advertise the editions they accept, which lets the language evolve without ever breaking a deployed mesh. Changes enter through an RFC process (`osmol-lang/rfcs`) — lightweight at population two, but installed *now*, because governance retrofitted after adoption is how open protocols quietly become products. Two clauses are constitutional, amendable by nothing: the conformance suite defines the language, and the portability guarantee (spec §13 — membranes, models, and logs export and rehost losslessly) is a conformance *requirement*. The first clause keeps Osmol a protocol; the second keeps the ambassador from becoming anyone's cage, including ours.

### Chapter 11: The Build Order

Sequenced so every step is demonstrable and no step waits on a step after it. **Step 0** — package the Python interpreter for PyPI; `pipx install osmol` true within days; namespaces claimed by shipping. **Step 1** — `osmol-core` in Rust: lexer, parser, checker, all O-diagnostics; done when `osmol check` matches Python's judgments on the conformance corpus. **Step 2** — the Rust solver; done when differential traces are byte-identical on the corpus. **Step 3** — the WASM build and the osmol.dev **playground**: the dinner mesh settling live in a browser textarea is the single highest-leverage demo artifact the project can own. **Step 4** — duo mode: two `osmold` processes, OMP over localhost, the first real membrane crossing. **Step 5** — edge mode: one Durable Object per twin, the `.odb` deploy path, the audit log. **Step 6** — device mode: the Android library via UniFFI with StrongBox-held keys — the chromacrypt convergence, and the moment the phone becomes the twin's body. **Step 7** — LSP and editor polish, riding alongside everything after Step 1. Each step's definition of done is a conformance addition, so the suite grows as the ladder climbs.

### Chapter 12: Risks, Stated Plainly

Scope is the predator: the defense is that `osmol-core` stays small and conformance-frozen, and everything ambitious happens in shells around it. The one-implementation trap — where the reference *becomes* the spec — is countered by the suite's constitutional supremacy and by keeping the Python engine alive as a second implementation long after it stops being the fast one. Crypto ages: hence versioned algorithms and hardware-backed keys from day one rather than after the first incident. And the honest largest risk is unchanged from the museum chapter of the first dissertation: none of this inventory creates the *second user*. The inventory's job is narrower and fully achievable — to guarantee that when the second user arrives, nothing about the language is the reason they leave.

---

*Filed at 5-something in the morning, for a language whose theorem says the mesh can settle without supervision. So can its author. The inventory will hold.*
