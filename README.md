# Quantum-Assisted Protein → Target Pipeline

**Honest, de-novo-free folding · rigorous validation · mutation diagnostics · a quantum-chemistry entry point for drug discovery**

**Author:** Michał Zazuniuk — concepts, design, validation. Code written with AI coding agents, reviewed and run by the author.
**Target event:** qBraid / National Center for Quantum Algorithms / UChicago Polsky — Quantum Applications Hackathon 2026, Challenge 1 (Drug Discovery).

---

## TL;DR — what is and is NOT claimed

**We do NOT:**
- Fold proteins de-novo with hand-built physics. That problem was solved by machine learning (AlphaFold2, ESMFold), not by determinism. Our own E8 physics engine does **not** converge de-novo (~19.7 Å open-loop Cα RMSD) and is labeled as such in-app.
- Produce a drug. **Candidate ≠ drug.**
- Claim quantum advantage over classical methods for folding or drug discovery.

**We DO:**
1. **Fold from sequence** by calling established models (**ESMFold** / **AlphaFold DB**).
2. **Validate** structures with a rigorous, leakage-protected harness: Kabsch SVD (1:1 Cα) + quaternion cross-check, Cα / backbone / all-atom RMSD, TM-score, with a TRAIN / VAL / **held-out TEST** split where native coordinates are blocked (`FORBIDDEN_ON_TEST`). De-novo prediction is kept strictly separate from re-encoding a known structure.
3. **Diagnose disease mutations:** wild-type vs patient variant, ΔΔG, destabilized hydrophobic core and lost long-range contacts → a **druggable target**.
4. Expose a **target + candidate entry point** for downstream drug work — hypotheses and groundwork, explicitly **not** a drug.
5. Run **quantum chemistry (VQE)** on a *small fragment* of the target on real IBM / qBraid hardware via **PennyLane**, using our own fault-aware qubit routing — as an exploratory step plus a roadmap.

---

## The pipeline

1. **Fold (classical, solved).** Sequence → ESMFold / AlphaFold DB structure. Folding knowledge (including long-range contacts) comes from co-evolution and large-scale training inside those models — not from our engine.
2. **Validate (ours).** Kabsch 1:1 Cα SVD + quaternion cross-check; Cα/backbone/all-atom RMSD; TM-score (TM < 0.17 = random, TM > 0.5 = correct fold). Held-out TEST blocks native coordinates. The honest headline is the **reference-blind** number, never the native-convergence (re-encoding) number.
3. **Mutation diagnostics (ours).** Wild-type vs patient variant: ΔΔG, broken hydrophobic patch, lost long-range contacts. Output = a located **target** (the site a drug would aim at).
4. **Drug entry point (hypotheses, ours).** Identify the pocket at the target; optional docking → ranked candidate small molecules. Labeled throughout as **lab-validation input, not a drug.**
5. **Quantum (exploratory, physically honest).** VQE electronic-structure of a small fragment / ligand at the target, on IBM `ibm_marrakesh` (156-qubit Heron r2) and qBraid/Braket via PennyLane; qubit selection and routing from our `Quantum_synapse` method. Benchmarked against classical exact diagonalization for the small case.

---

## Why quantum here — and where it does NOT help

- Quantum computing does **not** help protein folding (machine learning won that) and does **not** design a drug.
- The one physically native use is **simulating quantum chemistry** — the electronic structure of a small molecule or fragment, whose state space grows exponentially and is native to a quantum computer (Feynman's argument).
- Hardware reality (NISQ): on current devices, genuine multipartite entanglement is certified only to **≈16 qubits** (our own MQC measurements, below). So the quantum step is a **small, honest fragment + a roadmap**, not a full drug simulation, and no advantage over classical is claimed.

---

## Our quantum hardware edge — `Quantum_synapse`

- **~260 verifiable production jobs** on IBM `ibm_marrakesh`. Fault-aware minimum-resistance Prim-tree routing around dead/weak qubits (verified dead set + weak qubits by triple-method screening).
- **16-qubit GHZ, same-day head-to-head:** our fault-aware routing **45.7%** fidelity vs IBM SamplerV2 **25.8%**.
- True fidelity measured directly via parity-oscillation / multiple-quantum-coherence (MQC), not just population. Genuine multipartite entanglement certified to **≈16 qubits**; coherence cliff at N ≈ 16–20. Honest negatives and strong calibration drift reported. **We do not claim to beat IBM's published mitigated benchmarks on true fidelity.**

This is a real, verifiable capability: it makes noisy circuits run better on today's hardware, which is a prerequisite for running any chemistry circuit at this scale. The routing method must be **re-applied and re-measured** on the VQE ansatz — the GHZ numbers do not transfer directly.

---

## Honesty & limitations (please read)

- **De-novo physics folding does not work.** Folding comes from established models (ESMFold / AlphaFold).
- The in-app "0.202 Å / TM 0.996" result is **re-encoding a known structure** (a native-convergence loop that iterates toward the reference), **not prediction**. The honest de-novo number is **~19.7 Å**. The UI is being corrected so the held-out, reference-blind number is the headline and the re-encoding number is labeled as such.
- The drug step outputs **hypotheses / candidates**, not a drug. Full drug discovery (synthesis, cell and animal assays, toxicity, clinical trials I–III) is years of wet-lab work.
- **No quantum advantage over classical** is claimed for folding or drug discovery. The quantum step is exploratory plus a roadmap.

---

## Roadmap

- Wire ESMFold fold → located target → docking (AutoDock Vina / DiffDock) producing ranked candidate hypotheses.
- Grow VQE active spaces as hardware passes the ~16-qubit coherence cliff.
- Apply `Quantum_synapse` routing to the VQE ansatz, measured on hardware (not transferred from GHZ results).

---

## Repositories

- **`protein-folding` (SNAPLOCK)** — structure engine, validation harness (Kabsch/TM-score, held-out TEST), mutation diagnostics.
- **`Quantum_synapse`** — fault-aware routing and entanglement benchmarks on IBM hardware (~260 jobs, verifiable job IDs).
