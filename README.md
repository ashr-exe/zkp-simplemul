# Zero Knowledge Starter: Multiplier Circuit

This repository documents my first steps into Zero Knowledge Proofs (ZKPs) using **Circom** and **SnarkJS**.

The goal of this project was to demystify the "black box" of ZKPs by building the simplest possible circuit: proving I know two secret factors (`a` and `b`) that multiply to a public number (`c`), without revealing the factors themselves.

## 🧠 Theoretical Foundations

Before writing code, I explored how we translate "Logic" into "Math" that a cryptographic system can verify.

### 1. Arithmetic Circuits (The Translator)
Computers use Boolean Circuits (AND/OR/NOT gates on bits). ZK-SNARKs use **Arithmetic Circuits** (Addition/Multiplication gates on numbers).
* We translate logic into two operations: `+` and `*`.
* Example: An "AND" gate becomes multiplication ($1 \times 1 = 1$, $1 \times 0 = 0$).

### 2. R1CS (The Constraint System)
Complex programs are "flattened" into a list of simple equations called a **Rank-1 Constraint System (R1CS)**.
Every step of the computation must fit the format:
$$(\text{Left Input}) \times (\text{Right Input}) = (\text{Output})$$
* This flattening allows the Verifier to check every step of the calculation simultaneously using Matrix Algebra (which eventually becomes the Polynomials in the QAP).

## 📝 Engineering Notes

### 1. The Circuit is not a Script
Unlike Python, a Circom file describes a hardware circuit. The core logic uses the `<==` operator, which performs two distinct jobs simultaneously:
* **Assignment (`=`):** Tells the witness calculator (WASM) to *compute* the value (e.g., `3 * 11 = 33`).
* **Constraint (`===`):** Adds a mathematical rule to the R1CS matrix that the Verifier checks (e.g., `Signal_C` MUST equal `Signal_A * Signal_B`).

*Crucial Insight:* If you only assign (`=`) without constraining (`===`), you create an **"Under-Constrained Circuit,"** allowing a prover to cheat by submitting a valid result derived from invalid inputs.

### 2. The Privacy Architecture
* **Private Inputs:** `a` and `b` (The secrets). These never leave the Prover's machine.
* **Public Output:** `c` (The claim). This is revealed to the Verifier.
* **The Proof:** A cryptographic artifact proving that `a * b = c` without revealing `a` or `b`.

## 🛠 Prerequisites

* **Node.js** (v20+)
* **Circom** (Rust compiler)
* **SnarkJS** (npm package)

## 🚀 Quick Start

### 1. Setup
Clone the repo and install dependencies (SnarkJS).
```bash
npm install
````

### 2\. Secrets Management

Create your secret input file. (This is git-ignored for privacy).

```bash
# Create input.json manually
# Example content: {"a": "3", "b": "11"}
```

### 3\. The Workflow (Commands)

**Step A: Compile**
Generate the R1CS constraints and the WASM witness calculator.

```bash
circom multiplier.circom --r1cs --wasm --sym --c
```

**Step B: Calculate Witness**
Take the secrets (`input.json`) and calculate the result (`witness.wtns`).

```bash
node multiplier_js/generate_witness.js multiplier_js/multiplier.wasm input.json witness.wtns
```

**Step C: Prove**
Generate the proof using the `witness` and the `zkey` (Proving Key).

```bash
snarkjs groth16 prove multiplier_final.zkey witness.wtns proof.json public.json
```

**Step D: Verify**
Check the proof using only the `verification_key` and `public.json`.

```bash
snarkjs groth16 verify verification_key.json public.json proof.json
```

## 📂 File Structure Explainers

  * `multiplier.circom`: The source logic.
  * `multiplier.r1cs`: The mathematical constraints (Matrices A, B, C).
  * `multiplier_final.zkey`: The Proving/Verification keys (derived from the Trusted Setup).
  * `witness.wtns`: The specific calculation instance (e.g., 3, 11, 33). *Generated locally, do not commit.*

<!-- end list -->

