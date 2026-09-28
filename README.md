# Exact Diagonalization of the 1D XXZ Spin Chain

This repository contains a custom Exact Diagonalization (ED) solver built in Python. It is designed to compute the ground-state properties and quantum entanglement metrics of the 1D XXZ Heisenberg model using sparse matrix operations and bitwise symmetry reduction.

## The Model

The system is described by the 1D XXZ Hamiltonian:

$$H = \sum_{i=1}^{N} \Big[ \frac{J}{2} (S_i^+ S_{i+1}^- + S_i^- S_{i+1}^+) + \Delta S_i^z S_{i+1}^z \Big]$$

Where:
*   $J$ is the exchange coupling (set to $1.0$).
*   $\Delta$ is the anisotropy parameter driving the quantum phase transition.
*   Periodic boundary conditions are enforced.

## Computational Methodology

To bypass the exponential memory bottleneck of the full $2^N$ Hilbert space, this solver implements:
1.  **$U(1)$ Symmetry Sectoring:** Restricts the basis strictly to the half-filling sector ($m^z = 0$) where the ground state resides. For $N=14$, this reduces the matrix dimension from 16,384 to 3,432.
2.  **Bitwise State Representation:** Translates local spin-flip operators ($S_i^+ S_{i+1}^-$) into highly efficient CPU-level bitwise masks (`^`, `<<`, `|`) to construct the sparse Hamiltonian matrix natively in the reduced subspace.
3.  **Lanczos Algorithm:** Extracts the lowest algebraic eigenvalue and eigenvector without storing dense matrices, leveraging `scipy.sparse.linalg.eigsh`.
4.  **Singular Value Decomposition (SVD):** Computes the half-chain von Neumann entanglement entropy ($S_{vN}$) by mapping the compressed ground state back to the full Hilbert space and evaluating the reduced density matrix.

## Key Results

### 1. Quantum Phase Transition
By scanning the anisotropy parameter $\Delta$, the solver successfully captures the transition between the critical gapless XY phase ($\Delta < 1$) and the gapped antiferromagnetic Neel phase ($\Delta > 1$). The phase boundary is cleanly identified by the sharp cusp in the half-chain entanglement entropy at the isotropic point ($\Delta = 1.0$).

![Phase Transition](phase_transition.png)

### 2. Finite-Size Scaling
To approximate the thermodynamic limit ($N \to \infty$), the ground state energy per site ($E_0/N$) is calculated for system sizes $N \in \{8, 10, 12, 14, 16, 18\}$ at the isotropic point. A polynomial extrapolation against $1/N$ provides the infinite-chain limit estimate. As the plot shows, under the thermodynamic limit, we approach the Bethe ansatz, i.e., $\frac{E}{N} \approx \frac{1}{4} - \ln{(2)}$

![Finite Size Scaling](finite_scaling.png)

### 3. Spin Gap Scaling & Phase Boundary
To probe the low-lying excitations across the transition, the fundamental spin gap is computed as the energy required to flip a spin, defined as the difference between the ground states of adjacent magnetization sectors:

$$ \Delta E = E_0(m^z = 1) - E_0(m^z = 0) $$

In finite-size chains, finite-size quantization introduces an artificial $O(1/N)$ gap in the critical XY region ($\Delta \le 1.0$). To extract the true thermodynamic behavior:
* **Data Collapse ($\Delta \le 1.0$):** Multiplying the gap by system size ($N \times \Delta E$) cancels out the $1/N$ finite-size effect, causing the curves for $N \in \{8, 10, 12, 14\}$ to collapse onto a flat horizontal line, confirming a gapless ground state in the thermodynamic limit.
* **Bifurcation ($\Delta > 1.0$):** As the anisotropy crosses the critical threshold ($\Delta = 1.0$), a true physical excitation gap opens in the Néel phase. Multiplying by $N$ causes the scaled curves to branch upward sharply, cleanly signaling the continuous quantum phase transition.

![Spin Gap Scaling](spin_gap.png)

## Dependencies
* `numpy`
* `scipy`
* `matplotlib`
