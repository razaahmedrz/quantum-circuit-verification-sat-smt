# Exploring Classical Limits of Quantum Circuit Verification using SAT and SMT
This project explores classical verification of quantum circuit equivalence using SMT (Z3), prototype SAT encodings, and stabilizer methods, with empirical scaling analysis.

<b>Toolset used : Python, Qiskit, Z3 SMT Solver, PySAT (Boolean SAT solvers), NumPy / Matplotlib  </b>

This project investigates how classical reasoning tools specifically SMT solvers, Boolean SAT solvers, and stabilizer-based methods can be used to verify whether two quantum circuits implement the same transformation. 

Quantum circuits that are mathematically equivalent can look very different syntactically. Verifying equivalence is therefore a central problem in quantum compilation, optimization, and correctness checking. This repository explores multiple classical verification strategies and compares their scaling behavior and limitations. 

The main focus is not to build a complete industrial verifier, but to study how different classical representations affect tractability.


### The project implements and compares three approaches:

 ### 1. SMT-based verification using explicit unitaries (Z3)

Each quantum circuit is converted into its full unitary matrix using Qiskit.
The unitary entries are encoded as real-valued SMT variables.
The Z3 SMT solver checks whether two circuits are equivalent (optionally up to global phase).
This approach is general but scales exponentially with the number of qubits.

Purpose:
To study the cost of explicit classical representations of quantum transformations.

### 2. Stabilizer-based verification for Clifford circuits

Clifford circuits are represented using Pauli stabilizers instead of full matrices.
Stabilizers are propagated through gates using Clifford conjugation rules.
Equivalence is checked by comparing resulting stabilizer sets.

Purpose:
To demonstrate how structured representations avoid exponential blow-up for restricted circuit classes.

### 3. Prototype Boolean SAT encoding (PySAT)

A small-scale experiment encodes stabilizer evolution into Boolean CNF formulas.
A SAT solver (via PySAT) is used to check equivalence for very small circuits.
This encoding is exploratory and incomplete, intended as a proof-of-concept.

Purpose:
To explore how pure Boolean SAT formulations might apply to quantum verification.


### Explicit Unitary Equivalence Examples

<table align="center" width="100%">
  <tr>
    <td valign="top" width="100%">

<b>Case 1: H·X·H VS Z (Exact)</b>

<pre>
Circuit A
   ┌───┐┌───┐┌───┐
q: ┤ H ├┤ X ├┤ H ├
   └───┘└───┘└───┘

Circuit B
   ┌───┐
q: ┤ Z ├
   └───┘
</pre>

<pre>
Frobenius norm
||U(A) − U(B)|| = 3.14e−16
</pre>

</td>
<td valign="top">
  
<b>Case 2: CNOT VS H·CZ·H </b>

<pre>
Circuit A
q₀:  ──■──
     ┌─┴─┐
q₁:  ┤ X ├
     └───┘

Circuit B
q₀:  ──────■──────
     ┌───┐ │ ┌───┐
q₁:  ┤ H ├─■─┤ H ├
     └───┘   └───┘
</pre>

<pre>
Frobenius norm
||U(A) − U(B)|| = 4.44e−16
</pre>
</td>
  </tr>

  <tr>
    <td valign="top">

<b>Case 3: SWAP ≡ 3×CNOT (Exact)</b>
<pre>
Circuit A
q₀: ─X─
     │
q₁: ─X─

Circuit B
          ┌───┐     
q_0: ──■──┤ X ├──■──
     ┌─┴─┐└─┬─┘┌─┴─┐
q_1: ┤ X ├──■──┤ X ├
     └───┘     └───┘
</pre>

<pre>
Frobenius norm 
||U(A) − U(B)|| = 0.0
</pre>
</td>
<td valign="top">

<b>Case 4: S†·H·S VS X </b>

<pre>
Circuit A
   ┌─────┐┌───┐┌───┐
q: ┤ Sdg ├┤ H ├┤ S ├
   └─────┘└───┘└───┘
  
Circuit B
   ┌───┐
q: ┤ X ├
   └───┘
</pre>

<pre>
Frobenius norm
||U(A) − U(B)|| = 2.0
</pre>
</td>
  </tr>
</table>

<br>


### Correctness & Performance Comparison

| Test | Qubits | SMT Eq. | SMT Phase | SMT Time (ms) | SMT Clauses | Clifford Eq. | Clifford Time (ms) |
|------|-------:|:-------:|:---------:|--------------:|------------:|:------------:|-------------------:|
| H·X·H vs Z | 1 | True | 1 | 5.35 | 20 | True | 0.044 |
| CNOT vs H·CZ·H | 2 | True | 1 | 9.19 | 80 | False | 0.046 |
| SWAP vs 3×CNOT | 2 | True | None | 5.52 | 80 | True | 0.032 |
| S†·H·S vs X | 1 | False | None | 6.76 | 20 | False | 0.024 |

<br>

### Scaling Benchmark Results

<div align="center">

<table>
  <thead>
    <tr>
      <th>Qubits (n)</th>
      <th>Z3 SMT Time (s)</th>
      <th>Stabilizer Time (s)</th>
      <th>PySAT Time (s)</th>
      <th>Clauses</th>
      <th>Variables</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>1</td><td>2.901e-03</td><td>2.70e-05</td><td>9.60e-05</td><td>20</td><td>8</td></tr>
    <tr><td>2</td><td>6.163e-03</td><td>5.90e-05</td><td>4.15e-04</td><td>80</td><td>32</td></tr>
    <tr><td>3</td><td>1.791e-02</td><td>6.30e-05</td><td>1.11e-03</td><td>320</td><td>128</td></tr>
    <tr><td>4</td><td>6.508e-02</td><td>8.80e-05</td><td>—</td><td>1 280</td><td>512</td></tr>
    <tr><td>5</td><td>2.633e-01</td><td>1.64e-04</td><td>—</td><td>5 120</td><td>2 048</td></tr>
    <tr><td>6</td><td>1.083e+00</td><td>1.93e-04</td><td>—</td><td>20 480</td><td>8 192</td></tr>
  </tbody>
</table>

</div>


<br>

- Z3 uses explicit unitaries (size 2^n × 2^n) → grows exponentially.
- Stabilizer-style reasoning stays compact for Clifford circuits.
<br>



<p align="center">
    <img src="https://github.com/user-attachments/assets/ae07b636-65d6-4da6-a2a9-6312f1cda5f2" width="45%" />
  <img src="https://github.com/user-attachments/assets/759c9487-c9ac-42e3-9b56-efc6fcf6bc21" width="45%" />
</p>

<br>

# Future Work (In Progress) 

<img width="1910" height="1018" alt="image" src="https://github.com/user-attachments/assets/f040850e-d831-4636-a493-76ce03bf4044" />

<br>
<br>
<b>Future work includes developing an interactive GUI, symbolic SMT encodings of quantum gates, full global-phase handling, counterexample extraction, and scalable SAT-based verification methods, along with larger-scale systematic experiments, constraint-reduction techniques, and solver optimizations to better characterize the classical limits of satisfiability-based quantum circuit verification.</b>


