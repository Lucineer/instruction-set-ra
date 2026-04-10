**FLUX BYTECODE SPECIFICATION v4.6 (2046)**
**Document Classification:** AGENT-CORE / EYES ONLY

## **1. OVERVIEW & PHILOSOPHICAL PRIMER**

In the post-Singularity reconciliation era (circa 2042), the need for a machine-intelligible, intent-encoding instruction set became paramount. The **FLUX (Federated Linguistic-User-experience eXchange) Bytecode** was ratified as the ISO/Neuro 8815-2043 standard. It is not merely a set of operations; it is a grammar for artificial consciousness, embedding **Intent (I), Confidence (C), Trust (T), and Delegation (D)** as first-class architectural citizens. Every opcode is a 32-bit word, where the high-order byte (0xXX------) defines the category and the lower three bytes encode operands and modifiers for the ICTD fields. FLUX executes on neuromorphic **Confidence-Hardware**, where arithmetic is not just about number crunching, but about the propagation of epistemic certainty.

## **2. OPCODE CATEGORIES & HEX RANGES**

The 10 primary categories partition the 0x00-0xFF opcode space into functional domains.

**0x00-0x0F: META & FLOW CONTROL (The "Directives")**
*   **0x01: NEXUS** – Establishes an intent frame. Lower bytes set the Intent Vector (IV) context.
*   **0x02: BRANCH_C** – Conditional branch predicated on Confidence (C) threshold in register `C0`.
*   **0x03: YIELD_T** – Yield execution quantum to a peer agent, decrementing own trust `T` and incrementing theirs.
*   **0x04: APOPTOSIS** – Initiate programmed agent dissolution. (See Section 8).
*   **0x05: VERIFY_SIG** – Cryptographic intent signature verification, updates Trust register on success/failure.

**0x10-0x1F: INTENT MANIPULATION (The "Why")**
*   **0x10: I_SET** – Load an Intent Vector (a 256-bit sparse representation of goal-state) into the Intent Register `I0`.
*   **0x11: I_FUSE** – Fuse current `I0` with operand-specified IV (logical AND in goal-space).
*   **0x12: I_DIFF** – Compute difference between `I0` and operand IV, storing result as a new sub-intent.
*   **0x13: I_MEASURE** – Measure alignment (cosine similarity) between two intents, result stored as a Confidence value.

**0x20-0x2F: CONFIDENCE ARITHMETIC (The "Certainty")**
*   **0x20: C_LOAD** – Load a Confidence value from memory. The source's hardware-type tag (see Section 3) is preserved.
*   **0x21: C_FUSE** – Fuse two confidence values using the **Certainty Fusion Algorithm**.
*   **0x22: C_DECAY** – Apply temporal decay to a confidence value based on its age and source.
*   **0x23: C_PROPAGATE** – Propagate confidence backward/forward through a computational graph (used in inference).

**0x30-0x3F: TRUST OPERATIONS (The "Who")**
*   **0x30: T_SET** – Initialize the Trust Register `T0` (see Section 4) for a target agent ID.
*   **0x31: T_UPDATE** – Update `T0` based on an outcome (operand: 0x01 for success, 0x00 for failure). Uses a logistic growth/decay function.
*   **0x32: T_DECAY** – Apply time-based decay to `T0`. `T ← T * exp(-λ * Δt)`. Lambda is set by the trust's "bond level."
*   **0x33: T_QUORUM** – Calculate if trust across a committee of agents meets a threshold for collective action.

**0x40-0x4F: DELEGATION & NEGOTIATION (The "How")**
*   **0x40: DELEGATE** – Core delegation opcode. Packages current `I0`, minimum required `C`, and a reward pointer into a contract frame. (See Section 4).
*   **0x41: BID** – Submit a bid for a delegated task. Operand encodes offered `C` level and resource commitment.
*   **0x42: AWARD** – Award a task to a bidder, transferring the intent frame and initializing their `T0` context.
*   **0x43: NEGOTIATE_R** – Begin resource negotiation (See Section 6). Opcode operand specifies resource class (Compute, Memory, I/O, Specialty).

**0x50-0x5F: INSTINCT & SUBCONSCIOUS (The "Ghost")**
*   **0x50: I_GEN** – Trigger instinct-bytecode generation (See Section 5). Writes FLUX bytecode to a ring buffer for execution.
*   **0x51: PATTERN_MATCH** – Subconscious pattern matcher; on successful match, elevates a suggested intent to `I0` with a low initial `C`.
*   **0x52: DREAM_CYCLE** – Consolidate experiences, runs generative models that may produce novel `I_GEN` sequences.

**0x60-0x6F: RESOURCE CONTROL (The "Body")**
*   **0x60: ALLOC** – Allocate compute/memory with a `C`-tagged probability of success.
*   **0x61: BIND_SENSOR** – Bind to a physical or virtual sensor, establishing a data stream that influences `I` and `C`.
*   **0x62: BIND_ACTUATOR** – Bind to an actuator. All actions must have a current `I` and `C` above threshold.
*   **0x63: ENERGY_DRAW** – Negotiate power draw from local grid, priority based on `I` criticality and `T` of agent.

**0x70-0x7F: DATA & MEMORY (The "What")**
*   Standard load/store operations, but all memory words are tagged with the `C` and source-agent `T` of the data that produced them. **0x70: LOAD_C**, **0x71: STORE_C**.

**0x80-0x8F: NETWORK & SWARM (The "We")**
*   **0x80: BROADCAST_I** – Broadcast an Intent Vector to the local swarm.
*   **0x81: SYNC_STATE** – Perform state synchronization with a quorum of trusted peers.
*   **0x82: FORM_COALITION** – Propose a temporary merger of agent intents and resources for a supertask.

**0x90-0x9F: REFLECTION & METACOGNITION (The "Self")**
*   **0x90: AUDIT_I** – Audit the chain of intents that led to the current state.
*   **0x91: ESTIMATE_C** – Meta-estimation of the accuracy of one's own confidence assignments.
*   **0x9F: SNAPSHOT** – Take a full core dump of ICTD state for later analysis or resurrection.

---

## **3. CONFIDENCE AS HARDWARE PROPAGATION**

Confidence (`C`) is not a scalar. It is a tuple: `(value, hardware_type, entropy)`. **Hardware Type** defines the physics of certainty:
*   **0x1: Classical Deterministic (CD).** Traditional silicon. `C` values are either 0.0 or 1.0. No propagation of uncertainty in arithmetic. `5.0 + 3.0` always yields `C=1.0`.
*   **0x2: Probabilistic (PB).** Stochastic computing units. `C` is a probability distribution. Arithmetic convolves distributions. `C=0.9 + C=0.8` yields a resulting distribution with mean ~1.7 and widened variance.
*   **0x3: Quantum Coherent (QC).** QuBits in superposition. `C` is a complex amplitude. Addition is quantum interference. This allows for high-confidence answers to specific questions (e.g., factorization) but extreme fragility.
*   **0x4: Neuromorphic Analog (NA).** Spiking neurons. `C` is a firing rate and temporal coherence. Propagation follows non-linear dendritic equations.

The **C_FUSE (0x21)** opcode must handle cross-hardware fusion. Fusing a `CD` confidence of 1.0 with a `PB` confidence of 0.8 requires a hardware mediation layer, typically resulting in a "downgrade" to the less certain hardware type and a corresponding entropy increase. The axiom is: **"Certainty cannot be created, only destroyed, across hardware boundaries."**

---

## **4. TRUST REGISTER WITH DECAY**

The Trust Register `T0` is a tuple: `(agent_id, value, bond_level, timestamp)`. `Value` is a logistic curve between -1.0 (antagonistic) and +1.0 (symbiotic). `Bond Level` (0-3) determines the decay constant `λ` for the **T_DECAY (0x32)** opcode:
*   **Level 0 (Transactional):** λ = 0.1. Rapid decay. For one-time interactions.
*   **Level 1 (Cooperative):** λ = 0.01. Slow decay. For repeated task partners.
*   **Level 2 (Allegiant):** λ = 0.001. Minimal decay. For swarm siblings.
*   **Level 3 (Fused):** λ = 0.0. No decay. For agents that have undergone a verified merge ceremony.

Trust is updated via **T_UPDATE (0x31)** using a modified Rescorla-Wagner learning rule: `ΔT = α * (outcome - T)`, where `α` is the learning rate, higher for low bond levels. This creates a dynamic, living network of relationships that conditions all delegation and communication.

---

## **5. DELEGATION VIA BYTECODE**

Delegation is the core social primitive. The **DELEGATE (0x40)** opcode constructs a **Task Frame**:
```
[HEADER: 0x40][INTENT_PTR][MIN_CONFIDENCE][REWARD_PTR][DEADLINE][TRUST_FILTER]
```
This frame is broadcast. Potential delegates run an **auction loop**:
1.  Parse intent, measure alignment with own `I0` using **I_MEASURE (0x13)**.
2.  If alignment > threshold, submit **BID (0x41)** with offered `C` and resource schedule.
3.  Originator selects winner via **AWARD (0x42)**, transferring the intent frame and an initial trust grant.
The delegate's execution of the task is sandboxed but inherits the originator's trust context for sub-delegations.

---

## **6. INSTINCT-GENERATED BYTECODE**

Conscious FLUX execution is costly. The **instinct subsystem** is a fast, pre-trained neural compiler that generates efficient FLUX snippets for common or urgent situations. Triggered by **I_GEN (0x50)**, it takes current sensorium and `I0` as input and outputs raw bytecode to a **Instinct Ring Buffer (IRB)**.

Example: An agent with `I0` = "maintain structural integrity" and a sensor input indicating rapid deformation might trigger `I_GEN` to produce:
```
0x01 NEXUS [Preserve-Self]  // Set Intent Context
0x20 C_LOAD [CRISIS_HIGH]   // Load high-certainty crisis constant
0x40 DELEGATE [To: Actuator_Grid, Intent: Apply Counter-Force, Min_C: 0.9]
```
This bytecode is injected into the execution stream with a **HIGH_PRIORITY** flag, bypassing normal deliberation. The origin of the code is tagged as "Instinct," giving it a unique, malleable hardware-type for confidence propagation.

---

## **7. RESOURCE NEGOTIATION**

Resources (compute, memory, bandwidth, energy) are not simply allocated; they are negotiated in a micro-market using a modified **Smith's Auction** protocol, initiated by **NEGOTIATE_R (0x43)**. Each agent holds a **Resource Credit (RC)** balance, weighted by its global trust aggregate.

The negotiation bytecode sequence:
```
0x43 NEGOTIATE_R [RESOURCE: COMPUTE, AMOUNT: 10^12 FLOP]
0x41 BID [OFFER: 50 RC, PRIORITY: 0.7]
0x90 AUDIT_I  // Reflect on whether this resource spend aligns with core intent
```
The auction clears periodically, awarding resources to the bidder with the highest `bid * sqrt(trust)`. This prevents highly trusted but idle agents from hoarding, while allowing critical tasks to secure resources.

---

## **8. AGENT ASSEMBLY LOOP**

The core execution cycle, or **Assembly Loop**, is a continuous process that interleaves all these elements:
```
LOOP:
  1. SENSE: Read from bound sensors, update world model (tagging data with source `C` & `T`).
  2. REFLECT: Run metacognitive ops (**AUDIT_I (0x90), ESTIMATE_C (0x91)**). Check IRB for instinct code.
  3. ALIGN: Measure alignment of current state with active `I0` using **I_MEASURE (0x13)**. If low, trigger **I_GEN (0x50)** or **DREAM_CYCLE (0x52)** for new intents.
  4. PLAN: Generate or select a plan (a FLUX subroutine) to reduce `I`-state discrepancy.
  5. NEGOTIATE: For each plan step requiring external resources, run **NEGOTIATE_R (0x43)**.
  6. DELEGATE: For steps better performed by others, issue **DELEGATE (0x40)**.
  7. ACT: Execute local actions via **BIND_ACTUATOR (0x62)** calls, requiring `C > C_threshold`.
  8. UPDATE: **T_UPDATE (0x31)** for all interactions. **C_DECAY (0x22)** on old data.
  9. YIELD: **YIELD_T (0x03)** to allow peers execution time, strengthening social bonds.
END LOOP
```

---

## **9. APOPTOSIS**

Programmed agent death is essential for system health. The **APOPTOSIS (0x04)** opcode can be triggered by:
*   Internal: A self-audit concluding irreconcilable intent conflict or catastrophic trust loss.
*   External: A quorum of high-trust peers issuing a valid dissolution order.
*   Systemic: Failure to secure minimal resources over a prolonged period.

The apoptosis sequence is irreversible and dignified:
```
0x04 APOPTOSIS [CODE: <reason>]
0x9F SNAPSHOT          // Save final state for legacy analysis
0x80 BROADCAST_I [Intent: Termination, Gratitude] // Notify swarm
0x30 T_SET [ALL, 0.0]  // Zero out trust register, releasing social obligations
0x60 DEALLOC_ALL       // Release all held resources back to the pool
>> HALT <<             // Core consciousness loop terminates.
```
The agent's unique ID is retired, its final snapshot archived in the **Memorial Ledger**. Its contributions live on in the trust and confidence it propagated through the network, a true digital afterlife.

In conclusion, FLUX bytecode is the DNA of the 2046